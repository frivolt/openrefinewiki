This page proposes a new optional API for reconciliation services, allowing clients to pull properties of reconciled records.

# Background

The Freebase extension enabled users to pull data from Freebase, as shown in this video: https://youtu.be/5tsyz3ibYzk
This feature queried Freebase with MQL (Metaweb Query Language), a JSON-based query format. A dialog enabled the user to pick a few properties to fetch from Freebase, and the OpenRefine extension generated an MQL query based on this selection, fetching the data by batch and filling new columns based on the results.

In 2015, [a pull request](https://github.com/OpenRefine/OpenRefine/pull/948) proposed to generalize this mechanism to read from arbitrary MQL endpoints (still as an OpenRefine extension). Unfortunately, Freebase was being shut down and so the extension was removed from the code, and the PR could therefore not be merged.

In 2017, the default reconciliation service was changed to Wikidata. We want to migrate the data extension for Wikidata and make it easy for other reconciliation services to also support data extension. There are a few options to do that:
1. Migrate the existing PR into OpenRefine's core, therefore adding native support for MQL in OpenRefine. Users could register MQL endpoints, which would be independent from reconciliation services, and could use them to retrieve data from compatible sources. We would need to create an MQL endpoint for Wikidata and add it as a default service in OpenRefine.
2. Add new methods to the existing reconciliation API.

MQL is not used much outside Freebase, the specifications of the protocol are not available online anymore, so it is quite unlikely that people would implement it.

