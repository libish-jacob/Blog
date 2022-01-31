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

### Different workloads:
- OLTP – Online transaction processing. Lot of small operations involving whole row data.
- OLAP – Online analytics processing. A few large operations involving subsets of all columns.

Parquet is suitable for OLAP model processing.

### Encoding:
There are different ways for encoding data in parquet, but the most common ones are
- Plain – here the data is stored as-is.
- Dictionary encoding – Here repeating words are replaced by and integer value or an index. This is the most efficient one with 98% reduction.
- Delta encoding – This uses alphanumeric and special characters to replace repeating data where parts of the data are repeating. It can reduce the data by 58%

### Compression:
The file, as whole, is analyzed to see if repeating data can be replaced. In parquet, Column based approach has enabled compression opportunity since data is not fragmented. There are different options, and the most common ones are 
- LZO
- GZIP
- Snappy
Snappy is the default format. It is overall fast when compared to others. If the data is not accessed that frequently and if you don’t care about the query speed then use GZIP especially if you are selecting cold storage. If what you care is only the decompression or data access speed the use LZO. The tradeoff is between I/O savings vs decompression speed.
Parquet enables an option to split the data in different files. But if you are using GZIP then is not splitable.

### Parquet file:
On disk it is usually not a single file. Logical file is defined by a root directory. Root directory contains one or multiple files
./example_parquet_file
./example_parquet_file/part-000-8745-……-5678.snappy.parquet

 or contains sub directory structures with files in leaf directory.
./example_parquet_file
./example_parquet_file/country=Netherlands
./example_parquet_file/ country=Netherlands/part-000-8745-……-5678.snappy.parquet

### How to optimize the reading speed:
Metadata regarding the data is stored in the footer. Reader utilizes this and reads it first. The footer contains min max values of data in the row group. We can leverage this and speed up the reading process by asking the reader to consider this. This is enabled by default but in case if you need look for spark.sql.parquet.filterPushdown option in your reader. It doesn’t work well with unsorted data.
To get around this, pre-sort the predicate(The value by which you will search data)
Match predicate and column type. Don’t rely on the casting and conversion.

If you are using dictionary encoding for row group, then you can leverage dictionary filtering to make the search faster. The min max values in the footer cannot precisely say which row has the data since min and max can be in range which may not say exactly if the value is in there if the data is unsorted. But if we are using dictionary encoding, we could leverage from the index which dictionary encoding creates, which is stored at the beginning of the column chunk,  which can be used by reader to figure out if a data is in there. Enable this by looking for a setting like 
parquet.filter.dictionary.enabled – find the appropriate setting in the library that you are using.

### Optimization:
-### RLE_Dictionary encoding:
It is one of the best encodings in parquet. It uses 3 stages of encoding; Dictionary encoding creates index of repeating words and uses the index instead of the repeating words. RLE, Bit packing will check for repeating index and replaces it with an encode which says which index repeated how many times. E.g. (3,4). One of the downsides of this approach is that 
- 		Dictionary can become too big – When it exceeds the size, the encoding falls back to PLAIN.
-       There will be one dictionary per row group.
-       One way to get out dictionary reaching its size is to increase the dictionary size – 			parquet.dictionary.page.size – refer the library that you are using for this setting.
-       Another way is to decrease the row group size – parquet.block.size – this will help fit all data in the dictionary. – This is the preferred way.

-### Partitioning:



