# DBAL / Schema Reader
Every dbal driver provides the ability to read database and table schemas. Schema information includes all column names, their types, indexes and foreign keys.

## Getting list of tables
Once you receive an instance of `Database` you can check if the desired table exists using the `hasTable` method.

```php
if ($database->hasTable('users'))
{
}
```

In some cases you want to receive all database tables (array of `DBAL\Table`):

```php
foreach ($database->getTables() as $table)
{
    dump($table->getName());
}
```

> Attention, all table operation will automatically use database level table prefix. 

## Working with DBAL\Table
Once you received an instance of `Table`, the following operations (besides query builders) are available for you:

List of all table columns associated with their abstract types:

```php
dump($table->getColumns());
```

> Abstract type is internal representation of database type, you can read about it more in "DBAL / Schema Builder".

To get more low level table information, let's use `TableSchema`.

```php
$schema = $table->schema();
```

> Attention, table schema provides reading and writing abilities, you can read about it more in "DBAL / Schema Builder".

## Working with TableSchema
TableSchema provided low level access to table information such as column type (internal and abstract), indexes, foreign key. You can use this information to perform database export or build your own ORM.

Table primary keys:

```php
dump($schema->getPrimaryKeys());
```

Table indexes:

```php
foreach ($schema->getIndexes() as $index)
{
    dump($index->getName());
    dump($index->getColumns());
    dump($index->isUnique());
}
```

Table foreign keys (references):

```php
foreach ($schema->getForeigns() as $foreign)
{
    dump($foreign->getColumn());       //Local column name
    dump($foreign->getForeignTable()); //Prefix included (raw name)
    dump($foreign->getForeignKey());

    dump($foreign->getDeleteRule());   //NO ACTION, CASCADE
    dump($foreign->getUpdateRule());   //NO ACTION, CASCADE
}
```

Table columns:

```php
foreach ($schema->getColumns() as $column)
{
    dump($column->getName());

    dump($column->getType()); //Internal database type
    dump($column->abstractType()); //Abstract type like string, bigInt, enum, text and etc.
    dump($column->phpType()); //PHP type: int, float, string, bool

    dump($column->getDefaultValue()); //Can be instance of SqlFragment

    dump($column->getSize()); //Only for strings and decimal values

    dump($column->getPrecision()); //Decimals only
    dump($column->getScale()); //Decimals only

    dump($column->isNullable());
    dump($column->getEnumValues()); //Only for enums

    dump($column->getConstraints());

    dump($column->sqlStatement()); //Column creation syntax
}
```
> Some types can be mapped incorrectly if the table was created outside migrations or ORM. For example only MySQL has native enum support, all other databases use enum constraint.

## Console Commands
You can also use console commands to get information about configured tables and their schemas:

Command           | Description 
---               | ---
`dbal:databases`  | Get list of databases, their tables and records count.
`dbal:table`      | View table schema of specific database.


```
> ./spiral.cli dbal:table postgres people
Columns of postgres.people:
+---------+-------------------------+----------------+-----------+------------------------------------+
| Column: | Database Type:          | Abstract Type: | PHP Type: | Default Value:                     |
+---------+-------------------------+----------------+-----------+------------------------------------+
| id      | bigserial               | bigPrimary     | int       | nextval('people_id_seq'::regclass) |
| name    | character varying (255) | string         | string    | ---                                |
| income  | numeric (20,2)          | decimal        | float     | ---                                |
| cityID  | bigint                  | bigInteger     | int       | ---                                |
+---------+-------------------------+----------------+-----------+------------------------------------+

Indexes of postgres.people:
+-----------------------------------+-------+----------+
| Name:                             | Type: | Columns: |
+-----------------------------------+-------+----------+
| people_index_income_54ea144908c7c | INDEX | income   |
+-----------------------------------+-------+----------+

Foreign keys of postgres.people:
+-------------------------------------+---------+----------------+-----------------+------------+------------+
| Name:                               | Column: | Foreign Table: | Foreign Column: | On Delete: | On Update: |
+-------------------------------------+---------+----------------+-----------------+------------+------------+
| people_foreign_cityID_550ef169f1818 | cityID  | cities         | id              | CASCADE    | NO ACTION  |
+-------------------------------------+---------+----------------+-----------------+------------+------------+
```