This page proposes a new optional API for reconciliation services, allowing clients to pull properties of reconciled records. It is a **draft**, to be discussed with the community.

# Background

The Freebase extension enabled users to pull data from Freebase, as shown in this video: https://youtu.be/5tsyz3ibYzk
This feature queried Freebase with MQL (Metaweb Query Language), a JSON-based query format. A dialog enabled the user to pick a few properties to fetch from Freebase, and the OpenRefine extension generated an MQL query based on this selection, fetching the data by batch and filling new columns based on the results.

In 2015, [a pull request](https://github.com/OpenRefine/OpenRefine/pull/948) proposed to generalize this mechanism to read from arbitrary MQL endpoints (still as an OpenRefine extension). Unfortunately, Freebase was being shut down and so the extension was removed from the code, and the PR could therefore not be merged.

In 2017, the default reconciliation service was changed to Wikidata. We want to migrate the data extension for Wikidata and make it easy for other reconciliation services to also support data extension. There are a few options to do that:
1. Migrate the existing PR into OpenRefine's core, therefore adding native support for MQL in OpenRefine. Users could register MQL endpoints, which would be independent from reconciliation services, and could use them to retrieve data from compatible sources. We would need to create an MQL endpoint for Wikidata and add it as a default service in OpenRefine.
   * Pros:
     * the code is already available
     * MQL is rich: it supports retrieving nested properties natively and can integrate constraints on the properties to fetch (e.g. retrieve their count instead of their values)
   * Cons:
     * MQL basically died when Freebase was shut down, the reference guide of the language has to be retrieved [via the Internet Archive](https://web.archive.org/web/20160305144638/http://mql.freebaseapps.com/index.html)
     * it will be hard for data providers to implement a MQL service (this is a rich query language with few existing open source implementations)
     * MQL is not enough, we also need other APIs to suggest properties given a particular type (Freebase had a custom API for that)
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

# Specification