---
layout: post
title:  "The RESTful Writer"
date:   2015-02-28 08:00:00
categories: pneumatic
comments: true
---

As a sort of follow up to the [microservices]({% post_url 2015-02-25-restful-listener %}) post, I wanted to follow up with a post "from the other side". In that post, I demonstrated the creation of a microservice using just a few lines of Pneumatic markup. In this post, I'll demonstrate how to create a client for that service.

The RESTful writer filter will be the key for this task.

We'll use the same schema, which I'll reproduce here:

```XML
	<etl:schema id="mtbSchema" name="MTB Schema">
		<etl:column name="name" type="string" />
		<etl:column name="year" type="integer" />
		<etl:column name="cost" type="decimal" />
	</etl:schema>
```

We also need some data to send to the service. In the samples, there is a file called `mtb.txt` that we can use. Its contents look like this:

```text
	Yeti SB6c,2015,6499.0
	Trek Slash,2014,6000.0
	Cube Stereo,2015,4199.0
	Giant Trance,2014,3500.0
	Santa Cruz Bronson,2013,4500.0
	Ibis Mojo,2015,5500.0
```

Let's read the file and then send each line as a record to the RESTful service. To do that, we use the `fileReader` filter:

```XML
	<etl:pipe id="fileReaderOutput" />
	<etl:fileReader id="mtbReader" name="File Reader">
		<etl:fileResource location="classpath:data/mtb.txt" />
		<etl:output ref="fileReaderOutput" />
		<etl:outputSchema ref="mtbSchema" />
	</etl:fileReader>
```

Now, we send the output of the file reader to the RESTful writer:

```XML
	<etl:restfulWriter id="restfulWriter" name="RESTful Writer">
		<etl:http method="PUT" url="http://localhost:8080/mtb/{name}" />
		<etl:input ref="fileReaderOutput" />
		<etl:inputSchema ref="inputSchema" />
	</etl:restfulWriter>
```

The writer is using the `PUT` HTTP method, but it could use `POST` just as well. It's also using a column as part of the URL. Note the `{name}` token. This tells the filter to use the column with that name (`{name}`) as part of the URL. This means that, referring back to our `mtb.txt` input, the URLs would be:

```
	http://localhost:8080/mtb/Yeti%20SB6c
	http://localhost:8080/mtb/Trek%20Slash
	http://localhost:8080/mtb/Cube%20Stereo
	http://localhost:8080/mtb/Giant%20Trance
	http://localhost:8080/mtb/Santa%20Cruz%20Bronson,
	http://localhost:8080/mtb/Ibis%20Mojo
```

That's it. Here's the full source (minus the name space declarations):

```XML
	<etl:schema id="mtbSchema" name="MTB Schema">
		<etl:column name="name" type="string" />
		<etl:column name="year" type="integer" />
		<etl:column name="cost" type="decimal" />
	</etl:schema>
	
	<etl:pipe id="fileReaderOutput" />
	<etl:fileReader id="mtbReader" name="File Reader">
		<etl:fileResource location="classpath:data/mtb.txt" />
		<etl:output ref="fileReaderOutput" />
		<etl:outputSchema ref="mtbSchema" />
	</etl:fileReader>

	<etl:restfulWriter id="restfulWriter" name="RESTful Writer">
		<etl:http method="PUT" url="http://localhost:8080/mtb/{name}" />
		<etl:input ref="fileReaderOutput" />
		<etl:inputSchema ref="mtbSchema" />
	</etl:restfulWriter>
```

If you have an idea of how Pneumatic.IO can better support RESTful services, I'd love to hear from you (use the email link). A pull request is also welcome.