---
layout: post
title:  "A Microservice with the RESTful Listener"
date:   2015-02-25 19:00:00
categories: pneumatic
comments: true
---

I was recently working on documentation for the RESTful Listener filter and wanted to write about an example.

I've been reading about [microservices](http://martinfowler.com/articles/microservices.html). I thought I would use the concept as an example for the RESTful Listener.

In this example, I'll describe a service that reads a JSON message and writes the content to a database. The job will be run though Spring Boot.

The example starts with the schema for the data received by the service. Pneumatic uses this schema to parse the JSON message posted to the service:

```XML
	<etl:schema id="mtbSchema" name="MTB Schema">
		<etl:column name="name" type="string" />
		<etl:column name="year" type="integer" />
		<etl:column name="cost" type="decimal" />
	</etl:schema>
```

The following JSON is an example of what this schema will parse:

```JSON
	{"name": "Mojo HDR", "year": "2015", "cost": "3000.00"}
```

Next comes the listener itself:

```XML
	<etl:pipe id="restfulListenerOutput" />
	<etl:restfulListener id="restfulListener" name="Restful Input">
		<etl:path value="mtb" />
		<etl:output ref="restfulListenerOutput" />
		<etl:outputSchema ref="mtbSchema" />
	</etl:restfulListener>
```

The `path` in the listener is the "resource" relative to the context root. Therefore, if your service is hosted on your computer at `http://localhost:8080`, then the service is at:

```
	http://localhost:8080/mtb
```

Finally, we have the data source and the database writer. It uses the data source (`id="dataSource`) to get a connection to the target database. It then writes the records on its input to the `mtb` table in the database, using the schema (`id="mtbSchema"`) declared earlier.

```xml
	<jdbc:embedded-database id="dataSource" type="HSQL">
		<jdbc:script location="classpath:db-schema.sql" />
		<jdbc:script location="classpath:db-test-data.sql" />
	</jdbc:embedded-database>

	<etl:databaseWriter id="databaseWriter" name="Database Writer">
		<etl:input ref="restfulListenerOutput" />
		<etl:inputSchema ref="mtbSchema" />
		<etl:dataSource ref="dataSource" />
		<etl:insertInto table-name="mtb" />
	</etl:databaseWriter>
```

If you want to connect to another database, see the [Spring Framework reference guide](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/jdbc.html#jdbc-connections) or Google for a data source example for your database.

To show how "micro" our microservice is, here's the full source, minus the boilerplate XML:

```XML
	<etl:schema id="mtbSchema" name="MTB Schema">
		<etl:column name="name" type="string" />
		<etl:column name="year" type="integer" />
		<etl:column name="cost" type="decimal" />
	</etl:schema>

	<etl:pipe id="restfulListenerOutput" />
	<etl:restfulListener id="restfulListener" name="Restful Input">
		<etl:path value="mtb" />
		<etl:output ref="restfulListenerOutput" />
		<etl:outputSchema ref="mtbSchema" />
	</etl:restfulListener>
        
	<jdbc:embedded-database id="dataSource" type="HSQL">
		<jdbc:script location="classpath:db-schema.sql" />
		<jdbc:script location="classpath:db-test-data.sql" />
	</jdbc:embedded-database>

	<etl:databaseWriter id="databaseWriter" name="Database Writer">
		<etl:input ref="restfulListenerOutput" />
		<etl:inputSchema ref="mtbSchema" />
		<etl:dataSource ref="dataSource" />
		<etl:insertInto table-name="mtb" />
	</etl:databaseWriter>
```

That's pretty small and no programming is required.

The easiest way to run Pneumatic jobs right now is to the `pn.sh` (`pn.cmd` on Windows). The `pn.sh` is in the `pneumatic-samples` project. To run a job using Spring Boot:

```sh
	sh pn.sh boot restful-listener-job.xml
```

You can find the source for `restful-listner-job.xml` in the `pneumantic-samples` project.

Use something like the [Advanced Rest Client](https://chrome.google.com/webstore/detail/advanced-rest-client/hgmloofddffdnphfgcellkdfbfbjeloo) plugin for Chrome and the URL and JSON above and you should be able to POST data to the service.

