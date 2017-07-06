This page proposes a new optional API for reconciliation services, allowing clients to pull properties of reconciled records. It is a **draft**, to be discussed with the community.

# Background

The Freebase extension enabled users to pull data from Freebase, as shown in [this video](https://youtu.be/5tsyz3ibYzk).
This feature queried Freebase with MQL (Metaweb Query Language), a JSON-based query format. A dialog enabled the user to pick a few properties to fetch from Freebase, and the OpenRefine extension generated an MQL query based on this selection, fetching the
data by batch and filling new columns based on the results.

In 2015, [a pull request](https://github.com/OpenRefine/OpenRefine/pull/948) proposed to generalize this mechanism to read from arbitrary MQL endpoints (still as an OpenRefine extension). Unfortunately, Freebase was being shut down and so the extension was
removed from the code, and the PR could therefore not be merged.

In 2017, the default reconciliation service was changed to Wikidata. We want to migrate the data extension for Wikidata and make it easy for other reconciliation services to also support data extension. There are a few options to do that:
1. Migrate the existing PR into OpenRefine's core, therefore adding native support for MQL in OpenRefine. Users could register MQL endpoints, which would be independent from reconciliation services, and could use them to retrieve data from compatible
sources. We would need to create an MQL endpoint for Wikidata and add it as a default service in OpenRefine.
   * Pros:
     * the code is already available
     * MQL is rich: it supports retrieving nested properties natively and can integrate constraints on the properties to fetch (e.g. retrieve their count instead of their values)
   * Cons:
     * MQL basically died when Freebase was shut down, even the reference guide of the language has to be retrieved [via the Internet Archive](https://web.archive.org/web/20160305144638/http://mql.freebaseapps.com/index.html)
     * it will be hard for data providers to implement a MQL service (this is a rich query language with few existing open source implementations as far as I can tell), and [Wikidata is probably not going to implement that soon](https://phabricator.wikimedia.org/T85181)
     * MQL is not enough, we also need other APIs to suggest properties given a particular type (Freebase had a custom API for that) and for property auto-completion
2. Add new methods to the existing reconciliation API. Reconciliation services will update their service metadata to expose this new feature.
   * Pros:
     * transparent for the user: no need to manage two different lists of providers
     * easy to implement for data providers: they just need to add a few methods to their existing services
     * the OpenRefine community has full control over the specifications of the protocol
     * the helper API (suggestion of properties) is consistently integrated with the other services (no need to maintain separate endpoints).
   * Cons:
     * the query language will not be as rich as MQL, which could limit the range of applications
     * reconciliation endpoints and data extension endpoints are tied to each other, whereas in theory we might want to keep them separate, so that users could use different endpoints sharing a common identifier space.

This document proposes to follow the second approach and fleshes out a protocol specification.

# Overview of the workflow

1. Reconcile a column with a standard reconciliation service

2. Click "Add column from reconciled values"

3. The user is proposed some properties to fetch, based on the type they reconciled their column against (if any). They can also pick their own property with the suggest widget (same as for the reconciliation dialog).

4. A preview of the columns to be fetched is displayed on the right-hand side of the dialog, based on a sample of the rows.

5. Once the user has clicked "OK", columns are fetched and added to the project. Columns corresponding to other items from the service are directly reconciled, and the column is marked as reconciled against the type suggested by the service for that
property. The user can run data extension again from that column.

# Specification

Services supporting data extension must add an `extend` field in their service metadata. This field is expected to have the following subfields, all optional:
* `propose_properties` stores the endpoint of an API which will be used to suggest properties to fetch (see specification below). The field contains an object with a `service_url` and `service_path` which will be concatenated to obtain the URL where the endpoint is available, just like the other services in the metadata. If this field is not provided, no property will be suggested in the dialog (the user will have to input them manually).
* `property_settings` stores the specification of a form where the user will be able to configure how a given property should be fetched (see specification below). If this field is not provided, the user will not be proposed with settings.

The service endpoint must also accept a new parameter `extend` (in addition to `queries` which is used for reconciliation). Its behaviour is described in the following section.

Example service metadata:

    "extend": {
      "fetch_column": {
        "service_url": "https://tools.wmflabs.org/openrefine-wikidata",
        "service_path": "/en/fetch_properties_by_batch"
      },
      "property_settings": []
    }


## Data extension protocol

The consumer (OpenRefine) calls the service endpoint with a JSON object in the `extend` parameter, containing the following fields:
* `ids` is a list of strings, each of which being an identifier of a record as returned by the reconciliation method. These are the records whose properties should be retrieved.
* `properties` is a list of JSON objects. They specify the properties to be fetched for each item, and contain the following fields:
  * `id` (a string): the identifier of the property as returned by the property suggest service (and optionally the property proposal service)
  * `settings`: a JSON object storing parameters about how the property should be fetched (see below).

Example:

    {
      "ids": [
        "Q7205598",
        "Q218765",
        "Q845632",
        "Q5661356"
      ],
      "properties": [
        {
          "id": "P856"
        },
        {
          "id": "P159"
        }
      ]
    }

The service returns a JSON response formatted as follows:

* `meta` contains a list of column metadata. The order of the properties must be the same
   as the one provided in the query. Each element is an object containing the following keys:
   * `id` (mandatory): the identifier of the property
   * `name` (mandatory): the human-readable name of the property
   * `type` (optional): an object with `id` and `name` keys representing the expected
      type of values for that property. The notion of type is the same as the one
      used for reconciliation. The `type` field should only be provided when the property returns
      reconciled items.
* `rows` contains an object. Its keys must be exactly the record ids (`ids`) passed in the query.
   The value for each record id is an object representing a row for that id. The keys of a    row object must be exactly the property ids passed in the query (`"P856"` and `"P159"` in the example above). The value for a property id should be a list of cell objects.

Cell objects are JSON objects which contain the representation of an OpenRefine cell.
* an object with a single `"str"` key and a string value for it represents
  a cell with a (bare) string in it.
  Example: `{"str": "193.54.0.0/15"}`

* an object with `"id"` and `"name"` represents a reconciled value
  (from the same reconciliation service). It will be stored as 
  a matched cell (with maximum reconciliation score).
  Example: `{"name": "Warsaw","id": "Q270"}`

* an empty object `{}` represents an empty cell

* an object with `"date"` and an ISO-formatted date string represents a point in time.
  Example: `{"date": "1987-02-01T00:00:00+00:00"}`

* an object with `"float"` and a numerical value represents a quantity.
  Example: `{"float": 48.2736}`

* an object with `"int"` and an integer represents a number.
  Example: `{"int": 54}`

* an object with `"bool"` and `true` or `false` represents a boolean.
  Example: `{"bool": false}`

Example of a full response (for the example query above):

    {
      "rows": {
        "Q5661356": {
          "P159": [],
          "P856": []
        },
        "Q7205598": {
          "P159": [
            {
              "name": "Warsaw",
              "id": "Q270"
            }
          ],
          "P856": [
            {
              "str": "http://www.polkomtel.com.pl/english"
            },
            {
              "str": "http://www.polkomtel.com.pl/"
            }
          ]
        },
        "Q845632": {
          "P159": [
            {
              "name": "Bærum",
              "id": "Q57076"
            }
          ],
          "P856": [
            {
              "str": "http://www.telenor.com/"
            }
          ]
        },
        "Q218765": {
          "P159": [
            {
              "name": "Paris",
              "id": "Q90"
            }
          ],
          "P856": [
            {
              "str": "http://www.sfr.fr/"
            }
          ]
        }
      },
      "ids": [
        "Q7205598",
        "Q218765",
        "Q845632",
        "Q5661356"
      ],
      "meta": [
        {
          "id": "P159",
          "name": "headquarters location",
          "type": {
             "id": "Q7540126",
             "name": "headquarters",
          }
        },
        {
          "id": "P856",
          "name": "official website",
        }
      ]
    }


## Property proposal protocol

The role of the property proposal endpoint is to suggest a list of properties to fetch. As only input, it accepts the type a column was reconciled against, and proposes relevant properties for that type. If no type is provided, it should suggest properties for a column reconciled against no type.

The type is specified by its id in the `type` GET parameter of the endpoint, as follows:

https://tools.wmflabs.org/openrefine-wikidata/en/propose_properties?type=Q3354859

The endpoint returns a JSON response as follows:

    {
      "properties": [
        {
          "id": "P969",
          "name": "located at street address"
        },
        {
          "id": "P1449",
          "name": "nickname"
        },
        {
          "id": "P17",
          "name": "country"
        },
      ],
      "type": "Q3354859"
    }

This endpoint must support JSONP via the `callback` parameter (just like all other endpoints of the reconciliation service).

## Settings specification

We need to figure out how the service can specify which settings it accepts for the properties. Here are a few examples of settings we want to support for Wikidata:
* return counts instead of the lists of values
* fetch only one claim (the preferred one)
* fetch claims that hold at a given date
* only fetch referenced claims

So, the reconciliation service should be able to specify a form which would be exposed to the user, and would obtain the values of that form for each property.

For Freebase, the user had to input the properties directly as a JSON payload which was included in the MQL query:
![MQL constraint for Freebase](http://pintoch.ulminfo.fr/0113381cff/mql_constraints.png)

It is very easy to keep this setup and let the user pass some arbitrary JSON to the data extension service which will interpret it in some way, but it would be much nicer to have a proper form that would depend on the service.

TODO
