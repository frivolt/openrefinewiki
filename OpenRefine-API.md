# Refine API

This is a generic API reference for interacting with Refine's HTTP API.

When uploading files you will need to send the data as `multipart/form-data`, e.g.:

      Content-Disposition: form-data; name="project-file"; filename="operations.json"

      Content-Disposition: form-data; name="project-name"

      myproject
      
The other operations are just normal POST parameters

## Create project:

POST /command/core/create-project-from-upload

multipart form-data:

      'project-file' : file contents...
      'project-name' : project name...
      
Returns new project ID and other metadata

## Apply operations

POST /command/core/apply-operations

multipart form-data:

      'project' : project id...
      'operations' : file contents...
      
Returns JSON response
    
## Export rows

POST /command/core/export-rows

      'engine' : JSON string... (e.g. '{"facets":[],"mode":"row-based"}')
      'project' : project id...
      'format' : format... (e.g 'tsv', 'csv')
      
Returns exported row data
    
## Delete project

POST /command/core/delete-project

      'project' : project id...
      
Returns JSON response
    
## Check status of async processes

POST /command/core/get-processes

      'project' : project id...
      
Returns JSON response
    
## Get all project metadata:
    
> **Command:** _GET /command/core/get-all-project-metadata_

Recovers the meta data for all projects. This includes the project's id, name, time of creation and last time of modification.
    
### Response:
```
{
    "projects":{
        "[project_id]":{
            "name":"[project_name]",
            "created":"[project_creation_time]",
            "modified":"[project_modification_time]"
        },
        ...[More projects]...
    }
}
```
    
## Expression Preview
> **Command:** _POST /command/core/preview-expression_

Pass some expression (GREL or otherwise) to the server where it will be executed on selected columns and the result returned.

### Parameters:
* **cellIndex**: _[column]_  
The cell/column you wish to execute the expression on.
* **expression**: _[language]_:_[expression]_  
The expression to execute. The language can either be grel, jython or clojure. Example: grel:value.toLowercase()
* **project**: _[project_id]_  
The project id to execute the expression on.
* **repeat**: _[repeat]_  
A boolean value (true/false) indicating whether or not this command should be repeated multiple times. A repeated command will be executed until the result of the current iteration equals the result of the previous iteration.
* **repeatCount**: _[repeatCount]_  
The maximum amount of times a command will be repeated.

### Response:
**On success:**
```
{
  "code": "ok",
  "results" : [result_array]
}
```

The result array will hold up to ten results, depending on how many rows there are in the project that was specified by the [project_id] parameter. Each result is the string that would be put in the cell if the GREL command was executed on that cell. Note that any expression that would return an array or JSon object will be jsonized, although the output can differ slightly from the jsonize() function.

**On error:**
```JSON
{
  "code": "error",
  "type": "[error_type]",
  "message": "[error message]"
}
```