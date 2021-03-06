---
title: "Scala Shell"
# Top-level navigation
top-nav-group: apis
top-nav-pos: 10
---
<!--
Licensed to the Apache Software Foundation (ASF) under one
or more contributor license agreements.  See the NOTICE file
distributed with this work for additional information
regarding copyright ownership.  The ASF licenses this file
to you under the Apache License, Version 2.0 (the
"License"); you may not use this file except in compliance
with the License.  You may obtain a copy of the License at

  http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing,
software distributed under the License is distributed on an
"AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
KIND, either express or implied.  See the License for the
specific language governing permissions and limitations
under the License.
-->


Flink comes with an integrated interactive Scala Shell.
It can be used in a local setup as well as in a cluster setup. To get started with downloading
Flink and setting up a cluster please refer to
[local setup]({{ site.baseurl }}/setup/local_setup.html) or
[cluster setup]({{ site.baseurl }}/setup/cluster_setup.html)

To use the shell with an integrated Flink cluster just execute:

~~~bash
bin/start-scala-shell.sh local
~~~

in the root directory of your binary Flink directory.

To use it with a running cluster start the scala shell with the keyword `remote`
and supply the host and port of the JobManager with:

~~~bash
bin/start-scala-shell.sh remote <hostname> <portnumber>
~~~

## Usage

The shell supports Batch and Streaming.
Two different ExecutionEnvironments are automatically prebound after startup.
Use "benv" and "senv" to access the Batch and Streaming environment respectively.

The following example will execute the wordcount program in the Scala shell:

~~~scala
Scala-Flink> val text = benv.fromElements(
  "To be, or not to be,--that is the question:--",
  "Whether 'tis nobler in the mind to suffer",
  "The slings and arrows of outrageous fortune",
  "Or to take arms against a sea of troubles,")
Scala-Flink> val counts = text.flatMap { _.toLowerCase.split("\\W+") }.map { (_, 1) }.groupBy(0).sum(1)
Scala-Flink> counts.print()
~~~

The print() command will automatically send the specified tasks to the JobManager for execution and will show the result of the computation in the terminal.

It is possbile to write results to a file. However, in this case you need to call `execute`, to run your program:

~~~scala
Scala-Flink> benv.execute("MyProgram")
~~~

The Batch program above can be executed using the Streaming API through:

~~~scala
Scala-Flink> val textStreaming = senv.fromElements(
  "To be, or not to be,--that is the question:--",
  "Whether 'tis nobler in the mind to suffer",
  "The slings and arrows of outrageous fortune",
  "Or to take arms against a sea of troubles,")
 Scala-Flink> val countsStreaming = textStreaming.flatMap { _.toLowerCase.split("\\W+") }.map { (_, 1) }.keyBy(0).sum(1)
 Scala-Flink> countsStreaming.print()
 Scala-Flink> senv.execute("Streaming Wordcount")
~~~

Note, that in the Streaming case, the print operation does not trigger execution directly.

The Flink Shell comes with command history and autocompletion.

## Scala Shell with Flink on YARN

The Scala shell can connect Flink cluster on YARN. To connect deployed Flink cluster on YARN, use following command:

~~~bash
bin/start-scala-shell.sh yarn
~~~

The shell reads the connection information of the deployed Flink cluster from the `.yarn-properties` file, which is created in the configured `yarn.properties-file.location` directory or the temporary directory. If there is no deployed Flink cluster on YARN, the shell prints an error message.

The shell can deploy a Flink cluster to YARN, which is used exclusively by the shell. The number of YARN containers can be controlled by the parameter `-n <arg>`. The shell deploys a new Flink cluster on YARN and connects the cluster. You can also specify options for YARN cluster such as memory for JobManager, name of YARN application, etc.. 

## Adding external dependencies

It is possible to add external classpaths to the Scala-shell. These will be sent to the Jobmanager automatically alongside your shell program, when calling execute.

Use the parameter `-a <path/to/jar.jar>` or `--addclasspath <path/to/jar.jar>` to load additional classes.

~~~bash
bin/start-scala-shell.sh [local | remote <host> <port> | yarn] --addclasspath <path/to/jar.jar>
~~~
