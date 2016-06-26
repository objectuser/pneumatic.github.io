---
layout: post
title:  "Pneumatic YAML: Simplifying the Syntax"
date:   2016-06-27 08:00:00
categories: pneumatic
comments: true
---

The YAML syntax for Pneumatic.IO has a few wrinkles. I've ironed out a couple.

The two I'm talking about are the need for type declarations (`!pipe`, `!fileReader`, etc.) and the "reference" syntax when setting properties (`->`).

## Implicit Typing

Pipes and filters in Pneumatic have types, which are definitions of structure and behavior. In the past, pipes always looked like this in a Pneumatic job definition:

```YAML
# This is a pipe
fileOutput: !pipe
```

That still works. But you no longer need to tell Pneumatic that something is a pipe if you use some simple naming conventions. If you name something ending in "input", "output" or "pipe", Pneumatic will know you mean for it to be a pipe.

```YAML
# These are also pipes
fileOutput:
fileInput:
filePipe:
```

That's a bit less typing and noise as a payoff for using clear naming conventions.

## Simple References

References in Pneumatic used to be quite ugly. When configuring a databaseWriter, for example, it had to look something like this:

```YAML
mtbDatabaseWriter:
  name: Database Writer
  input: ->fileReaderOutput
  inputSchema: ->mtbSchema
  dataSource: ->dataSource # Reference the data source declared in Spring XML
  insertInto: mtb
```

I hated those references, they're so easy to forget and seem pointless when writing jobs. Why doesn't Pneumatic know that I'm setting a pipe to the `input` property?

Well, now it does. That filter would now be configured like this:

```YAML
mtbDatabaseWriter:
  name: Database Writer
  input: fileReaderOutput
  inputSchema: mtbSchema
  dataSource: dataSource # Reference the data source declared in Spring XML
  insertInto: mtb
```

Much nicer! Pneumatic should now do the right thing if a property needs an object or is just set by a primitive string.

I'm sure there are other wrinkles that can be ironed out a bit, but these were two of the biggest. I hope to continue to improve things to make the syntax better. If you have ideas, please let me know.
