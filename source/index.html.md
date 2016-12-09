---
title: Open Bouquet API Reference

language_tabs:
  - shell : cURL

toc_footers:
  - <a href='#'>Sign Up for a Developer Access</a>
  - <a href='https://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---

# Introduction

Welcome to the Open Bouquet API! You can use our API to perform analytics on your data-sources connected to your Open Bouquet server.

We have language bindings in Shell and HTML! You can view code examples in the dark area to the right, and you can switch the programming language of the examples with the tabs in the top right.

# Authentication

> To check if you need to authenticate, use this code for example:

```shell
# With shell, just check the analytics endpoint
curl -X GET "http://yourserverdomain/v4.2/analytics
```
> If you need authentication you'll get the following error, where loginURL is the authentication end-point. You can also use the selfLoginURL to be automatically redirected to the original URL after sign-in.

```json
{"loginURL":"http://yourserverdomain/dev/auth/oauth",
"selfLoginURL":"http://yourserverdomain/dev/auth/oauth?client_id=admin_console
&redirect_uri=https://yourserverdomain/dev/v4.2/analytics?style=HTML",
"code":401,
"error":"Auth failed : invalid access_token",
"type":"InvalidTokenAPIException",
"suppressed":[]}
```

The authentication mechanism depends on the version of OB server you are using.
By default the server authentication is turned-off.

When the authentication is turned on, it uses OAuth2. So you will have to retrieve an authentication token from the authentication server.

Note that by default, if you are not authenticated and you try to access the API, it will throw a 401 error code, with the authentication URL in the error message.

OB expects for the OAuth token to be included in all API requests to the server in a header that looks like the following:

`Authorization: Bearer TOKEN`

For covenience you can also provide the TOKEN as part of the URL, using the `access_token` parameter:

`GET http://yourserverdomain/dev/v4.2/analytics&access_token=TOKEN`

<aside class="notice">
In all example you must replace <code>TOKEN</code> with the OAuth2 token.
</aside>

# Bouquet Concepts

This is a brief overview of the various concepts use by Bouquet to organize information.

## Projects

A Project is the top level object that organize data-source model. First a Project defines how to accessing the data-source, through a JDBC url and required authentication. Also the Project defines which part of the data-source will be made available through the API, by listing visible schemas.

A Project also maintains a list of Domain.

## Domains

A Domain is associated with a table or view in the data-source. A Domain can be created dynamically by Open Bouquet, or explicitly by a user. Also a user can modify a dynamic Domain to make it permanent and persist the changes to the model.

A Domain has Dimension and Metric attributes, which are dynamically mapped to the underlying data-source. A user can also modify any existing attribute, or create new ones. Those changes are then persisted in the model.

### Dimensions

A Dimension is any non-aggregated expression defined on the Domain.

### Metrics

A Metric is any expression defined on the Domain. If the metric is an aggregated expression, it will be use as is. If it is not an aggregated expression, nor an analytics expression, it must be possible to apply an aggregation operator like SUM(), ...

Since a Metric can be any expression, it can contain conditional operator like CASE(), combined with an aggregation operator.

## Relations

A Relation defines how to join two Domains. It is defined by a left and right domain, and a predicate expression between the two. It also specify the relation cardinality.

## Bookmarks

A Bookmark is a simple way to create a custom view on your model. From a Domain, you can define a default analysis, select filters, restrict the dimension and metrics usable for pivoting. Bookmarks are also organized on their own inside folders, independently from the Projects - so you can mix in the same folder Bookmarks from different Projects.

# Analytics API

The Analytics API provides you all you need to interact with your data:

* list available content like Projects, Bookmarks and Domains
* query a specific Bookmark or Domain
* view dataviz for a specific Bookmark or Domain (as VegaLite output)
* explore a Bookmark or Domain scope
* create a Bookmark out of a query

<aside class="notice">
The Analytics API traits Domain and Bookmark mostly the same. Although they are different kind of object, you can query and view them using the same operations. 
</aside>

<aside class="notice">
In order to discover the API feature in a more friendly way, you can use most of the API methods with style=HTML.
In that mode the API will output some simple HTML that allows interaction through forms. It also provides reference to the API parameters and links to related call you may want to do.
</aside>

## List available content

> This is a very simple example demonstrating how you can get the top level content in your account. In that example you don't need to provide any parameter.

```shell
curl -X GET 
	--header 'Accept: application/json' 
	--header 'Authorization: Bearer TOKEN'
	"http://yourserverdomain/v4.2/analytics"
```

This method retrieves all the content available for the authenticated user.

<aside class="notice">
This API method support the style=HTML for interactive exploration.
</aside>

### HTTP Request

`GET http://yourserverdomain/dev/v4.2/analytics`

<aside class="success">
Remember to provide a valid OAuth TOKEN!
</aside>

### URL Parameters

There are no URL parameters for this method.

### Query Parameters

Parameter | Required | Description
--------- | ------- | -----------
parent | false, default=/ | This is the path of a valid parent container. If the path is not defined, or path=/, it will display the top level content. In order to view a child content, use path=<child.selfRef>. For example in order to list all projects, path=/PROJECTS. This is working like a standard Unix path, for example: /PROJECTS/MyProject/ to list MyProject's domains, or /MYBOOKMARKS/some/folder to list bookmarks and folders. See 
q | false | This is a search string that you can use to filter the result. Multiple tokens can be separated by comma.
visibility | false, default=ALL | If visibility is ALL, will show all existing content. If visibility is VISIBLE, it will only show objects that have been made explicitly visible.
style | false, default=HUMAN | defines the output style, check API reference
envelope | false, default=RESULT | defines the output content, check API reference

### Parent parameter

> This is an example demonstrating how you can list the projects available in your account.

```shell
curl -X GET 
    --header 'Accept: application/json' 
    --header 'Authorization: Bearer TOKEN'
    "http://yourserverdomain/v4.2/analytics?parent=/PROJECTS"
```

The parent parameter is a path. Each section of the path identify a valid container, using either its name or immutable ID.

When browsing the content, you can always use a child `selfRef` property to deep dive into it.

#### Top Level content

The top level container are immutable: `/PROJECTS`, `/MYBOOKMARK`, `/SHARED`

* PROJECTS lists all available projects for the authenticated user
* MYBOOKMARK lists all bookmarks created by the authenticated user
* SHARED lists all bookmarks available to the authenticated user

#### Project content

In order to list a project content, you can use it's ID or name. Note that the name can change, so if you need to make a persistent reference to the project, it is better to use its ID (remember that by using the style=HUMAN, the API will return objects by name. If you'd better list by ID, you can use style=ROBOT. Note that in all case you can use aither name or ID t reference ID). So for example to list your project content you can use either:

* path=`/PROJECTS/@MyProjectID/`
* path=`/PROJECTS/My Project/`

Note that in order to list a project content, the OB server will request access to the project's database, and update the content dynamically. So the first time you access it may take some delay depending on the database.

#### Bookmarks

Both the `/MYBOOKMARK` and `/SHARED` paths returns bookmarks. The only difference is the bookmark visibility:

* `/MYBOOKMARK` only returns the bookmarks that the authenticated user has created 
* `/SHARED` returns the bookmarks shared with the authenticated user

Both path returns a list of Bookmarks and Folders.

### Reply

> This is the reply from the previous API call for listing available projects

```json
{
"parent": {
    "name": "Projects",
    "description": "list all your Projects",
    "parentRef": "/",
    "selfRef": "/PROJECTS",
    "type": "FOLDER",
    "link": "http://yourserverdomain/v4.2/analytics?parent=/PROJECTS&style=HUMAN&visibility=ALL&access_token=xxx",
    "upLink": "http://yourserverdomain/v4.2/analytics?parent=/&style=HUMAN&visibility=ALL&access_token=xxx"
    },
"children": [{
    "name": "demo project",
    "attributes": {
        "jdbc": "jdbc:postgreql://databasehost:5432/demo"
    },
    "parentRef": "/PROJECTS",
    "selfRef": "/PROJECTS/demo project",
    "type": "PROJECT",
    "link": "http://yourserverdomain/v4.2/analytics?parent=/PROJECTS/demo project&style=HUMAN&visibility=ALL&access_token=xxx",
    "objectLink": "http://yourserverdomain/v4.2/rs/projects/576002908294a27bac5a6946?style=HUMAN&access_token=xxx"
    }]
}
```

The reply has the following structure.

The `parent` node describe the current location of the `parent` parameter. For instance you could use it to browse upward, because it provides also `parentRef` property (unless it is the root folder).

The `children` node is an array of nodes, and it is the content of the `parent` node.

Each node has the same structure:

Property | Description
-------- | -----------
name |
description | 
parentRef | 
selfRef | 
type | 

The `link` provides you direct navigation into the content hierarchy. If the object type is a container, the link navigate to the content. If the object type cannot be navigated, it is a link to the associated query.

The `objectLink` provides a link to the detailed definition of the object. So it will redirect you to a different API depending on the object type. This API is actually the metadata management API that allows you to discover but also modify the model definition.

<aside class="notice">
Navigation in your browser is easier if you install a json prettify plugin :)
</aside>

## Query a Bookmark or Domain

This method performs a query on a Bookmark or Domain and return the result as a paginated data table.

The result of the query is automatically stored in the server cache, so it is not a performance problem to perform the same query repeatedly, no need to cache the result on the application side. You can also paginate the results to simplify display. Also it is a good practice to set a `limit` on the query, especially for interactive queries. Although the server will paginate the result internally to avoid memory overflow, you may have trouble retrieving the result in JSON. If you really need to extract a large table, prefer using the /export API method which provides streaming results.

It is possible to change the layout of the table using the `data` parameter.

<aside class="notice">
This API method support the style=HTML for interactive exploration.
</aside>

### HTTP Request

`GET http://yourserverdomain/v4.2/analytics/<REFERENCE>/query`

<aside class="success">
Remember to provide a valid OAuth TOKEN!
</aside>

### URL Parameters

Parameter | Required | Description
--------- | -------- | -----------
REFERENCE | true | The identifier of the Bookmark/Domain to retrieve

#### Bookmark/Domain Reference

### Query Parameters

Note that you can use the API without providing any additional parameters. In that case the API will compute the default analysis for the reference.
If it is a Bookmark, it will use the bookmark default analysis. In case of a Domain, it will do an no-op analysis.

Parameter | Required | Description
--------- | -------- | -----------
groupBy | false | A list of expression to group the result by. This is a multi-valued parameter. Note that the list can be empty, in that case the resulting dataset will be one row as maximum if you provide metrics to compute. Each value must be a valid expression in the Bookmark or Domain scope. See the Expression chapter for reference, but usually you will pass a dimension name you want to group the result by. Alternatively you can use a wildcard (*) as a value to replace the default reference settings. In case of Bookmark it will be replaced by the selected dimensions; in case of Domain it will be replaced by ALL dimensions (so use with care).
metrics | false | A list of expression to compute. This is a multi-valued parameter. Note that the list can be empty, in that case only the group by is applied (However the query will still perform a group by: if you want to export raw data you need to include an unique key in the list, such as a customer identifier). Each value must be a valid expression in the Bookmark or DOmain scope. See the Expression chapter for reference, but usually you will pass a metric name you want to aggregate the data upon. You can also use any valid expression (usually of numeric value) by combining dimensions, metrics and functions.
filters | false | A list of conditional expression to filter the data. Note that the list can be empty. Each value must be a valid expression in the Bookmark or Domain scope. See the Expression chapter for reference, but usually you can use any dimension of type conditional (also called segment), or create a simple condition using constant values for instance. You can also use any valid conditional expression by combining dimensions, metrics and functions.
period | false | the period expression is a shorthand to filter on a timeframe. The expression type is expected to be a date or a timestamp. You can use any valid expression in the Bookmark or Domain scope; usually you can just use a Date dimension (with or without indexing). You can use any function, especially date functions. If this parameter is set, you should provide a timeframe parameters too - see next one for details.
timeframe | false | the `timeframe` parameter is used to define the date or time range of the period. You can either use explicit values (in that case you should provide both lower and upper bounds, thought you can pass a null value to create a semi-open range) or use an alias value or a constant expression. See the [Period definition](#period-timeframe) for details. 
compareTo | false | the `compareTo` parameter is used in conjunction with the `timeframe` parameter to create an automatic comparison between present and past metrics. Setting a value will automatically turn the compare feature on (see [compareTo section for details](comparing-present-and-past-metrics))  You can either use explicit values (in that case you should provide both lower and upper bounds, thought you can pass a null value to create a semi-open range) or use an alias value or a constant expression. Note that the alias values for the `compareTo` parameter are different from the `timeframe` parameter.
orderBy | false | a list of sort expression to order the results. Note that the list can be empty. Each value must be a valid *sort* expression in the Bookmark or Domain scope. A valid *sort* expression is any expression enclosed by the `DESC()` or `ASC()` functions. In order to define the value to sort on, you can use either an index value, the name of a column (Dimension or Metric), or any valid expression.
rollup | false | a list of rollup expression to define how to compute sub-totals. Note that the list can be empty. Each value must be a valid *rollup* expression identifying a query groupBy column. See [Computing Rollup](#computing-rollup) for details. 
limit | true | the `limit` parameter simply allows to limit the number of rows extracted **server side**. This is mandatory for the /query API to avoid memory overflow. If you need a full resultset, you should better use the [/export API](#export-a-bookmark-or-domain) which is designed to handle large number of rows, potentially exceeding the server internal memory. See [Query Pagination](#query-pagination) for details.
offset | false | the `offset` parameter allows to paginate the rows extracted **server side**. Default value is to start at the first row. See [Query Pagination](#query-pagination) for details.
maxResults | false | the `maxResults` parameter allows to control the client-side pagination, and defines the page size. Note that paginating is unexpensive since the complete query (with limit and offset) is stored in the server cache. See [Query Pagination](#query-pagination) for details.
startIndex | false | the `startIndex` parameter allows to control the client-side pagination, and defines the page starting row. Note that paginating is unexpensive since the complete query (with limit and offset) is stored in the server cache. See [Query Pagination](#query-pagination) for details.
lazy | false | when the `lazy` parameter is set to true, the server tries to lookup the results from the cache directly. If it is not possible, no actual computation is performed **at all**. Note that the server may try to perform some transformation if it cannot lookup the exact same query from the cache (that's the smart cache). If the result is not in cache, the /query API returns an error code 404. See [Controlling Query Execution](#controlling-query-execution) for details.
timeout | false | The timeout is in milliseconds. If provided the request execution will be interrupted if exceeding provided timeout. In that case the API will return an ComputingInProgressAPIException. Note that the execution will continue server side, you are only interrupting the request. You can run the same request again (with or without timeout) in order to get the reply (each request is actually pooled, so performing multiple request is actually running the query only once). Note also that the error message contains a QueryID that identifies running query. You can use the /status/QueryID API in order to get information regarding the query (see [/status API](#query-status)) or cancel the execution.
state | false | The `state` parameter, if provided, is used to apply the state configuration to the current domain. It will override any other settings. You can copy an application state - it will compute the corresponding query. Note however that an application can store custom information and do additional computation, that won't be supported by the API. Also the state must be for the same domain for this to work.
data | false | This parameter defines how to layout the data output. You can use it in conjunction with style and envelope to further customize the reply. See the [Supported Data Layouts](#supported-data-layouts) for up to date information.
style | false | This parameter defines the reply style. The style can modify the response type in order to get a HTML version of the reply. It can also modify the content of the JSON reply by altering the way it references objects. If `style=HUMAN` the API will use object names, so you can read and easily understand the output; but on the other side those outputs are fragile to name change so you cannot use them to build an application for instance. If you need the references to be immutable, you need to use `style=ROBOT`; but then the output is going to be difficult to read. Note that you can switch between `HUMAN` and `ROBOT` easily, only the references are modified.
envelope | false | This parameter defines the content of the reply. You can use it to further customize what information you need, from a full reply including the explicit query as it is interpreted by the API, reply meta-information, header and data; or you can just ask for the data. See [Style Parameter](#style-parameter) for details.


### Default Query

If no parameter is provided, the method will try to compute a default query.

In case of a Bookmark reference, the /query API will compute the Bookmark pre-defined analysis, with default settings.

In case of a Domain, the /query API will just add the available metrics and compute totals. If a date or timestamp Dimension is defined, the /query API will also try to use it to restrict the query to the current month.

#### Using State

If a stateID is provided using the `state` parameter, the /query API will try to apply the state. Note that the state must be defined on the same Domain for this to work.

### Expressions

The expressions are all evaluated in the current Domain scope (if the reference is a Bookmark, it is its domain).

See the ["Expression Reference Guide"](#expression-reference-guide) for details about how to create expressions.

### Defining Query

Group By and Metrics

### Filtering



### Period & TimeFrame

The `period` parameter defines the expression to use as the main Period for filtering on `timeframe` and also to perform time-over-time comparison (see [Comparing Present and Past Metrics](#comparing-present-and-past-metrics)).

Also once you defined the `period`, you can then use the `__PERIOD` alias in the expression you are using for other parameters (for example `groupby` or `orderby`; it also work with the [/view API](#view-a-bookmark-or-domain) as `x`, `y`, `color`, ... parameters). Note that you can use the alias `__PERIOD` also in expression, for example: `MONTHLY('__PERIOD')` to extract the first day of month of the date - if you want to aggregate results by month for instance.

Note you can also use the `filters` parameter to filter on period. This may be useful for special cases, e.g. to define a non-constant period filter... Also if you use directly the `filter` parameter, the `__PERIOD` alias is not available.

The `timeframe` parameter is an array of values that you can use to filter the data on the `period` value.
If the `timeframe` is not defined, even if the `period` is defined, no filter is applied.
If the `timeframe` is defined, you must also define the `period` (unless there is a default value available for it)

There are three ways to define the `timeframe`: 
 * using explicit date range
 * using an alias
 * using a expression

#### TimeFrame defined using explicit date range

In that case you can just provide the date or timestamp value as a formatted string. The API supports the following formats:
 * ISO 8601 timestamp: `yyyy-MM-dd'T'HH:mm:ss.SSSZ`
 * ISO 8601 date: `yyyy-MM-dd`
 * default Java timestamp format: `EEE MMM dd HH:mm:ss zzz yyyy`

#### TimeFrame defined using an alias

 * `__LAST_7_DAYS` 
 * `__CURRENT_MONTH` 
 * `__PREVIOUS_MONTH`
 * `__CURRENT_YEAR`

#### TimeFrame defined using an expression

The last method is to define a constant expression. 

### Comparing Present and Past metrics

The `compareTo` parameter accept the same definitions as the `timeframe` parameter, except for the aliases. The `compareTo` parameter defines a different set of aliases that you can use to easily define relative comparison:

 * `__COMPARE_TO_PREVIOUS_PERIOD` 
 * `__COMPARE_TO_PREVIOUS_MONTH` 
 * `__COMPARE_TO_PREVIOUS_YEAR`
 
#### ordering by comparison metric
 
 

### Ordering the results

#### Sort Expression

#### defining how to sort

The easiest way to define a sort order is to use an existing resultset column and provide the position of the column. For example in order to sort on the first column:
`ASC(0)`
The position is zero-based, so the first column is 0, the second is 1, etc...

<aside class="warning">
You cannot use index based orderBy to sort by a comareTo metric. In that case (if the `compareTo` parameter is set), indexes are not taking into account the compareTo columns generated by the query. If you need to sort the result based on a compareTo column you must use the explicit column name (see <a href="#ordering-by-comparison-metric">ordering by comparison metric</a> for details)
</aside>

### Computing Rollup


### Controlling Query Execution

There is different ways to control the Query execution:
 * using Lazy execution to get result only from the cache
 * using a timeout to control how long to wait for the results
 * when the Query timeout it is then possible to monitor progress and cancel it, see [Query Status](#query-status) for details.

#### Lazy execution

when the `lazy` parameter is set to true, the server tries to lookup the results from the cache directly. If it is not possible, no actual computation is performed **at all**. Note that the server may try to perform some transformation if it cannot lookup the exact same query from the cache (that's the smart cache).

If the query is not in cache, the reply will state an error of cause NOT_IN_CACHE and a code 404.

Note that if you use the `lazy=noError`, the /query API will return a valid reply instead of an error code, with an empty resultset and a problem identifying the cause (not in cache). That may be useful in some situation!

#### Timeout

### Query Pagination

### Reply

#### Table Header

#### Supported Data Layouts

The `data` parameter allows to select the output layout.

Data&nbsp;Value | Table Layout
---------- | ------------
`TABLE` | return a matrix, that is an array of row array. A row is an array cell values.
`RECORDS` | return the table as an array of records. Each row is a record of the form `{"columnName":value,...}`
`TRANSPOSE` | if the query has multiple metrics, transpose the table to create a row for each record/metric. It adds two additional columns `metric` and `value`
`SQL` | additionally you can ask to retrieve the SQL code for the Query instead of the results
`LEGACY` | return a JSON compatible with legacy API - this is for internal use only, you should not use it, it will be deprecated soon.

Note that you can play with the `envelope` parameter in order to customize further the reply. For instance if you want to get only the data as the reply, you can add `envelope=DATA` parameter. If `data=SQL` is used, this will return just the SQL statement. See [Envelope Parameter](#envelope-parameter) for details.

### Examples

#### How to compute metrics by month and display it by pivot?

How can one create a query that returns the following table:

Country | Total | Jan 2016 | Fev 2016
------- | ----- | -------- | --------
France | 1.000 | 400 | 600
US | 2.000 | 800 | 1.200

<aside class="notice">
Note that we are using POST version of /analysis/ID/query for clarity. You can as well use the GET version.
</aside>

> this is the POST payload you have to send to run the query.

```json
{
"period":"date",
"timeframe":["2016-01-01","2016-02-29"],
"groupBy":["'Country'"],
"metrics":["'Total'"]
}
```

Let's start with the simple query that returns the total for the selected period (from 2016/01/01 to 2016/02/29):

Country | Total 
------- | ----- 
France | 1.000 
US | 2.000 

```json
{
"period":"Date",
"timeframe":["2016-01-01","2016-02-29"],
"groupBy":["'Country'","monthly(__PERIOD)"],
"metrics":["'Total'"]
}
```

We can easily add the monthly total to get the details:

Country | Monthly | Total 
------- | ------- | -----
France | 2016/01/01 | 1.000 
US | 2016/02/29 | 2.000 

<aside class="notice">
Note that we are using the `__PERIOD` alias in place of the `Date` dimension, and group by month using the `monthly()` function.
</aside>

```json
{
"period":"Date",
"timeframe":["2016-01-01","2016-02-29"],
"groupBy":["'Country'","monthly(__PERIOD)"],
"metrics":["'Total'"],
"orderBy":["'Country'","monthly(__PERIOD)"],
"rollup":[0]
}
```

We can also add the total by country using the rollup feature:

GROUPING_ID | Country | Monthly | Total 
----------- | ------- | ------- | -----
1 | France |  | 1.000 
  | France | 2016-01-01 | 400
  | France | 2016-02-01 | 600
1 | US | | 2.000 
  | US | 2016-01-01 | 800
  | US | 2016-02-01 | 1.200
  

```json
{
"period":"Date",
"timeframe":["2016-01-01","2016-02-29"],
"groupBy":["'Country'","monthly(__PERIOD)"],
"metrics":["'Total'"
          ,"'Total' ON monthly(__PERIOD)=date(\"01/01/16\") as 'JAN 2016'"
          ,"'Total' ON monthly(__PERIOD)=date(\"01/02/16\") as 'FEV 2016'"]
}
```

Now in order to pivot, we need to add additional metrics for each required column:

Country | Total | Jan 2016 | Fev 2016
------- | ----- | -------- | --------
France | 1.000 | 400 | 600
US | 2.000 | 800 | 1.200


## Query Status

This method provides the status of a running query.



## Export a Bookmark or Domain

This method is similar to the /query API but will stream the result as a data file instead of returning a JSON reply.

It support several format for export: CSV, XLS, XLSX

### HTTP Request

`GET http://yourserverdomain/v4.2/analytics/{REFERENCE}/export/{filename}`

<aside class="success">
Remember to provide a valid OAuth TOKEN!
</aside>

### URL Parameters

Parameter | Required | Description
--------- | -------- | -----------
REFERENCE | true | The identifier of the Bookmark/Domain to retrieve
filename | true | The filename can be defined here to provide a filename when the user will be prompted to save the file. The name part is optional, but the file extension must be provided to define the format of the export

Filename&nbsp;extension | Format
----------------------- | ------
.csv | Comma separated file with an header for the columns
.xls | Excel file format
.xlsx | Excel file format


## View a Bookmark or Domain


## API output parameters

All API methods returning data accept some parameters to modify the output:

Parameter | Required | Description
--------- | -------- | -----------
style | false | defines the output style. Values can be HUMAN, ROBOT, HTML or LEGACY. See next chapter for details
envelope | false | defines the output content. Values can be ALL, RESULT, DATA

### Style parameter

This parameter defines the reply style.

Style&nbsp;Value | Table Layout
---------- | ------------
`HUMAN` | When the `style` parameter is set to `HUMAN`, the API will use name as object reference. This allows the reply to be easily readable by human. The drawback is that the information is fragile to name change, so you shouldn't use it statically in your apps.
`ROBOT` | When the `style` parameter is set to `ROBOT`, the API will use immutable identifier as object reference. That makes the information robust to name change, but you may have hard time deciphering it (unless you are NEO).
`HTML` | This special style modify the output to be a static HTML page, instead of the regular JSON data feed. This is mainly used for exploring the API in a friendly way (think of it as a custom swagger UI). The HTML pages are actually good old forms that you can use also to input parameters and quickly see the result. It also provides some additional shortcut links to related queries you may need.

### Envelope parameter

This parameter defines the content of the reply. You can use it to further customize what information you need, from a full reply including the explicit query as it is interpreted by the API, reply meta-information, header and data; or you can just ask for the data.

Envelope&nbsp;Value | Table Layout
---------- | ------------
`ALL`  | This the default. It will return the full reply, including the original query as evaluated by the API (so it can be different than the query you send, because it will list all implicit settings and override), and the complete result including meta informations, table header and the data.
`RESULT` | This setting will only return the result part
`DATA` | This setting will only return the data with no additional information.


# Model API

The Model API allows you to configure the meta-model, including project creation, customization of domains, creation of dimensions and metrics, configuring hierarchies and indexing, etc...

# Management API

This is a set of APIs to manage the OB server.

# Expression Reference Guide

This part describes how to write Expression and the available Functions to use.

## Constant Expression

Constant type | Example
------------- | -------
Numerical   | 123.4
Text   | "Hello world"
Date   | DATE("DD/MM/YY")<br><i>note that you cannot change the date format when defining a constant</i>

<aside class="success">
String constants are enclose using <b>double quotes</b>, be careful!
</aside>

## Referencing objects

Expression allows you to create references to a model object and to perform operations on their underlying values.

To create a reference, you can use two syntax:

 * Shortand
 * Explicit

>examples of referencing objects using identifiers

```Shell

// always enclose the identifier in single-quotes

// using a named identifier 
// without specifying the object type, 
// so the scope will manage the order
// in case of conflicts
'this is a identifier name'

// using an ID
@'someID'

// using a typed name, 
// in case of collision
[dimension:'this is a dimension']

// using a typed ID 
// only the ID is enclosed, not the type
@bookmark:'someBookmarkID'
```

Important: The resolution of an object identifier is case sensitive, and the identifier must be enclosed in simple quotes.

<aside class="success">
Identifiers must be enclosed between <b>single quotes</b>, be careful!
</aside>

Object type | Shorthand | Explicit
----------- | --------- | --------
Column | #'CUSTOMER_NAME' | [col:'CUSTOMER_NAME'] or @col:'CUSTOMER_NAME'
Object by name<br>(Domain, Dimension, Metric or Relation) | 'Name' | [dimension:'Name'] or @metric:'Name'<br><i>note: using the explicit form can be useful to resolve name collision
Object by ID<br>(Domain, Dimension, Metric or Relation) | @'someID' | [id:'someID']
Parameter | $'M0' | [param:'M0']

## Composing Expression

If an expression is an Object, it is usually possible to compose it with expression valid in the target object scope.

For example if the scope is the `Sales` domain, you can create the following expression to resolve the country of the customer: 

    `'customer'.'address'.'country'.'name'`

You can also use the special expression `$self` to reference the domain itself, for example: 

    `'customer'.'address'`
## Naming expressions

Since version 4.2.2, Bouquet server allows us to give a name to expressions. Just use the AS keyword in the expression editor:

'metric1' + 'metric2' AS 'sum_of_both_metrics'

This can be useful when using custom expressions to compute an analysis. The resulting column in the data-table will have the expression's name. Note however that if you define a dimension or a metric, the dimension or metric given name will take precedence over the expression's name.

## Operators

Operator | Type | Description
-------- | ---- | -----------
expr1 &#124;&#124; expr2 | Logical | OR logical operator
expr1 && expr2 | Logical | AND logical operator
expr1 LIKE expr2  |  Logical | Search for a specified pattern in a column
expr1 = expr2 | Comparison | Checks if expr1 is equal to expr2
expr1 > expr2 | Comparison | Greater comparator
expr1 >= expr2 | Comparison | Greater or equal comparator
expr1 < expr2 | Comparison | Less comparator
expr1 <= expr2 | Comparison | Less or equal comparator
expr1 != expr2 | Comparison | Not equal comparator
expr1 RLIKE expr2 | Comparison | Executes a "LIKE" comparison using a regexp expression
n1 % n2 | Math  |  Modulo operator
n1 + n2 | Math  |  Addition operator
n1 - n2 | Math  |  Subtraction operator
n1 *n2 | Math  |  Multiplication operator
n1 / n2 | Math  |  Division operator
n1 ** n2 | Math  |  Exponential operator (n1 to n2th power)

## Math functions

Function | Definition
-------- | ----------
ABS(n) | Absolute value of number
ACOS(n) | Arc cosine of n
ASIN(n) | Arc sine of n
ATAN(n) | Arc tangent of n
COS(n) | Cosine of n
CEIL(n) | Smallest integer value that is greater than or equal to a number.
COSH(n) | Hyperbolic cosine of n
DEGREES(n) | Argument converted to degrees of n
EXP(n1,n2) | Exponential (n1 to n2th power)
FLOOR(n) | Largest integer value that is less than or equal to a number.
GREATEST(expr1, expr2, ...) |Greatest value in a list of expressions
LEAST(expr1, expr2, ...) | Smallest value in a list of expressions
LN(n) | Natural Log of n, where n>0
LOG(n1,n2) | Log of n1, base n2
MINUS(n1,n2) | Difference between n1 and n2
PI() | PI number of n
POWER(n1,n2) | n1 raised to the n2th power.
RADIANS(n) | Argument converted to radians of n
RAND() |Generate a random value
ROUND(n,[p]) |Number n rounded to a given (optionnal) precision p. p can be negative, to cause p digits left of the decimal point of the value n to become zero.
SIGN(n) |Sign of a number (-1 or 1)
SIN(n) | Sine of n
SINH(n) | Hyperbolic Sine of n
SQRT(n) | Square root of n
SUBSTRACTION(n1,n2) | Substract n2 from n1
TAN(n) | Tangent of n
TANH(n) | Hyperbolic tangent of n
TRUNCATE(n1,n2) | n1 truncated to n2 decimal places.

##Date functions

Function | Definition
-------- | ----------
ADD_MONTHS(date,num_months) | Returns date + num_months (can be negative)
CURRENT_DATE() | Get the current system date
CURRENT_TIMESTAMP() | Get the current system timestamp
DAILY(timestamp) | convert the timestamp into a date
DATE_ADD(date1,number,period) | Add to date1 the number of period (unit in SECOND,MINUTE,HOUR,DAY,MONTH,YEAR)<br>Note: Adding two dates is not possible
DATE_INTERVAL(date1,date2,period) | Compute the difference btw date1 and date2 in a particular period (unit in SECOND,MINUTE,HOUR,DAY)
DATE_SUB(date1,date2) | Remove date2 from date1
DATE_SUB(date1,number,period) | Remove from date1 the number of period (unit in SECOND,MINUTE,HOUR,DAY,MONTH,YEAR)
DATE_TRUNCATE(date,format) | Truncate the date (it can be a Date or a Timestamp) depending on the format constant string:<br>if format="day" then the result is the Date part of date (excluding the time)<br>if format="week" then the result is the first day of the week containing date<br>if format="month" then the result is the first day of the month containing date<br>if format="quarter" then the result is the first day of the quarter containing date<br>if format="year" then the result is the first day of the year containing date<br><br>Note: this is useful to perform weekly, monthly, quarterly or yearly analysis (you can also use the shortcut functions DAILY(), WEEKLY(), MONTHLY(), QUARTERLY() or YEARLY())
DAY(date) | Extract the day of the month from date
DAY_OF_WEEK(date) | extract the day of the week date, between 1 and 7. Day 1 is sunday
DAY_OF_YEAR(date) | Extract the day of the year from the date
FROM_EPOCH(n) | Return a timestamp based on a numeric field representing the number of seconds since 01-01-1970.
HOUR(date) | Extract the hour from date
MINUTE(date) | Extract minutes from date
MONTHLY(date or timestamp) | return the first day of month from the date
MONTHS(date) | Extract the month in the year from date
MONTHS_BETWEEN(date1,date2) | Number of months between d1 and d2
QUARTERLY(date or timestamp) | return the first day of the quarter from the date
SECOND(date) | Extract seconds from date
TO_DATE(timestamp) | Convert a timestamp to a date
TO_DATE(text, format) | Convert a text to a date with a specific format
TO_EPOCH(timestamp) | Return a numeric value based on a timestamp field representing the number of seconds since 01-01-1970.
TO_TIMESTAMP(date) | Convert a timestamp to a date
TO_TIMESTAMP(text, format) | Convert a text to a timestamp with a specific format
WEEKLY(date or timestamp) | return the first day of the week from the date
YEAR(date) | Extract the year from date
YEARLY(date or timestamp) | return the first day of the year from the date

## Text functions

Function | Definition
-------- | ----------
CONCAT(s1,s2) | Concatenate Text1 and text2
LENGTH(s) | Length of a text
LOWER(s) | Lower case text
MD5(string) | Calculates the MD5 hash of string, returning the result in hexadecimal
POSITION(s1, s2) | Returns the position of s2 within s1 starting from index 1, 0 otherwise
REPLACE(text, from, to) | Replace within the first text from text with to text
REVERSE(string) | Returns reversed string
SPLIT_PART(string, delimiter, position) | Splits string on delimiter and return the given field (counting from one)
SUBSTRING(text,n1 [,n2]) | Substring of s, starting from index n1 optionally ending at index n2
TRANSLATE(text, from, to) | Replace within the first text each single char specified in from text withto the corresponding char specified in to
TO_CHAR(number) | Convert a number to a text
TO_CHAR(date or timestamp, format) | Convert a date or timestamp to a text with a specific format. You can use any format supported by the underlying data-source. Check next § for a quick refernce guide
UPPER(s) | Uppercase text

### Date Formatting

This is just a quick reference guide for usual formats. You can use any format supported by the underlying data-source. 

Format | Definition
------ | ----------
YYYY | 4 digits year, e.g. 2016
YYY  | 3 digits year, e.g. 016
YY  | 2 digits year, e.g. 16
Y  | 1 digit year, e.g. 6
MONTH | Month name (uppercase)
MON | abbreviated month name
MM | month number (01-12)
DAY | name of the day
DDD | Day of the year (1-366)
DD | Day of the month (1-31)
D | Day of week (1-7)

## Regex functions

Function | Available | Definition
-------- | --------- | ----------
REGEXP_COUNT(string, regexp) | RedShift only | Counts the number of occurrence of the regexp within the string
REGEXP_INSTR(string, regexp) | RedShift & Oracle | Returns the position of the first occurrence found of the searched regexp within the string
REGEXP_REPLACE(string, regexp, replace) | All but MySQL | Replace substring matching a POSIX regular expression
REGEXP_SUBSTR(string, regexp) | All but MySQL | Extract substring matching POSIX regular expression

## JSON functions

These functions are only supported by the Redshift plugin.

Function | Definition
-------- | ----------
JSON_ARRAY_LENGTH ('json string') | Returns the number of elements in the outer array of a JSON string
JSON_EXTRACT_ARRAY_ELEMENT_TEXT('json string', pos) | Returns a JSON array element in the outermost array of a JSON string, using a zero-based index.
JSON_EXTRACT_PATH_TEXT(json string', 'path_elem' [,'path_elem'[, …]]) | Returns the value for the key:value pair referenced by a series of path elements in a JSON string. The JSON path can be nested up to five levels deep.

## Aggregate functions

Function | Definition
-------- | ----------
COUNT(*) or COUNT([DISTINCT] expr) | Count the no of rows returned or no of rows returned by expr
AVG([DISTINCT] value expr) | Average value of ‘n' ignoring NULL
DISTINCT(expr) | Get distinct values returned by expr
MAX(value expr) | Maximum value within an expression
MEDIAN([DISTINCT] n) | Median of n, ignoring NULLs.
MIN(value expr) | Minimum value within an expression
SUM(value expr) | Sum an expression

VARIANCE([DISTINCT] n) | Variance of n, ignoring NULLs

### Windowing / Analytics functions

You can apply an operator on a group of rows using the windowing syntax:

 * Use the partition expression to partition the query result set into groups based on one or more value expression. If you omit this clause, then the function treats all rows of the query result set as a single group
 * Use the order by expression to specify how data is ordered within a partition

Function | Definition
-------- | ----------
AVG([DISTINCT] value expr, [partition expr], [order by expr]) | Average value of ‘n' ignoring NULL
DISTINCT(expr) | Get distinct values returned by expr
MAX(value expr, [partition expr], [order by expr]) | Maximum value within an expression
MEDIAN([DISTINCT] n) | Median of n, ignoring NULLs.
MIN(value expr, [partition expr], [order by expr]) | Minimum value within an expression
RANK([partition expr], [order by expr]) | Returns the rank of a value in apartition expression. Same values within a partition have the same rank and the partition can be ordered
ROW_NUMBER([partition expr], [order by expr]) | Returns a number to each row to which it is applied in a partition expression. The number returned is unique within its partition and the partition can be ordered
SUM(value expr, [partition expr], [order by expr]) | Sum an expression

## Numeric functions

Function | Definition
-------- | ----------
TO_INTEGER(expr) | Convert a text, or a numeric field to an integer
TO_NUMBER(expr) | Convert a text, an integer or a numeric field to a float
TO_NUMBER(expr,size,precision) | Convert a text, an integer or a numeric field to a specific numeric format

## Logical functions

Function | Definition
-------- | ----------
CASE(condition1,then1,...,[else]) | Group the data into sub-sets
ISNULL(expr) | Expr is null logical function
NOT(expr1, expr2) | Expr1 not equal to expr2 logical operator
NULLIF(expr1, expr2) | Returns the first expression if the two expressions aren't equal or a null value if the two expressions are equal

## Miscellaneous functions

Function | Definition
-------- | ----------
EXISTS(expr) | Checks that a sub condition is matched. The EXISTS condition is considered "to be met" if the subquery returns at least one row.
LPAD(text,length,text) | Left pad with the text the text to specified length
LTRIM(text,[character]) | Remove leading characters from the text. By default, the character to remove is a whitespace
RPAD(text,length,text) | Right pad with the text the text to specified length
RTRIM(text,[character]) | Remove trailing characters from the text. By default, the character to remove is a whitespace
TRIM(text, [character]) | Remove leading and trailing characters from the text. By default, the character to remove is a whitespace
