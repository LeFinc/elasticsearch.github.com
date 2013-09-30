---
layout: guide
title: Cascading
cat: guide
sidebar: reference_api
---
[Cascading](http://www.cascading.org/) is a data processing API and
processing query planner used for defining, sharing, and executing
data-processing workflows on a single computing node or distributed
computing cluster.

 

 

-- Cascading website

Cascading abstracting the Map/Reduce API and focusing on [data
processing](http://docs.cascading.org/cascading/2.1/userguide/htmlch03.html)
in terms of *tuples*
[*flowing*](http://docs.cascading.org/cascading/2.1/userguide/htmlch03s08.html)
through
[pipes](http://docs.cascading.org/cascading/2.1/userguide/htmlch03s02.html)
between
[*taps*](http://docs.cascading.org/cascading/2.1/userguide/htmlch03s05.html),
from input (called `SourceTap`{.literal}) to output (named
`SinkTap`{.literal}). As the data flows, various operations are applied
to the tuple; the whole system being transformed to Map/Reduce
operations at runtime. With elasticsearch-hadoop, Elasticsearch can be
plugged into Cascading flows as a `SourceTap`{.literal} or
`SinkTap`{.literal} through `ESTap`{.literal}.

**Local or Hadoop mode? **Cascading supports two *execution* modes or
[platforms](http://docs.cascading.org/cascading/2.1/userguide/htmlch03s04.html):

 Local 
:   for unit testing and quick POCs. Everything runs only on the local
    machine and file-system.
 Hadoop 
:   production mode - connects to a proper Hadoop cluster (as oppose to
    the *local* mode which is running just on the local machine).

elasticsearch-hadoop supports **both** platforms automatically. One does
not have to choose between different classes, `EsTap`{.literal} can be
used as both `sink`{.literal} or `source`{.literal}, in both modes
transparently.

### Installation

Just like other libraries, elasticsearch-hadoop needs to be available in
the jar classpath (either by being manually deployed in the cluster or
shipped along with the Hadoop job).

### Configuration

Cascading is configured through a `Map<Object, Object>`{.literal},
typically a `Properties`{.literal} object which indicates the various
Cascading settings and also the application jar:

~~~~ {.programlisting .prettyprint .lang-java}
Properties props = new Properties();
AppProps.setApplicationJarClass(props, Main.class);
FlowConnector flow = new HadoopFlowConnector(props);
~~~~

elasticsearch-hadoop options can be specified in the same way, these
being picked up automatically by all \`ESTap\`s down the flow:

~~~~ {.programlisting .prettyprint .lang-java}
Properties props = new Properties();
props.setProperty("es.index.auto.create", "false"); 
...
FlowConnector flow = new HadoopFlowConnector(props);
~~~~

[![1](images/icons/callouts/1.png)](#CO4-1)

set elasticsearch-hadoop option

This approach can be used for local and remote/Hadoop flows - simply use
the appropriate `FlowConnector`{.literal}.

### Type conversion

Depending on the
[platform](http://docs.cascading.org/cascading/2.1/userguide/htmlch03s04.html)
used, Cascading can use internally either `Writable`{.literal} or JDK
types for its tuples. Elasticsearch handles both transparently (see the
Map/Reduce
[conversion](mapreduce.html#type-conversion-writable "Type conversion")
section) though we recommend using the same types (if possible) in both
cases to avoid the overhead of maintaining two different versions.

![[Important]](images/icons/important.png)

If automatic index creation is used, please review
[this](mapping.html#auto-mapping-type-loss) section for more
information.

### Writing data to Elasticsearch

Simply hook, `ESTap`{.literal} into the Cascading flow:

~~~~ {.programlisting .prettyprint .lang-java}
Tap in = Lfs(new TextDelimited(new Fields("id", "name", "url", "picture")),
                                "src/test/resources/artists.dat");
Tap out = new ESTap("radio/artists" , new Fields("name", "url", "picture") );
new HadoopFlowConnector().connect(in, out, new Pipe("write-to-Eleasti")).complete();
~~~~

[![1](images/icons/callouts/1.png)](#CO5-1)

elasticsearch-hadoop resource (index and type)

[![2](images/icons/callouts/2.png)](#CO5-2)

Cascading tuple declaration

### Reading data from Elasticsearch

Just the same, add `ESTap`{.literal} on the other end of a pipe, to read
(instead of writing) to it.

~~~~ {.programlisting .prettyprint .lang-java}
Tap in = new ESTap("radio/artists/_search?q=me*" );
Tap out = new StdOut(new TextLine());
new LocalFlowConnector().connect(in, out, new Pipe("read-from-ES")).complete();
~~~~

[![1](images/icons/callouts/1.png)](#CO6-1)

elasticsearch-hadoop resource (index and type)

