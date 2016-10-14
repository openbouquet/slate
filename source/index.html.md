---
title: OPen Bouquet API Reference

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

## Project

A Project is the top level object that organize data-source model. First a Project defines how to accessing the data-source, through a JDBC url and required authentication. Also the Project defines which part of the data-source will be made available through the API, by listing visible schemas.

A Project also maintains a list of Domain.

## Domain

A Domain is associated with a table or view in the data-source. A Domain can be created dynamically by Open Bouquet, or explicitly by a user. Also a user can modify a dynamic Domain to make it permanent and persist the changes to the model.

A Domain has Dimension and Metric attributes, which are dynamically mapped to the underlying data-source. A user can also modify an existing attribute, or create new ones. Those changes are persisted in the model.

## Bookmark

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

```shell
curl -X GET 
	--header 'Accept: application/json' 
	--header 'Authorization: Bearer TOKEN'
	"http://yourserverdomain/v4.2/analytics"
```

> The above command returns JSON structured like this:

```json
{
"parent": {
"name": "Root",
"description": "list all your available content, organize by Projects and Bookmarks",
"selfRef": "/",
"type": "FOLDER"
},
"children": [
{
"name": "Projects",
"description": "list all your Projects",
"parentRef": "/",
"selfRef": "/PROJECTS",
"type": "FOLDER"
},
{
"name": "Shared Bookmarks",
"description": "list all the bookmarks shared with you",
"parentRef": "/",
"selfRef": "/SHARED",
"type": "FOLDER"
},
{
"name": "My Bookmarks",
"description": "list all your bookmarks",
"parentRef": "/",
"selfRef": "/MYBOOKMARKS",
"type": "FOLDER"
}
]
}
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

The reply has the following structure:


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
groupBy | false | This is a multi-valued parameter. Each value must be a valid expression in the Bookmark/Domain scope. See the Expression chapter for reference, but usually you will pass the dimension name you want to group the result by. Alternatively you can use a wildcard (*) as a value to replace the default reference settings. In case of Bookmark it will be replaced by the selected dimensions; in case of Domain it will be replaced by ALL dimensions (so use with care).
metrics | false |
filters | false |
period | false |
timeframe | false |
compareTo | false |
orderBy | false | 
limit | false | 
offset | false | 
maxRsults | false | 
startIndex | false | 
lazy | false | 
data | false | 
style | false | 
envelope | false | 
timeout | false | 

#### Default Query

If no parameter is provided, the method will try to compute a default query.

In case of a Bookmark reference, it will compute the Bookmark pre-defined analysis, with default settings.

#### Expressions

### Reply

#### Table Header

#### Table layout

The `data` parameter allows to select the output layout.

Data&nbsp;Value | Table Layout
---------- | ------------
`table` | return a matrix, that is an array of row array. A row is an array cell values.
`records` | return the table as an array of records. Each row is a record of the form `{"columnName":value,...}`
`transpose` | if the query has multiple metrics, transpose the table to create a row for each record/metric. It adds two additional columns `metric` and `value`
`legacy` | return a JSON compatible with legacy API - do not use!
`sql` | additionally you can ask to retrieve the SQL code for the Query instead of the results


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

### Envelope parameter

# Model API

The Model API allows you to configure the meta-model, including project creation, customization of domains, creation of dimensions and metrics, configuring hierarchies and indexing, etc...

# Management API

This is a set of APIs to manage the OB server.

