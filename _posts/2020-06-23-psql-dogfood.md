---
title: "PostgreSQL Makes Delicious Dog Food"
layout: post
---

# Introduction

It's probably not news to you, but folks behind the [PostgreSQL
Database](https://www.postgresql.org) have made an _incredible_ piece of
software. It's fast, it can handle loads of data, and it has interesting
built-in functions and index types for almost anything you can think to ask.
For this post, though, I'd like to highlight how it facilitates DB maintenance
tasks by [eating its own dog
food](https://en.wikipedia.org/wiki/Eating_your_own_dog_food)—that is, how
answering questions _about_ the database is no different from running regular
SQL queries.

# Problem statement

At [work](https://www.quantifind.com), we've got a fair amount of data. I'm
charged with the care and feeding of a couple of these databases, one of which
recently got a _huge_ upgrade of incoming data; previously it only contained
our data vendor's restricted version of US and Canada data, and we upgrade to
the `WHOLE_DANG_WORLD` option.

When we (I) created our DB schema, we relied heavily upon materialized views;
the thought was we'd keep the incoming raw data in tables, perform some
DB-level ETL extraction work, then put the main data we wish to use into
consolidated materialized views. This design made sense given what we thought
our paradigm was going to be: we were expecting

  - raw data that needed extensive work before being suitable for processing, and
  - we would receive small incremental updates of that data frequently from our vendor, so we could `UPDATE` the raw tables and `REFRESH` the materialized views to stay up to date.

However, this is decidedly not the paradigm in which we find ourselves. The
data is almost ready for our ML algorithms to grab, modulo some
normalizations, and instead of frequent small updates we receive quarterly
dumps of the _entire_ database. This led to a lot of waste, but there was one
particular pain point: because we used materialized views, when our intrepid
Ops team copied the DB over to other environments (e.g. stage & prod), they had
to _rebuild every index_ on the materialized views—unlike a table index, an
index on a materialized view doesn't copy over.

I had a strong suspicion that we didn't need every index we were copying over;
over half are obsolete artifacts of query patterns we no longer used. I thought
of combing through our Scala monolith to find every time we accessed these
materialized views via [Slick](https://scala-slick.org), but calling that
error-prone would be an understatement. Part of the appeal of Slick is that it
turns normal-looking Scala code into SQL queries for you, but that means you
need to translate it—either in code by, say, debug logging the query statements
or in your head—to fully understand what it's doing with your DB.

# It's tables all the way down

Luckily, the PostgreSQL devs have made answering these types of questions 
manageable. One answers questions about PostgreSQL by, well, querying PostgreSQL.
There are a bevy of [system
catalogs](https://www.postgresql.org/docs/9.6/catalogs-overview.html) and
[collected statistics
views](https://www.postgresql.org/docs/12/monitoring-stats.html#MONITORING-STATS-VIEWS-TABLE)
chock-full of helpful information.

Trimming down an example query from the wiki page on [index
maintenance](https://wiki.postgresql.org/wiki/Index_Maintenance), I ran the
following to get the names, table names, and human-readable sizes of the
indexes that were

- public (`d.schemaname = 'public'`), meaning we made them,
- not unique (`not a.indisunique`), so they're not primary keys[^1], and
- never scanned (`d.idx_scan = 0`), so they're unused:

```SQL
select
  c.relname as index_name,
  b.relname as table_name,
  pg_size_pretty(pg_relation_size(
    quote_ident(d.schemaname) || '.' || quote_ident(d.indexrelname)
  )) as index_size
from pg_index a
join pg_class b on b.oid = a.indrelid
join pg_class c on c.oid = a.indexrelid
join pg_stat_all_indexes d on a.indexrelid = d.indexrelid
where d.schemaname = 'public' and not a.indisunique and d.idx_scan = 0
order by table_name, index_name;
```

When run, this query immediately returns something like[^2]

```
|   index_name   |   table_name   |   index_size   |
|----------------|----------------|----------------|
|   unused_idx   |   a_table      |   1.81 GB      |
|   also_bad     |   another_tbl  |   77 MB        |
```

With this helpful query in hand, I knew exactly which indexes we could safely
delete and how much memory we'd save the database by doing so.


[^1]: Or the approximate equivalent for materialized views, which can't have primary indexes, but can have unique indexes.
[^2]: Names changed to protect the guilty.
