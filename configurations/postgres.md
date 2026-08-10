- [Local](#local)
  - [Download](#download)
  - [pgAdmin](#pgadmin)
  - [Creating a table](#creating-a-table)
  - [Using the query tool](#using-the-query-tool)
- [Basic usage](#basic-usage)
  - [Config](#config)
  - [CLI Usage](#cli-usage)
  - [windows shell path](#windows-shell-path)
  - [commands](#commands)
  - [Creating a database basics](#creating-a-database-basics)
  - [Creating schemas](#creating-schemas)
- [Docker](#docker)
- [Node](#node)
  - [packages](#packages)
  - [The code](#the-code)
- [Vectors](#vectors)
  - [The math](#the-math)
  - [docker image](#docker-image)
  - [Getting started](#getting-started)
- [Indexing](#indexing)
- [Embedding](#embedding)
  - [Getting started](#getting-started-1)
  - [Code](#code)

# Local

## Download
Download it from https://www.postgresql.org/download/windows/

contains:
- The PostgreSQL server
- pgAdmin, a graphical tool for managing and developing your databases
- StackBuilder, a package manager for downloading and installing additional PostgreSQL tools and drivers.
- Command line tools - CLI for postgresql

## pgAdmin
 - Right click servers
    - Register server
        - give it any name
        - connection tab: 
            - localhost
            - username and password

## Creating a table

Select the database
- Schemas
    - public
        - right-click Tables 
            - Create
            - Table.
                - Give it a name: users

- Go to Columns tab
    - Add the following columns:
        - Column: id
            - Type: serial 
            - check Primary Key
        - Column: username 
            - Type: character varying(50) 
            - check Not Null + Unique
        - Column: email 
            - Type: character varying(100)
            - check Not Null + Unique
        - Column: full_name 
            - Type: character varying(100)
        - Column: created_at 
            - Type: timestamp 
            - Default: CURRENT_TIMESTAMP
Click Save.

## Using the query tool
```sql
-- When the table is not in the public schema, we gotta use its entire path test.xx
INSERT INTO test.users (username, email)
VALUES ('yosmel', 'yosmel@mail.com'), 
('hello', 'hello@world.com');

-- Gets all records
SELECT * FROM test.users;
```

# Basic usage

## Config
- The superuser is called `postgres`, similar to `root` in `mysql`
- set a password for superuser
- Port is `5432`

## CLI Usage
- Open a command line interface and do `psql --version` to verify installation
- Interact with the server with `psql -U postgres`
    - `psql -U testuser` (with no -d flag) always tries to connect to a database that has the exact same name as the username, so here it will connect to postgres default with postgres user
    - If we are trying to connect to a different database it must be `psql -U testuser -d testdb`

## windows shell path
If after installing, the above doesnt work, test this
```bash
export PATH="$PATH:/C:\Program Files\PostgreSQL\18\bin"
```

- change 17 with the version installed
- then try psql --version
- if it works, add it permantently:
    - Open Windows Search - type “Environment Variables”
    - Under System variables, find and edit Path
    - Give it the name `psql` then add:
    - C:\Program Files\PostgreSQL\18\bin

## commands
- \l or \list - similar to show databases
- \c dbname - select a database (postgres is selected by default)
- \d tablename - describe a table
- \du - list all users/roles and their attribues
- \dt - lists all tables in selected database - similar to show tables
- \? shows available commands
- \h shows helpful sql commands
- \! clear - clears the screen, `\!` acts as a shell runner, so it can do any shell commands without exiting `psql`
- \q exits psql

## Creating a database basics

```sql
-- Create the database
CREATE DATABASE myfirstdb;

-- Create a user for it
CREATE USER testuser WITH PASSWORD 'strongpassword';

-- Grant privileges
GRANT ALL PRIVILEGES ON DATABASE myfirstdb TO testuser;

-- Grant privileges on public schema
GRANT ALL PRIVILEGES ON SCHEMA public TO testuser;


```
- run `\l` to see a list of db and users with access to it
- run `du` to see what users can do

## Creating schemas
```sql
-- Creates a schema
CREATE SCHEMA mydbschema;

-- Shows current search paths
SHOW search_path;

-- Sets new path to point to the created schema 
SET search_path TO mydbschema;
```

# Docker
```bash
docker run -d -e POSTGRES_PASSWORD=myStrongPassword -p 5432:5432 postgres:18
```

Optional flags
- `-e POSTGRES_DB=dbname` - Sets a database (postgres is the default)
- `-e POSTGRES_USER=username` - Sets a usrename (postgres is the default)
- `-v postgres_data:/var/lib/postgresql` - Sets a persistent volume so data survives restarts/deletions

# Node

## packages
There are two packages, `pg` and `postgres`
- pg is the og, has most downloads and is supported by most ORMs like sequelize
- postgres is more modern

## The code
```js
require("dotenv").config();

const express = require("express");
const app = express();
const conString = process.env.DB || "";

const { Pool } = require("pg");

const pool = new Pool({ connectionString: conString });

app.get("/users", async (req, res) => {

    try {
        const sqlStr = "SELECT * FROM newschema.users";
        const { rows } = await pool.query(sqlStr);

        res.status(200).json(rows)

    } catch(err) {
        console.log("Error: ", err)
    }
    
})

app.get("/users/:id", async (req, res) => {
    const { id } = req.params;
    
    try {
        const sqlStr = "SELECT * FROM newschema.users WHERE id = $1"
        const { rows } = await pool.query(sqlStr, [id]);
        
        res.status(200).json(rows);

    } catch(err) {
        console.log("Error: ", err)
    }
})

app.listen(3000, () => console.log("Express is running on port 3000"))
```

# Vectors
To work with vectors in postgres with need an extension called [pgvector](https://github.com/pgvector/pgvector).

It adds:
- a special data type called `vector`
    -`CREATE TABLE items (id serial PRIMARY KEY, embedding vector(n));`
        - `n` is the dimensionality of the embeddings (1536 if using openai text embeddings)
- distance operators for perform mathematical similarity search
    - <-> - L2 distance
    - <#> - (negative) inner product
    - <=> - cosine distance
    - <+> - L1 distance
    - <~> - Hamming distance (binary vectors)
    - <%> - Jaccard distance (binary vectors)
- optimized indexing to handle embeddings alongside regular data in the database

## The math
Inside the database, to get the distance between related vectors, [cosine similarity](https://en.wikipedia.org/wiki/Cosine_similarity) is recommended when using `openai`.

## docker image
- there is an image called [pgvector](https://hub.docker.com/r/pgvector/pgvector), its based on postgres image with pgvector installed.

The image doesnt have a latest tag, so we gotta use one specifically, here is the code
```bash
 docker run -d --rm -it -e POSTGRES_PASSWORD=admin --name pgvector-test -p 5432:5432 pgvector/pgvector:0.8.5-pg18-trixie
```

## Getting started

Enable pgvector
```sql
CREATE EXTENSION IF NOT EXISTS vector;
```

Create a table where we can store vectors
```sql
CREATE TABLE movies(
    id serial PRIMARY KEY,
    content text NOT NULL,
    embedding vector(1536)
);
```

# Indexing

# Embedding
In order to populate a postgres database table with vector data, we need an [embeddings](https://developers.openai.com/api/docs/guides/embeddings) model.

An embeddings LLM model is a model that turns text strings into an array of floating point numbers (vectors). These numbers represent how the words and characters are related.

The distance between two vectors measures their relatedness.
- Small distance suggest high relatedness

Commonly used for:
- Search (where results are ranked by relevance to a query string)
- Clustering (where text strings are grouped by similarity)
- Recommendations (where items with related text strings are recommended)
- Diversity measurement (where similarity distributions are analyzed)
- Classification (where text strings are classified by their most similar label)

## Getting started
- Convert text to vector using an embedding LLM model such as `text-embedding-3-small`
- Upload the vector to the db

## Code
```js
import "dotenv/config";
import { OpenAI } from "openai";

const openai = new OpenAI({
    apiKey: process.env.OPENAPI_KEY
})

const embeddingsResponse = await openai.embeddings.create({
    model: "text-embedding-3-small",
    input: "Hello world",
    dimensions: 1536 // We can provide dimensions, which is optional default is 1536
})

console.log(embeddingsResponse.data)
```

The reuslt
```bash
[
  {
    object: 'embedding',
    embedding: [
        -0.00212860107421875,    -0.049041748046875,    0.0209808349609375,
             0.0313720703125,     -0.04534912109375,     -0.02642822265625,
            -0.0289306640625,     0.060272216796875,      -0.0257568359375,
         -0.0148162841796875,       0.0155029296875,       -0.030029296875,
          -0.020416259765625,     -0.03338623046875,    0.0258331298828125,
         0.01425933837890625,     -0.07000732421875,    0.0124053955078125,
           0.014801025390625,      0.04888916015625,    0.0207672119140625,
         -0.0088653564453125,  -0.01514434814453125,   -0.0166168212890625,
          0.0259552001953125,   -0.0028839111328125,   -0.0243988037109375,
           0.024261474609375, 0.0018024444580078125,    -0.055755615234375,
             0.0230712890625,     -0.04547119140625,  -0.00864410400390625,
       0.0030994415283203125,  0.004566192626953125, 0.0017919540405273438,
          0.0267486572265625,   0.01010894775390625,   -0.0120086669921875,
        -0.01151275634765625,   -0.0149078369140625,   -0.0231781005859375,
           0.025421142578125,      0.03680419921875,    -0.035552978515625,
            0.02130126953125,      -0.0631103515625,       0.0404052734375,
             0.0535888671875,      0.06158447265625,     -0.03363037109375,
           -0.00665283203125,       0.0255126953125,      0.10968017578125,
           -0.00469970703125,    -0.039520263671875,  0.007083892822265625,
           0.051544189453125,         -0.0263671875,    0.0278778076171875,
          0.0304107666015625,    0.0206146240234375,    0.0172576904296875,
         0.01236724853515625, 0.0010843276977539062,  0.007068634033203125,
          -0.037078857421875,    0.0235137939453125,     -0.01068115234375,
            0.04083251953125,  -0.00205230712890625,     0.031341552734375,
          -0.042755126953125,  -0.00675201416015625,      -0.0472412109375,
          -0.021759033203125,    -0.045684814453125, 0.0010194778442382812,
      0.00007617473602294922,     -0.04742431640625,       -0.032958984375,
          0.0234222412109375,    -0.052886962890625,     -0.05621337890625,
          -0.022552490234375,    0.0180816650390625,       -0.035888671875,
         0.00118255615234375, -0.006000518798828125,    -0.019805908203125,
           0.011199951171875, 0.0009560585021972656,     -0.02008056640625,
              0.045166015625,       0.0413818359375,   0.00545501708984375,
        -0.00733184814453125,      0.03912353515625,      0.05975341796875,
            -0.0086669921875,
      ... 1436 more items
    ],
    index: 0
  }
]
```
