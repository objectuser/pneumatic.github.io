---
layout: post
title:  "Partitioning from a Database"
date:   2015-03-15 10:00:00
categories: pneumatic
comments: true
---

One way of scaling ETL solutions is to split the data, sending it down different processing paths that better use available infrastructure, thus increasing the capacity available for processing.

[Partitioning](http://en.wikipedia.org/wiki/Partition_%28database%29) is a strategy for segmenting data along a particular key. We can use this strategy in Pneumatic.IO by creating separate database readers, either in separate jobs (possibly to scale horizontally), or in the same job (to make better use of the capacity of a single machine).

The database readers might look something like this:

```XML
	<pipe id="databaseOutput1" />
	<databaseReader id="databaseReader1" name="Database Reader A-I">
		<output ref="databaseOutput1" />
		<outputSchema ref="schema" />
		<dataSource ref="dataSource" />
		<select>
			<sql>
				select *
				from hospital
				where HospitalName &lt; 'J'
			</sql>
		</select>
	</databaseReader>
```

For efficient partitioning, segmenting the data evenly is necessary, but dividing the alphabet by three in this example, serves our purposes.

The `dataSource` may be local or remote: Pnematic uses standard JDBC drivers to connect to databases.

You can find this code in the following samples in `pneumatic-samples`:

* `insert-hospital-records.xml`
* `dump.xml`
* `partitioning.xml`

You might run the first one using the Pneumatic Shell:

```sh
sh pn.sh run configs/performance/insert-hospital-records.xml
```
