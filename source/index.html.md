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

# Analytics

The Analytics API provides you all you need to interact with your data:

* list available content like Projects, Bookmarks and Domains
* query a specific Bookmark or Domain
* view dataviz for a specific Bookmark or Domain (as VegaLite output)
* explore a Bookmark or Domain scope
* create a Bookmark out of a query

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
This API method support the style=HTML.
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

<aside class="success">
Remember — a happy kitten is an authenticated kitten!
</aside>

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


## Query a Specific Bookmark or Domain

This method performs a query on the specifed Bookmark or Domain.

### HTTP Request

`GET http://yourserverdomain/v4.2/<ID>`

### URL Parameters

Parameter | Required | Description
--------- | -------- | -----------
REFERENCE | true | The identifier of the Bookmark/Domain to retrieve

### Query Parameters


### Bookmark/Domain Reference

## API output parameters

All API methods returning data accept some parameters to modify the output:

Parameter | Required | Description
--------- | -------- | -----------
style | false | defines the output style. Values can be HUMAN, ROBOT, HTML or LEGACY. See next chapter for details
envelope | false | defines the output content. Values can be ALL, RESULT, DATA

### Style parameter

### Envelope parameter



