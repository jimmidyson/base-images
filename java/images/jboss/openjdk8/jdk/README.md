## Fabric8 Java Base Image OpenJDK 8 (JDK)

This image is based on JBoss with OpenJDK and provides
OpenJDK 8 (JDK)

It includes:


*  A [Jolokia](http://www.jolokia.org) agent. See below how to configure this.


* A startup script [`/app/run-java.sh`](#startup-script-run-javash) for starting up Java applications.

### Agent Bond

In order to enable Jolokia for your application you should use this 
image as a base image (via `FROM`) and use the output of `agent-bond-opts` in 
your startup scripts to include it in for the Java startup options. 

For example, the following snippet can be added to a script starting up your 
Java application

    # ...
    export JAVA_OPTIONS="$JAVA_OPTIONS $(agent-bond-opts)"
    # .... use JAVA_OPTIONS when starting your app, e.g. as Tomcat does

The following versions and defaults are used:

* [Jolokia](http://www.jolokia.org) : version **undefined** and port **8778**
* [jmx_exporter](https://github.com/prometheus/jmx_exporter): version **undefined** and port **9779**  

You can influence the behaviour of `agent-bond-opts` by setting various environment 
variables:

#### Jolokia configuration

* **AB_JOLOKIA_OFF** : If set disables activation of Joloka (i.e. echos an empty value). By default, Jolokia is enabled.
* **AB_JOLOKIA_CONFIG** : If set uses this file (including path) as Jolokia JVM agent properties (as described 
  in Jolokia's [reference manual](http://www.jolokia.org/reference/html/agents.html#agents-jvm)). 
  By default this is `/opt/jolokia/jolokia.properties`. 
* **AB_JOLOKIA_HOST** : Host address to bind to (Default: `0.0.0.0`)
* **AB_JOLOKIA_PORT** : Port to use (Default: `8778`)
* **AB_JOLOKIA_USER** : User for authentication. By default authentication is switched off.
* **AB_JOLOKIA_PASSWORD** : Password for authentication. By default authentication is switched off.
* **AB_JOLOKIA_ID** : Agent ID to use (`$HOSTNAME` by default, which is the container id)
* **AB_JOLOKIA_OPTS**  : Additional options to be appended to the agent opts. They should be given in the format 
  "key=value,key=value,..."

Some options for integration in various environments

* **AB_JOLOKIA_AUTH_OPENSHIFT** : Switch on OAuth2 authentication for OpenShift. The value of this parameter must be the OpenShift API's 
  base URL (e.g. `https://localhost:8443/osapi/v1/`)



### Startup Script /run-java.sh

The default command for this image is `/run-java.sh`. Its purpose it
to fire up Java applications which are provided as fat-jars, including
all dependencies or more classical from a main class, where the
classpath is build up from all jars within a directory.x1

The run script can be influenced by the following environment variables:

* **JAVA_APP_DIR** the directory where all JAR files can be
  found. This is `/app` by default.
* **JAVA_WORKDIR** working directory from where to start the JVM. By
  default it is `$JAVA_APP_DIR`
* **JAVA_OPTIONS** options to add when calling `java`
* **JAVA_MAIN_CLASS** A main class to use as argument for `java`. When
  this environment variable is given, all jar files in `$JAVA_APP_DIR`
  are added to the classpath as well as `$JAVA_APP_DIR` and
  `JAVA_WORKDIR` themselves, too.
* **JAVA_APP_JAR** A jar file with an appropriate manifest so that it
  can be started with `java -jar` if no `$JAVA_MAIN_CLASS` is set. In all
  cases this jar file is added to the classpath, too.
* **JAVA_APP_NAME** Name to use for the process
* **JAVA_CLASSPATH** the classpath to use. If not given, the script checks 
  for a file `${JAVA_APP_DIR}/classpath` and use its content literally 
  as classpath. If this file doesn't exists all jars in the app dir are 
  added (`classes:${JAVA_APP_DIR}/*`). 
* **JAVA_ENABLE_DEBUG** If set remote debugging will be switched on
* **JAVA_DEBUG_PORT** Port used for remote debugging. Default: 5005


If neither `$JAVA_APP_JAR` nor `$JAVA_MAIN_CLASS` is given,
`$JAVA_APP_DIR` is checked for a single JAR file which is taken as
`$JAVA_APP_JAR`. If no or more then one jar file is found, the script
throws an error. 

The classpath is build up with the following parts:

* If `$JAVA_CLASSPATH` is set, this classpath is taken.
* The current directory (".") is added first.
* If the current directory is not the same as `$JAVA_APP_DIR`, `$JAVA_APP_DIR` is adsed. 
* If `$JAVA_MAIN_CLASS` is set, then 
  - A `$JAVA_APP_JAR` is added if set
  - If a file `$JAVA_APP_DIR/classpath` exists, its content is appended to the classpath. 
  - If this file is not set, a `${JAVA_APP_DIR}/*` is added which effectively adds all 
    jars in this directory in alphabetical order. 

These variables can be also set in a
shell config file `run-env.sh`, which will be sourced by 
the startup script. This file can be located in the directory where 
this script is located and in `${JAVA_APP_DIR}`, whereas environment 
variables in the latter override the ones in `run-env.sh` from the script 
directory.

This script also checks for a command `run-java-options`. If existant it will be
called and the output is added to the environment variable `$JAVA_OPTIONS`.

Any arguments given during startup are taken over as arguments to the
Java app. 

### Versions:

* Base-Image: **JBoss with OpenJDK undefined**
* Java: **OpenJDK 8 1.8.0** (Java Development Kit (JDK))
* Jolokia: **1.3.1**