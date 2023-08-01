---
layout: default
title: How To Improve Database Performance
parent: Database
nav_order: 4
---

### How to improve database performance (MySQL and Postgres)
An index is a data structure that improves the speed of data retrieval from a database table. It works similar to an index in a book: Instead of reading the entire book to find specific information, you can use the index at the back of the book to quickly locate the page where the information is located.

In general, indexes are beneficial for frequently queried columns and should be used judiciously to achieve the best performance for specific application.

### create an index on the <table_name> table in a relational database. 
```shell
CREATE INDEX <name_of_index> ON <table_name> (column_1, column_2, column_3);
```
#### Explanation of above SQL statement
```shell
CREATE INDEX: This part of the command indicates that you want to create an index in the database.

<name_of_index>: This is the name of the index. In this case, it is named "<name_of_index>." The name is user-defined and can be any valid identifier.

ON <table_name>: This specifies the table on which the index is being created. In this case, the index is created on the "<table_name>" table.

(column_1, column_2, column_3): This part of the command defines the columns that will be included in the index. The index will be built on the combination of the values in the "column_1," "column_2," and "column_3" columns.
```
