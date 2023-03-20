title:: Why MySQL Could Be Slow With Large Tables
author:: [[Eduardo Krieg]]
url:: https://www.percona.com/blog/why-mysql-could-be-slow-large-tables
tags:: #[[database]] #[[Readwise]] #[[performance]] #[[Readwise]]

- > MySQL can handle TBs of data with good performance. ([View Highlight](https://read.readwise.io/read/01gtyad89m082dp63jdven7dd1))
- > Also, keep in mind that a portion of the primary key will be added at the end of each secondary index, so try to avoid selecting strings as the primary key, as it will make the secondary indexes to be larger and the performance will not be optimal. ([View Highlight](https://read.readwise.io/read/01gtyat82rrpr6ygmp84e86jn2))
	- 二级索引对应的数据的主键
- > However, there are cases where the same column is defined on multiple indexes in order to serve different query patterns, and sometimes some of the indexes created for the same column are redundant, leading to more overhead when inserting or deleting data (as indexes are updated) and increased disk space for storing the indexes for the table. ([View Highlight](https://read.readwise.io/read/01gtyaw26nexkpa8jecv0b9m8m))