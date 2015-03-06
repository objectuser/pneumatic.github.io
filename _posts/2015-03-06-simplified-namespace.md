---
layout: post
title:  "Simplified ETL Namespace"
date:   2015-03-06 08:00:00
categories: pneumatic
---

I've updated the examples and documentation to default to the ETL namespace. This removes the need for the ETL namespace prefix (``etl:``).

I think it makes the examples clearer. Here's a file reader in the ETL namespace:

```XML
<?xml version="1.0" encoding="UTF-8"?>
<beans:beans xmlns="http://www.surgingsystems.com/schema/etl"
	...
	<pipe id="fileReaderOutput" />
	<fileReader id="fileReader" name="File Reader 1">
		<fileResource location="classpath:data/companylist.csv">
			<skipLines value="1" />
		</fileResource>
		<output ref="fileReaderOutput" />
		<outputSchema ref="inputSchema" />
	</fileReader>
</beans:beans>
```

Nothing magical, but it removes some noise from the text.

If you prefer the previous namespace you can still use it by using the Spring `beans` namespace:

```XML
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	...
	<etl:pipe id="fileReaderOutput" />
	<etl:fileReader id="fileReader" name="File Reader 1">
		<etl:fileResource location="classpath:data/companylist.csv">
			<etl:skipLines value="1" />
		</etl:fileResource>
		<etl:output ref="fileReaderOutput" />
		<etl:outputSchema ref="inputSchema" />
	</etl:fileReader>
</beans>
```
