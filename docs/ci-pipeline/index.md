---
title: Connectivity CI Pipeline
---

The connectivity continuous integration (CI) pipeline can test TACO files across a broad matrix of environmental variables, including multiple operating systems, databases, driver versions, and Tableau versions. It allows you to debug on machines where errors happen and provides messages to identify and help debug specific issues.

The Connectivity CI pipeline lets you:

* Use distributed test suites across machines. 
    Each individual machine is running a very specific configuration, making diagnostics easier.
* Run each test suite in a clean environment.
* Test a broad matrix of operating systems, databases, drivers, Tableau versions, and TACO files.
* Run and get results from the Tableau Data Source Verification Tool (TDVT) test suite within 10 minutes. This gives you quick feedback on the relative health of your TACO file. 
    *Note:* Some TDVT tests take longer than 10 minutes. 
* Bubble up actionable log messages.
* Save the machine state when there’s a failure to help with diagnostics.
* Test partner-built connectors on demand.
    <Reviewers: Do we mean “partner-built connectors” in this case? That is how we refer to connectors submitted thru the gallery.>
* Analyze test results in Tableau. This allows the developer to extrapolate things such as: prioritizing how to fix failing tests, identifying longest running test suite, detecting test cases failing across all operating systems, and comparing runs across various driver versions.
* Isolate partner test results in a separate database.

# Connectivity CI pipeline flow

[Image: Connectivity CI Pipeline Flow.jpg]
## Entry points

The pipeline has two entry points:

* New builds
    When there is a new build from Team City or other automatic trigger, this is dropped in to Amazon S3. You can also manually trigger the pipeline.
    <need more detail about what “dropped in to S3” means>
* Partner builds submitted by GitHub actions.

## The flow

The Connectivity CI Pipeline flow has four stages:

1. *Job scheduling*: This is triggered by a configuration matrix, which identifies what needs testing and what specific configuration for each is needed. The job scheduler creates and queues jobs for the required tests.
2. *Virtual machine provisioning*: These machines are set up based on the configurations defined in the matrix.
3. *Artifacts setup*: This pulls in artifacts (including Perforce, S3, Artifactory, and so on) from various locations and sets up the machine for testing.
    The pipeline is designed to be flexible and allows other frameworks to be added in the future.
    *Note:* Today, only TDVT is being tested. Other frameworks, such as EPS, can be added later.
4. *Test runner*: Runs the test cases against a specific database, located either locally for internal Tableau tests, or remotely for an external partner. 
    *Note*: We have the ability to use docker containers in the future that would be localized to the machine.

## Connectivity CI pipeline results

The results are collected in an Amazon S3 bucket. From there, they are processed and added to the pipeline’s results database. The database captures all the data points you need, including individual details about each test case. These details are for each of the test runs and failures, not just information at the test suite level.

# Database interface

You can monitor everything in the pipeline to get a full status of all activity. For example, you can see which jobs are scheduled, what the status is, and which jobs have failed. All results are captured in the Connectivity CI database. Partner details are copied to an individual, isolated database to ensure privacy.
<talk about the workbook associated with each run and provided to partners>

# Error handling and log monitoring

Information gets routed to various places, including Slack and email, as appropriate.
<need more details here>
For example:
[Image: Connectivity CI Pipeline email.jpg]Here are all of the test results and logs that result in Amazon S3: <is this useful and/or necessary? should this be a table with a defintion for each instead?>
[Image: Connectivity CI Pipeline logs.jpg]
# GitHub repos for partners

Each partner has a GitHub repo for the CI Pipeline. Each repo has a specific folder file structure where the partners locate all of their resources: TACO files, related drivers, a script to install the driver, TDS files that contain credentials to access their database, etc. 

Partners use a config file to configure a specific run. If they want to test various version of their configuration, they can maintain several config files to define each unique set of options. 

When partners are ready to run a test using the Connectivity CI Pipeline, they can add a release tag to the GitHub repo to trigger a GitHub action. In turn, the GitHub action copies all of that partner’s resources into an S3 bucket for use by the pipeline.





















Tableau has great connectivity that allows you to visualize data from virtually anywhere. Tableau includes dozens of connectors already, and also gives you the tools to build a new connector with the Tableau Connector SDK.

With this SDK, you can create a new connector that you can use to visualize your data from any database through an ODBC or JDBC driver.
You can customize connector behavior, fine-tune SQL generation, use the connectivity test harness to validate the connector behavior during the development process, and then package and distribute the connector to users.

# What is a Tableau connector?

A connector is a set of files that describe:

- UI elements needed to collect user input for creating a connection to a data source
- Any dialect or customizations needed for the connection
- How to connect using the ODBC or JDBC driver

A connector can have most of the same features that any built-in Tableau connector supports, including publishing to a server if the server has the connector, creating extracts, data sources, vizzes, and so on.

A connector developed using this SDK is appropriate for connecting to an ODBC or JDBC driver which interfaces using SQL. The underlying technology works well with relation databases.

See the relationship between the connector files (in blue) and the Tableau **Connect** pane and connection dialog:

![]({{ site.baseurl }}/assets/files-overview.png)

# Why build a connector?

You can user the "Other Databases (ODBC)" and "Other Databases (JDBC)" connectors to connect to your database. The Tableau Connector SDK is similar, but offers the following advantages:
- Better live query support. You can customize the dialect used to generate SQL queries so they are compatabile and optimized for your database. The Other Database connectors rely on higher level standard SQL which may not always be appropriate.
- Simpler connection experience. An SDK connector can provide its own customized dialect and you do not need to rely on using DSNs. Users will not need to enter in obscure JDBC URL strings or create a DSN or configure odbc.ini files. Your connector can provide a simple customized connection dialog.
- Runs in Tableau Desktop and Tableau Server. No configuration is required once you install the connector.

If your data source does not fit the relational ODBC/JDBC model, then it may be worth looking into [Web data connectors](https://tableau.github.io/webdataconnector).

# What is a TACO file?
A TACO file (.taco)  is a packaged Tableau connector file that can be placed in your "My Tableau Repository/Connectors" folder. From there, Tableau automatically loads all connectors it finds.

For more information about packaging your connector into a TACO, see [Package and sign your connector for distribution]({{ site.baseurl }}/docs/package-sign)

# Overview of the process

These are the general steps you will follow to create a fully functional connector.

1. Have a look at one of the sample connectors located in the [postgres_odbc or postgres_jdbc folder](https://github.com/tableau/connector-plugin-sdk/tree/master/samples/plugins). These connectors can make a good starting point if you copy the connector files to your workspace.

2. Customize the connector files as needed to name your connector and allow it to connect to your database. See the [Example]({{ site.baseurl }}/docs/example) for more information.

3. Make sure your connector has all the required files:
> * __Manifest file__. This defines the connector.
> * __Connection resolver (ODBC-based connectors only)__. ODBC connectors should include a driver-resolver element. JDBC connectors do not currently support the driver-resolver.
> * __Connection builder JavaScript file__. JDBC connectors can make use of a properties builder JavaScript file.
> * __Dialect definition file__.
> * __Connection dialog__.

4. Once your connector is able to connect, start running the test tool [TDVT]({{ site.baseurl }}/docs/tdvt) to verify your connector is compatible with Tableau. Load the test data into your database, [for example](https://github.com/tableau/connector-plugin-sdk/blob/master/tests/datasets/TestV1/postgres/README.md).

5. When the TDVT tests are passing you are ready to [package and sign your connector]({{ site.baseurl }}/docs/package-sign).

## Prerequisites for development

To develop connectors, be sure you have the following installed on your computer:
- Windows or Mac operating system
- Tableau Desktop or Tableau Server 2019.2 or higher
- __Note:__ Tableau 2019.4 is required to run TACO files
- Python 3.7 or higher
- An ODBC or JDBC data source and driver that meets the requirements listed [here]({{ site.baseurl }}/docs/drivers)
- The provided test data loaded in your data source
- JDK 8 or higher

## Install the SDK tools
Install the following:
 - __TDVT.__ This is our test harness. See the "Installation" section of [Test Your Connector Using TDVT]({{ site.baseurl }}/docs/tdvt) for details.
 - __The packaging tool__. See the "Set up the virtual environment for packaging and signing" section of [Package and sign your connector for distribution]({{ site.baseurl }}/docs/package-sign) for details.

The resulting connector will work on Tableau Desktop and Tableau Server on Windows, Linux, and Mac.

# Using a connector

## Packaged connector (TACO)
Place your packaged TACO file in the My Tableau Repository/Connectors folder and launch Tableau. See [Run your packaged connector (.taco)]({{ site.baseurl }}/docs/run-taco) for more information.

Note: Support for loading TACO files was added in the 2019.4 release of Tableau.

## Developer path
You can tell Tableau to load unpackaged connectors with a special command-line argument that tells Tableau where to find your connector. See [Run Your "Under Development" Connector]({{ site.baseurl }}/docs/run-taco#run-your-under-development-connector) for more information.
