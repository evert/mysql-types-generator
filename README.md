# mysql-types-generator

[![npm](https://img.shields.io/npm/v/mysql-types-generator)](https://www.npmjs.com/package/mysql-types-generator) [![install size](https://packagephobia.com/badge?p=mysql-types-generator)](https://packagephobia.com/result?p=mysql-types-generator)

**Warning: This library is incomplete; it works for some personal projects and is updated as issues are found. Use at your own risk.**

Inspects a mysql database and generates Typescript types for each table. Useful when you are using `knex` or raw mysql clients instead of an ORM.

Table names in the database must be in snake_case and will be converted to PascalCase for type names.

## Usage

Create a file to configure and run the generator:

`src/db/updateTypes.js`
```
import { generateMysqlTypes } from 'mysql-types-generator';

generateMysqlTypes({
  db: {
    host: 'localhost',
    port: 3306,
    user: 'myuser',
    password: 'mypassword',
    database: 'mydatabase'
  },
  output: {
    path: 'src/db/types', // or e.g. 'src/db/types.ts' for a single file output
  },
  suffix: 'PO',
  ignoreTables: [
    'my_table_a',
    'my_table_b',
  ],
  overrides: [
    {
      tableName: 'my_table',
      columnName: 'my_actual_tinyint_column',
      columnType: 'int',
    },
    {
      tableName: 'my_table',
      columnName: 'my_column',
      columnType: 'enum',
      enumString: `enum('a','b','c')`
    }
  ]
})
```

- `db` : **Required** - the database connection and credentials
- `output` : **Required**
  - `path` : **Required** - the path to the output directory or file
    - if `path` is a directory, each type will be output in a separate file in this directory, along with an `index.ts`. ***WARNING: This directory will be emptied and overwritten if it already exists.***
    - if `path` is a file (ending in `.ts`), all types will be put into that single file. ***WARNING: This file will be overwritten if it already exists.***
- `suffix` : Optional - a string appended to the PascalCase Type name (`PO` in the example refers to `Persistence Object` but you should use whatever convention you wish)
- `ignoreTables` : Optional - a list of tables to ignore; types won't be generated for these
- `overrides` : Optional - a list of columns where the column type in the database is ignored and the specified `columnType` is used instead
  - `columnType` can be any of the `mysql` column types, e.g. `'varchar'`, `'json'`, etc. Check the file `src/getColumnDataType.ts` in this repo for a list
    - if `columnType` = `'enum'`, you should specify `enumString`
  - `enumString` : Optional unless `columnType` = `'enum'`. Specify the enum options, for example `enum('a','b','c')` will become `'a' | 'b' | 'c'`

Run this file after running your database migrations. For example with `knex` :

`package.json`
```
(...)
  "scripts": {
    "migrate:dev": "npm run build && npx knex migrate:latest && node src/db/updateTypes.js"
  }
(...)
```

You can use [env-cmd](https://www.npmjs.com/package/env-cmd) to load environment variables from a `.env` file before running: `env-cmd node src/db/updateTypes.js`

## Notes
- `TINYINT` data type is always assumed to be `boolean`
- `SET` data type is treated as a simple string because `knex` returns a comma-delimited string in queries. You need to manually split it by comma if you want to convert it to an array or javascript `Set<>` type.

## Dependencies
- [mysql2](https://www.npmjs.com/package/mysql2)

## Change Log
- `0.0.12`
  - Fixed typos in `README.md`
- `0.0.11`
  - Bugfix: `overrides` config option wasn't working properly
  - Added feature: `output` can now be a path to a single file instead of a directory
  - Added feature: output files now contain a warning comment at the top to indicate that the file was auto-generated and will be overwritten