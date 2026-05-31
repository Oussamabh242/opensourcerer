+++
title = "I hate database management tools"
date = "2026-02-07T22:40:41+01:00"
#dateFormat = "2006-01-02" # This value can be configured for per-post date formatting
author = "Oussama Ben Hassen"
authorTwitter = "" #do not include @
cover = ""
tags = ["Progmming", "Go", "Databases", "Dev Tools"]
keywords = ["SQL", "DB", "AI", "Programming", "Go", "Databases", "Dev Tools"]
description = "I despise database management tools, So i built my own."
showFullContent = false 
readingTime = true 
hideComments = false
+++

As you have probably guessed from the title or the description, in this post i am going to brag about building a DB visualization tool.

#### Are you in a hurry here is the repo so you can check it out: [Vision](https://github.com/Oussamabh242/vision).

#### You will find screenshots and how to use it in the reamdme.

# Why Bother

Ok, Ngl, i am not a big fan of database management tools, i find them to be bloated, and over kill, like for example openning pgadmin or dbeaver
i feel powerless seeing schema, tables, triggers, fucntion and all the other stuff that i don't really give a damn about, i just want to see the data and maybe run some queries, but i don't want to be overwhelmed by all the features that i don't need.

I use `Prisma` for my typescript projects and i really really like the `prisma studio` it is simple, fast and very minimal and gets the job done
but it has its own problems:

- Not being able to connect to other databses so you are stuck with the db in you env file.
- Not being able to run custom queries, so if i need something I rely on the `psql` or `sqlite3` cli tools which are not a problem for me but you know some times you just want to run a quick query without opening a terminal and typing the command.
- Not being able to see the schema, so if i want to see the tables and their columns i have to open the terminal and run `\d` and it would be nice to see it in a nice way.
- Not portable, so if i want to explore a db of a project for exmp that is not written in typescript or that doesn't use prisma i would rely on the terminal.
- Sometimes you like to run a query and see the results and you don't know how to write it so you rely on AI by copying the schama and the data and asking it to write the query for you, but you can't do that with prisma studio. (i like it in the mongodb compass)

The outcome of this frustration is to be a good boy and build my own database managment tool (Vision btw), and satisfy my urge to see the data in a nice way and run custom queries.

# The Tech Sack

### Programming language:

`Go`, It's my favourite programming language, it's fast, it's simple and it has a great standard library, and it is also very good for building stuff
it is verbose without the `try catch` hell, and it produces portable binaries that can run on different platforms without any deps, And i didn't want to use
Rust because it has become the new "One For All" language.

### UI Stuff:

I Chose to use:

`Templ` for the UI, for being react like, simple to use, typesafety and you can split your UI into components and reuse them.

`Tailwind CSS` the obvious choice nowadays.

`HTMX` for the interactivity, it is a great library that allows you to make your UI interactive without writing any javascript, it is simple to use and it works well with templ.

`Alpine.js` for the small interactions that htmx can't handle, it is a great library that allows you to write small interactions in a simple way without writing a lot of javascript.

### Database Drivers:

I use the `sqlx` library for the database drivers, it is a great library that allows you to work with different databases in a simple way, it supports postgres, mysql, sqlite and many other databases, it also has a great API that allows you to work with the database in a simple way.

### AI:

I use the `Azure OpenAI` for it is free that's it.

# Implemetation

Implemetation is quite striaght forward, i am not going to brag about it that much, i will just list the features that this tool has and small details about the hexagonal architecture that i used to build it.

### Features:

1. Connect databases, add SQLite/PostgreSQL database connections.
2. Test connections first, check if credentials/connection details work before saving.
3. Save/reuse connections, previously added DBs show up again on the home screen.
4. Browse tables, see all tables inside a connected database.
5. View table rows, inspect table data directly in the browser.
6. Pagination + sorting, move through large tables and sort columns.
7. Filter table data, narrow down rows without full page reloads.
8. Inspect schema, view columns, types, primary keys, and table metadata.
9. Create/update/delete rows, basic CRUD directly from the UI.
10. Run SQL / AI-generate SQL, execute custom SQL, or type a natural-language request and let the assistant generate SQL.

### Hexagonal Architecture:

the UI and handlers don’t talk directly to Postgres or Sqlite, they talk to a port/interface, and each database connector
implements it. For example:

```go
  type DatabasePort interface {
      TestConnection(ctx context.Context, params ConnectionParams) error
      Connect(ctx context.Context, params ConnectionParams) (DatabasePort, error)

      ListTables(ctx context.Context) ([]types.TableItem, error)
      GetTableSchema(table string) (types.TableSchema, error)
      GetRowsForTable(table string, params types.QueryParams) ([][]string, []string, error)

      GetTableJsonSchema(tableName string) (string, error)
      SearchTables(query string) ([]string, error)
      GetSampleData(table string) (string, error)

      CreateRow(table string, cols []string, data []string) error
      UpdateRow(table string, cols []string, data []string, identity types.RowIdentity) error
      DeleteRow(table string, identity types.RowIdentity) error
  }
```

So `postgres` and `sqlite` are just adapters behind this interface. If I want to add `mysql` later, I don’t need to rewrite the table browser, SQL page, CRUD handlers, or AI assistant I only create a `MySQLAdapter` that satisfies DatabasePort and register it in the manager.

The AI layer benefits from the same design: tools like
search_tables, get_table_json_schema, and get_sample_data call the manager/port, not a specific database, so the assistant automatically works with any new connector that implements the interface.
