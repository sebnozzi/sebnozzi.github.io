---
id: 444
title: Slick-based (Scala) Flyway Migrations
date: 2015-04-08T22:31:18+00:00
author: Sebastian Nozzi
layout: post
guid: http://www.sebnozzi.com/?p=444
permalink: /444/slick-based-scala-flyway-migrations/
categories:
  - Scala
tags:
  - flyway
  - scala
  - slick
---
One of the features I really like about [Flyway](http://flywaydb.org/) is the ability to have [Java-based](http://flywaydb.org/documentation/migration/java.html) migrations.

But how would one go about using [Slick](http://slick.typesafe.com/) inside a Flyway Java-migration?
  
<!--more-->

Particularly: how does one get hold of a Slick &#8220;Session&#8221; from one java.sql Connection?

The trick is in using Slick&#8217;s &#8220;[UnmanagedSession](http://slick.typesafe.com/doc/2.0.1/api/index.html#scala.slick.jdbc.UnmanagedSession)&#8220;&#8230;

Here I want to share with you a trait that can be used to have first-class Slick Flyway migrations:

```scala
package db.migration

import org.flywaydb.core.api.migration.jdbc.JdbcMigration
import scala.slick.jdbc.UnmanagedSession
import scala.slick.driver.JdbcDriver.simple._
import java.sql.Connection

/**
 * By including this trait in your Flyway JdbcMigration, you only
 * need to provide an implementation for this the slick_migrate
 * method, which accepts a Slick session.
 *
 * Given that Session, you can perform any Slick operation, like
 * creating or modifying a table, populating the database, etc.
 *
 */
trait SlickMigration { self: JdbcMigration =>

  // Implement this in your subclass
  def slick_migrate(implicit session: Session)

  override final def migrate(conn: Connection) {
    val session = new UnmanagedSession(conn)
    try {
      session.withTransaction {
        slick_migrate(session)
      }
    } finally {
      session.close()
    }
  }

}
```

Once you have this in place, you can write a Slick-based migration like this:

```scala
package db.migration.default

import org.flywaydb.core.api.migration.jdbc.JdbcMigration
import scala.slick.driver.JdbcDriver.simple._
import db.migration.SlickMigration

class V1__001_CreateMoviesTable extends JdbcMigration with SlickMigration {

  override def slick_migrate(implicit session: Session) {

    class Movies(tag: Tag) extends Table[(Option[Int], String, Int)](tag, "MOVIES") {
      def id = column[Int]("ID", O.PrimaryKey, O.AutoInc)
      def title = column[String]("TITLE")
      def year = column[Int]("YEAR")
      def * = (id.?, title, year)
    }
    val movies = TableQuery[Movies]

    session.withTransaction {
      movies.ddl.create
    }
  }

}
```

Advantages:

  * One can create tables, and manipulate data in an DB-agnostic way
  * One has the full power of Slick / Scala

Disadvantages:

  * Slick is not suited for table or column alterations. For this, other approaches should be sought. 

Happy Scala / Slicking!
