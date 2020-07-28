# Sqoop

## 1. Introduction

<font color="#aa0">Sqoop</font> is a tool designed to transfer data between Hadoop and relational databases or mainframes. 

## 2. Basic usage

With <font color="#aa0">Sqoop</font>, you can <i>import</i> data from a relational database system or a mainframe into HDFS. The input to the import process is either database table or mainframe datasets. For databases, <font color="#aa0">Sqoop</font> will read the table <font color="#a00">row-by-row</font> into HDFS. For mainframe datasets, <font color="#aa0">Sqoop</font> will read records from each mainframe datasets into HDFS. The output s a set of file containing a copy of the imported table or datasets. The import process is performed <font color="#a00">in parallel</font>. 

<font color="#aa0">Sqoop's</font> <i>export</i> process will read a set of delimited text files from HDFS <font color="#a00">in parallel</font>, parse them into records, and inserts them as a new row in a target database table, for consumption by external applications or users.

Most aspects of the <i>import, code generation, and export</i> processes can be customized.

## 3. Sqoop Tools

<font color="#aa0">Sqoop</font> is a collection of related tools. To use <font color="#aa0">Sqoop</font>, you specify the <font color="#a00">tool</font> you want to use and the <font color="#a00">arguments</font> that control the tool.

<b>Syntax</b>: <i><b>sqoop tool-name [tool-arguments]</b></i>

### 3.1 Using Command Aliases

In addition to typing the <font color="#aa0">sqoop (tool-name) </font>syntax, you can use alias scripts that specify the <font color="#aa0">sqoop-(tool-name)</font> syntax.

### 3.2 Controlling Hadoop Installation

The <font color="#aa0">sqoop</font> command-line program is a <b>wrapper</b> which runs the <font color="#aa0">bin/hadoop</font> script shipped with Hadoop. If you have multiple installations of Hadoop present on your machine, you can select the Hadoop installation by setting the <font color="#aa0">$HADOOP_COMMON_HOME</font> and <font color="#aa0">$HADOOP_MAPRED_HOME</font> environment variables. 

If either of these variables are not set, <font color="#aa0">Sqoop</font> will fall back to the <font color="#aa0">$HADOOP_HOME</font>.

The active Hadoop configuration is loaded from <font color="#aa0">$HADOOP_HOME/conf/</font>, unless the <font color="#aa0">$HADOOP_CONF_DIR</font> environment variable is not set.

### 3.3 Using Generic and Specific arguments

You must supply the generic arguments <font color="#aa0">-conf, -D, and so on</font> after the tool name but <b>before</b> any tool-specific arguments (such as <font color="#aa0">--connection</font>).

Note that generic Hadoop arguments are preceded by a single dash character (<font color="#aa0">-</font>), whereas tool-specific arguments start with two dashes (<font color="#aa0">--</font>), unless they are single character arguments such as <font color="#a00">-P</font>.

### 3.4 Using Options files to parse arguments

When using <font color="#aa0">Sqoop</font>, the command line options that do not change from invocation to invocation can be put in an options file for convenience. An options file is a text file where each line identifies an option in the order that it appears otherwise on the command line.

To specify an options file, simply create an options file in a convenient location and pass it to the command line via <font color="#aa0">--options-file</font> argument.

## 4. Sqoop import

### 4.1 purpose

The <i>import</i> tool imports an individual table from an RDBMS to HDFS.

### 4.2 Syntax

common arguments:

| Argument              | Description                                                  |
| --------------------- | ------------------------------------------------------------ |
| --connect <jdbc-url>  | Specify JDBC connect String                                  |
| --username <username> | set authentication username                                  |
| -P                    | Read password from console prompt                            |
| --password-file       | set the path for a file containing the authentication password |
| --driver <class-name> | manually specify the JDBC driver class to user               |
| --relaxed-isolation   | set the connection transaction isolation to read uncommitted for the mappers |

<b>Secure way of supplying password to the database</b>,

- save the password in a file on the users home directory with 400 permissions and specify the path to that file using the <font color="#aa0">--password-file</font> argument.
- using <font color="#aa0">-P</font> argument to read password from console prompt

<b>Validation arguments more details</b>

| Argument                                 | Description                                                  |
| ---------------------------------------- | ------------------------------------------------------------ |
| --validate                               | enable validation of data copied, supports single table copy only |
| --validator <class-name>                 | specify the validator class to use                           |
| --validation-threshold <class-name>      | specify the validation threshold class to use                |
| --validation-failurehandler <class-name> | specify the validation failure handler class to use          |

#### 4.2.1 Selecting the data to import

<font color="#aa0">Sqoop</font> typically imports data in a table-centric fashion. Use the <font color="#aa0">--table</font> argument to select the table to import. By default, all columns within a table are selected for import.

By default, sqoop will use query <font color="#a00"><i>select min(<split-by>), max(<split-by>) from <table name></i></font> to find out boundaries for <font color="#a00"><i>creating splits</i></font>.

#### 4.2.2 Free-form query

You can specify a SQL statement with the <font color="#aa0">--query</font> argument.

When importing a free-form query, you <b>MUST</b> specify a destination directory with <font color="#aa0">--target-dir</font>.

If you want to import the results of a query in <font color="#a00">parallel</font>, then each map task will need to execute a copy of the query, with results partitioned by bounding conditions inferred by <font color="#aa0">Sqoop</font>. Your query <b>MUST</b> include the token <font color="#aa0">$CONDITIONS</font> which each Sqoop process will replace with a <font color="#a00">unique condition expression</font>. You <b>MUST</b> also select a splitting column with <font color="#aa0">--split-by</font>.

> <b>Note</b>
>
> The facility of using free-form query in the current version of Sqoop is limited to simple queries where there are no ambiguous projections and  no <font color="#aa0">OR</font> conditions in the <font color="#aa0">WHERE</font> clause.

#### 4.2.3 Controlling Parallelism

Sqoop imports data in parallel from most database sources. You can specify the number of map tasks (<font color="#aa0">parallel processes</font>) to use to perform the import by using the <font color="#aa0">-m</font> or <font color="#aa0">--num-mappers</font> argument. Each of these arguments takes an integer value which corresponds to the degree of parallelism to employ. By default, it's <font color="#a00">four</font>. Some will increase it to <font color="#a00">8</font> or <font color="#a00">16</font>.

Using the <font color="#aa0">--split-limit</font> parameter places a limit on the size of the split section created. If the size of split created is larger than the size specified in this parameter, then the splits would be resized to fit within this limit, and the number of splits will change according to that. This affects the actual number of mappers. 

If the size of a split calculated based on provided <font color="#aa0">--num-mappers</font> parameter exceeds <font color="#aa0">--split-limit</font> parameter the actual number of mappers will be increased.

If the <font color="#aa0">--split-limit</font> parameter is 0 or negative, the parameter will be <u><b>ignored</b></u> altogether.

#### 4.2.4 Controlling transaction isolation

By default, Sqoop uses the <font color="#aa0">read committed</font> transaction isolation in the mappers to import data. The <font color="#aa0">--relaxed-isolation</font> option can be used to instruct Sqoop to use read uncommitted isolation level.

#### 4.2.5 Incremental Imports

Sqoop provides an incremental import mode which can be used to retrieve only <font color="#aa0">rows newer</font> than some previously-imported set of rows.

<b>Arguments</b>:

| Argument             | Description                                                  |
| -------------------- | ------------------------------------------------------------ |
| --check-column (col) | Specifies the column to be examined when determining which rows to import |
| --incremental (mode) | Specifies how Sqoop determines which rows are new. <font color="#aa0">append</font> and <font color="#aa0">lastmodified</font>. |
| --last-value (value) | Specifies the maximum value of the check column from the previous import. |

Two types of incremental imports:

- <u><b>append</b></u>: used when importing a table where new rows are <font color="#a00">continually</font> being added with increasing row id values. Sqoop imports rows where the check column has a value greater than the one specified with <font color="#aa0">--last-value</font>.
- <u><b>lastmodified</b></u>: used when rows of the source table may be <font color="#a00">updated</font>, and each such update will set the value of a last-modified column to be the current timestamp.

## 5. Sqoop import all tables

### 5.1 Purpose

The <font color="#aa0">import-all-tables</font> tool imports a set of tables from an RDBMS to HDFS. Data from each table is stored in a separate directory in HDFS.

The following conditions <b>MUST</b> be met:

- Each table <b>MUST</b> have a single-column primary key or <font color="#aa0">--auto-to-one-mapper</font> option <b>MUST</b> be used
- <b>MUST</b> intend to import all columns of each table
- <b>MUST</b> not intend to use non-default splitting column, nor impose any conditions via a <font color="#aa0">WHERE</font> clause.

### 5.2 Syntax

```shell
sqoop import-all-tables (generic-arguments) (import-arguments)
sqoop-import-all-tables (generic-arguments) (import-arguments)
```

## 6. Sqoop Export

The <font color="#aa0">export</font> tool exports a set of files from HDFS back to an RDBMS. The target table <b>MUST</b> already exist in the database.

The default operation is to transform these into a set of <font color="#aa0">INSERT</font> statements that inject the records into the database. In "<font color="#aa0">update</font> mode", Sqoop will generate <font color="#aa0">UPDATE</font> statements that replace existing records in the database, and in "<font color="#aa0">call</font> mode" Sqoop will make a stored procedure call for each record.

### 6.1 Syntax

``` shell
sqoop export (generic arguments) (export arguments)
sqoop-export (generic arguments) (export arguments)
```

The <font color="#aa0">--export-dir</font> argument and one of <font color="#aa0">--table</font> or <font color="#aa0">--call</font> are required. These specify the table to populate in database (or the stored procedure to call), and the directory in HDFS that contains the source data.

Since Sqoop breaks down export process into <u><font color="#aa0">multiple transactions</font></u>, it is possible that a failed export job may result in <font color="#a00">partial data being committed</font> to the database. This can further lead to subsequent jobs failing due to insert collisions in some cases, or lead to duplicated data in others. You can <u><b>overcome this problem by specifying a staging table via the <font color="#aa0">--staging-table</font> option</b></u> which acts as an auxiliary table that is used to stage exported data. The staged data is finally moved to the destination table in <u>a single transaction</u>.

#### 6.1.1 Inserts & Updates

If you specify the <font color="#aa0">--update-key</font> argument, Sqoop will instead modify an existing dataset in the database. Each input record is treated as an <font color="#aa0">UPDATE</font> statement that modifies an existing row. 

If an <font color="#aa0">UPDATE</font> statement modifies no rows, this is not considered an error; the export will silently continue.

Depending on the target database, you may also specify the <font color="#aa0">--update-mode</font> argument with <font color="#aa0">allowinsert</font> mode if you want to update rows if they exist in the database already or insert rows if they do not exist yet.

### 6.2 Failed exports

Exports may fail for a number of reasons:

- Loss of connectivity from the Hadoop cluster to the database
- Attempting to <font color="#aa0">INSERT</font> a row which violates a consistency constraint
- Attempting to parse an incomplete or malformed record from the HDFS source data
- Attempting to parse records using incorrect delimiters
- Capacity issues (such as insufficient RAM or disk space)

## 7. validation

### 7.1 Purpose

Validate the data copied, either import or export by <font color="#aa0">comparing the row counts</font> from the source and the target post copy.

### 7.2 Introduction

3 basic interfaces:

- <b><u>ValidationThreshold</u></b>, determines if the <font color="#aa0">error margin</font> between the source and target are acceptable: Absolute, Percentage Tolerant, etc.
- <u><b>ValidationFailureHandler</b></u>, responsible for handling failures: log an error/warning, abort, etc.
- <b><u>Validator</u></b>, drives the validation logic by delegating the decision to <font color="#aa0">ValidationThreshold</font> and delegating failure handling to <font color="#aa0">ValidationFailureHandler</font>.

### 7.3 Limitations

Validation currently only <u>validates data copied from a single table</u> into HDFS. The following are the limitations in the current implementation:

- all-tables option
- free-form query option
- Data imported to Hive, HBase or Accumulo
- table import with --where argument
- incremental imports

## 8. Saved Jobs

Imports and exports can be repeatedly performed by <u>issuing the same command multiple times</u>. Especially when using the incremental import capacity, this is an expected scenario.

By default, job descriptions are saved to a private repository stored in <font color="#aa0">$HOME/.sqoop/</font>. You can configure Sqoop to instead use a shared <i>metastore</i>, which makes saved jobs available to multiple users across a shared cluster.

## 9. sqoop-job

### 9.1 Purpose

Allows you to <font color="#aa0">create and work</font> with saved jobs. Saved jobs remember the parameters used to specify a job, so they can be re-executed by invoking the job by its handle.

### 9.2 Syntax

By default, a private meta-store is instantiated in <font color="#aa0">$HOME/.sqoop</font>. If you have configured a hosted meta-store with the <font color="#aa0">sqoop-metastore</font> tool, you can connect to it by specifying the <font color="#aa0">--meta-connect</font> argument.

In <font color="#aa0">conf/sqoop-site.xml</font>, you can configure <font color="#aa0">sqoop.metastore.client.autoconnect.url</font> with this address, so you do not have to supply <font color="#aa0">--meta-connect</font> to use a remote meta-store.

If you configure <font color="#aa0">sqoop.metastore.client.enable.autoconnect</font> with the value <font color="#aa0">false</font>, then you must explicitly supply <font color="#aa0">--meta-connect</font>.

### 9.3 Saved jobs and passwords

The Sqoop meta-store is <b>NOT</b> a secure resource.

You can enable passwords in the meta-store by setting <font color="#aa0">sqoop.metastore.client.record.password</font> to <font color="#aa0">true</font> in the configuration.

><b>Note</b>
>
>You have to set <font color="#aa0">sqoop.metastore.client.record.password</font> to <font color="#aa0">true</font> if you are executing saved jobs via Oozie.