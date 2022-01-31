---
layout: post
modified: '2022-01-31 19:42 +0530'
published: false
comments: true
title: 'Parquet file experiments, findings and recommendations'
---
   Parquet is a binary file format designed with big data in mind where we must access data frequently and efficiently. The way it stores file on the disk is also different from other file formats. It is a column-based data file. And in reality it uses both row based and column based approach to bring the best of both worlds. The data is encoded on disk which ensures that the size remains small compared to actual data and is then compressed where the file is scanned as whole and cut out redundant parts. The query/read speed is dramatically fast when compared to other file formats. Nested data is handled efficiently which is quite cumbersome in other file format to achieve. Doesn’t require to parse the entire file to find data due to its way of storing data. This makes it efficient in reading data. Works quite efficiently with data processing frameworks. Automatically stores schema information. SQL querying is possible with this file format using proper tools.
   
### Data formats:
Data formats can be
- Unstructured – When there is no specific structure. e.g Text, csv
- Semi structured – XML, Json
- Structured –  Has records and rows, well defined schema, has very predictable locations where you can find the data - SQL, Parquet

Parquet uses structured data format.

### Data storage:
On a Logical front data is stored as – Rows and columns.
On Physical front data can be stored as– 
- Row based – each row is written one after the other – column data is fragmented.
- Column based – each column is written one after the other. Here the column data is stored together. To read subset of column, it must read full column value.
- Hybrid – mix of both row based, and column based are used. Here column data chunk is broken down to manageable chunks and stored. It ensures that entire column chunk does not have to be read to read a subset of column value.
Parquet use the hybrid approach.





