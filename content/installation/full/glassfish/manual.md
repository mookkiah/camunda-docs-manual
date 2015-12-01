---

title: 'Install the Full Distribution on a Glassfish Application Server manually'
weight: 20

menu:
  main:
    name: "Manual Installation"
    identifier: "installation-guide-full-glassfish-install-manually"
    parent: "installation-guide-full-glassfish"
    pre: "Install and configure the Full Distribution on a vanilla Glassfish Application Server."

---


This section will describe how you can install the Camunda BPM platform and its components on a vanilla [Glassfish 3.1](http://glassfish.java.net/), if you are not able to use the pre-packaged Glassfish distribution. Regardless, we recommend that you [download a Glassfish 3.1 distribution](http://camunda.org/download/) to use the required modules.

{{< note title="Reading this Guide" class="info" >}}
Throughout this guide we will use a number of variables to denote common path names and constants.

* `$GLASSFISH_HOME` points to the glassfish application server main directory (typically `glassfish3/glassfish` when extracted from a glassfish distribution).
* `$PLATFORM_VERSION` denotes the version of the Camunda BPM platform you want to install or already have installed, e.g. `7.0.0`.
* `$GLASSFISH_DISTRIBUTION` represents the downloaded pre-packaged Camunda BPM distribution for Glassfish, e.g. `camunda-bpm-glassfish-$PLATFORM_VERSION.zip` or `camunda-bpm-glassfish-$PLATFORM_VERSION.tar.gz`.

{{< /note >}}


# Setup

Before you can install the Camunda components, you need to perform a number of required setup steps.


## Create the Database Schema

By default, the database schema is automatically created in an H2 database when the engine starts up for the first time. If you do not want to use the H2 database, you first have to create a database schema for the Camunda BPM platform. The Camunda BPM distribution ships with a set of SQL create scripts that can be executed by a database administrator.

The database creation scripts are reside in the `sql/create` folder:

`$GLASSFISH_DISTRIBUTION/sql/create/*_engine_$PLATFORM_VERSION.sql`
`$GLASSFISH_DISTRIBUTION/sql/create/*_identity_$PLATFORM_VERSION.sql`

There is an individual SQL script for each supported database. Select the appropriate script for your database and run it with your database administration tool. (e.g., SqlDeveloper for Oracle).

When you create the tables manually, then you can also configure the engine to **not** create tables at startup by setting the `databaseSchemaUpdate` property to `false` (or, in case you are using Oracle, to `noop`). In GlassFish, this is done in the `bpm-platform.xml`, located in the `$GLASSFISH_HOME\glassfish\domains\domain1\applications\camunda-bpm-platform\camunda-glassfish-service-VERSION.jar\META-INF\` folder.

{{< note title="Heads Up!" class="info" >}}
If you have defined a specific prefix for the entities of your database, then you will have to manually adjust the `create` scripts accordingly so that the tables are created with the prefix.
{{< /note >}}


## Configure a JDBC Connection Pool

The JDBC Connection Pool and the JDBC Resource can be configured by editing the file `domain.xml` inside the folder `$GLASSFISH_HOME/glassfish/domains/<domain>/config/`.

This could look like the following example for an H2 database:

```xml
<domain>
  ...
  <resources>
    ...
    <jdbc-resource pool-name="ProcessEnginePool"
                   jndi-name="jdbc/ProcessEngine"
                   enabled="true">
    </jdbc-resource>

    <jdbc-connection-pool is-isolation-level-guaranteed="false"
                          datasource-classname="org.h2.jdbcx.JdbcDataSource"
                          res-type="javax.sql.DataSource"
                          non-transactional-connections="true"
                          name="ProcessEnginePool">
      <property name="Url"
                value="jdbc:h2:./camunda-h2-dbs/process-engine;DB_CLOSE_DELAY=-1;MVCC=TRUE;DB_CLOSE_ON_EXIT=FALSE">
      </property>
      <property name="User" value="sa"></property>
      <property name="Password" value="sa"></property>
    </jdbc-connection-pool>
  </resources>

  <servers>
    <server>
      ...
      <resource-ref ref="jdbc/ProcessEngine"></resource-ref>
    </server>
  </servers>
</domain>
```

In case another database than H2 is used (i.e., DB2, MySQL etc.), you have to adjust the `datasource-classname` and the `res-type` attributes with the corresponding database classes and set the database specific properties (such as the url, etc.) inside the JDBC Connection Pool. Furthermore, you have to add the corresponding JDBC driver to `$GLASSFISH_HOME/glassfish/lib/`. For example, you can add the H2 JDBC driver which is located at `$GLASSFISH_DISTRIBUTION/server/glassfish3/glassfish/lib/h2-VERSION.jar` to run with the H2 database.


## Configure a Thread Pool for the Job Executor

To do so, you have to edit the file `$GLASSFISH_HOME/glassfish/domains/<domain>/config/domain.xml` and add the following elements to the `resources` section.

```xml
<domain>
  ...
  <resources>
    ...
    <resource-adapter-config
      enabled="true"
      resource-adapter-name="camunda-jobexecutor-rar"
      thread-pool-ids="platform-jobexecutor-tp" >
    </resource-adapter-config>

    <connector-connection-pool
        enabled="true"
        name="platformJobExecutorPool"
        resource-adapter-name="camunda-jobexecutor-rar"
        connection-definition-name=
            "org.camunda.bpm.container.impl.threading.jca.outbound.JcaExecutorServiceConnectionFactory"
        transaction-support="NoTransaction" />

    <connector-resource
        enabled="true"
        pool-name="platformJobExecutorPool"
        jndi-name="eis/JcaExecutorServiceConnectionFactory" />
  </resources>

  <servers>
    <server>
      ...
      <resource-ref ref="eis/JcaExecutorServiceConnectionFactory"></resource-ref>
    </server>
  </servers>
</domain>
```

To configure a thread pool for the job executor you have to add it to the corresponding `config` elements of `domain.xml`.

```xml
<domain>
  ...
  <configs>
    ...
    <config name="server-config">
      ...
      <thread-pools>
        ...
        <thread-pool max-thread-pool-size="6"
                     name="platform-jobexecutor-tp"
                     min-thread-pool-size="3"
                     max-queue-size="10">
        </thread-pool>
      </thread-pools>
    </config>
  </configs>
</domain>
```


# Required Components

The following steps are required to deploy the Camunda BPM platform:

1. Merge the shared libraries from `$GLASSFISH_DISTRIBUTION/modules/lib` into the `GLASSFISH_HOME/glassfish/lib` directory (i.e., copy the content into the Glassfish library directory).
2. Copy the job executor resource adapter `$GLASSFISH_DISTRIBUTION/modules/camunda-jobexecutor-rar-$PLATFORM_VERSION.rar` into `$GLASSFISH_HOME/glassfish/domains/<domain>/autodeploy`. The job executor resource adapter has to be deployed first because the artifact `camunda-glassfish-ear-$PLATFORM_VERSION.ear` depends on it and cannot be deployed successfully without the resource adapter. If you try to deploy both components with the auto-deploy feature in one step you should be aware that the deployment order is not defined in this case. Due to this, we propose to startup the Glassfish application server to initially deploy the job executor resource adapter. After a successful startup, shutdown the Glassfish application server.
3. Copy the artifact `$GLASSFISH_DISTRIBUTION/modules/camunda-glassfish-ear-$PLATFORM_VERSION.ear` into `$GLASSFISH_HOME/glassfish/domains/<domain>/autodeploy`.
4. (optional) [Configure the location of the bpm-platform.xml file]({{< relref "reference/deployment-descriptors/descriptors/bpm-platform-xml.md#configure-location-of-the-bpm-platform-xml-file" >}}).
5. Startup the Glassfish application server.
6. After a successful startup, the Camunda BPM platform is installed.


# Optional Components

This section describes how to install optional Camunda dependencies onto a Glassfish server. None of these are required to work with the core platform.


## Cockpit and Tasklist

The following steps are required to deploy the applications:

1. Download the Camunda web application that contains both applications from our [Maven Nexus Server](https://app.camunda.com/nexus/content/groups/public/org/camunda/bpm/webapp/camunda-webapp-glassfish/).
   Or switch to the private repository for the enterprise version (User and password from license required).
   Choose the correct version named `$PLATFORM_VERSION/camunda-webapp-glassfish-$PLATFORM_VERSION.war`.
2. Optionally, you may change the context path to which the application will be deployed (default is `/camunda`).
   Edit the file `WEB-INF/sun-web.xml` in the war file and update the `context-root` element accordingly.
2. Copy the war file to `$GLASSFISH_HOME/domains/domain1/autodeploy`.
3. Startup the Glassfish Application Server.
4. Access Cockpit and Tasklist via `/camunda/app/cockpit` and `/camunda/app/tasklist` or under the context path you configured.


## REST API

The following steps are required to deploy the REST API:

1. Download the REST API web application archive from our [Maven Nexus Server](https://app.camunda.com/nexus/content/groups/public/org/camunda/bpm/camunda-engine-rest/).
   Or switch to the private repository for the enterprise version (User and password from license required).
   Choose the correct version named `$PLATFORM_VERSION/camunda-engine-rest-$PLATFORM_VERSION.war`.
2. Optionally, you may change the context path to which the REST API will be deployed (default is `/engine-rest`).
   Edit the file `WEB-INF/sun-web.xml` in the war file and update the `context-root` element accordingly.
3. Copy the war file to `$GLASSFISH_HOME/domains/domain1/autodeploy`.
4. Startup the Glassfish Application Server.
5. Access the REST API on the context path you configured.
   For example, <a href="http://localhost:8080/engine-rest/engine">http://localhost:8080/engine-rest/engine</a> should return the names of all engines of the platform,
   if you deployed the application in the context `/engine-rest`.


## Camunda Connect

Add the following artifacts (if not existing) from the folder `$GLASSFISH_DISTRIBUTION/modules/lib/` to the folder `$GLASSFISH_HOME/glassfish/lib/`:

* `camunda-connect-core-$CONNECT_VERSION.jar`
* `camunda-commons-utils-$COMMONS_VERSION.jar`

In order to activate Camunda Connect functionality for a process engine, a process engine plugin has to be registered in the BPM platform configuration as follows:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<bpm-platform ... >
  <process-engine name="default">
    ...
    <plugins>
      ... existing plugins ...
      <plugin>
        <class>org.camunda.connect.plugin.impl.ConnectProcessEnginePlugin</class>
      </plugin>
    </plugins>
    ...
  </process-engine>

</bpm-platform>
```


## Camunda Spin

Add the following artifacts (if not existing) from the folder `$GLASSFISH_DISTRIBUTION/modules/lib/` to the folder `$GLASSFISH_HOME/glassfish/lib/`:

* `camunda-spin-core-$SPIN_VERSION.jar`
* `camunda-commons-utils-$COMMONS_VERSION.jar`

In order to activate Camunda Spin functionality for a process engine, a process engine plugin has to be registered in BPM platform configuration as follows:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<bpm-platform ... >
  ...
  <process-engine name="default">
    ...
    <plugins>
      ... existing plugins ...
      <plugin>
        <class>org.camunda.spin.plugin.impl.SpinProcessEnginePlugin</class>
      </plugin>
    </plugins>
    ...
  </process-engine>
  ...
</bpm-platform>
```


## Groovy Scripting

Add the following artifacts (if not existing) from the folder `$GLASSFISH_DISTRIBUTION/modules/lib/` to the folder `$GLASSFISH_HOME/glassfish/lib/`:

* `groovy-all-$GROOVY_VERSION.jar`


## Freemarker Integration

Add the following artifacts (if not existing) from the folder `$GLASSFISH_DISTRIBUTION/modules/lib/` to the folder `$GLASSFISH_HOME/glassfish/lib/`:

* `camunda-template-engines-freemarker-$TEMPLATE_VERSION.jar`
* `freemarker-2.3.20.jar`
* `camunda-commons-logging-$COMMONS_VERSION.jar`
* `camunda-commons-utils-$COMMONS_VERSION.jar`
