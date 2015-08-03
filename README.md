This document gives a brief description of the http endpoints available via the Plum API, and how to use them.

#### Table of contents  
* [Authentication](#authentication)
* [The three types](#three-types)
* [Environments](#environments)
* [Group endpoints](#group-endpoints)
    - [PUT group](#put-group)
    - [GET group](#get-group)
        + [_display action](#group-display)
        + [_profile action](#group-profile)
        + [_statistics action](#group-statistics)
        + [_artifacts action](#group-artifacts)
        + [_groups action](#group-groups)
        + [_researchers action](#group-researchers)
    - [DELETE group](#delete-group)
* [Researcher endpoints](#researcher-endpoints)
    - [PUT researcher](#put-researcher)
    - [GET researcher](#get-researcher)
        + [_display action](#researcher-display)
        + [_profile action](#researcher-profile)
        + [_statistics action](#researcher-statistics)
        + [_artifacts action](#researcher-artifacts)
    - [DELETE researcher](#delete-researcher)
* [Artifact endpoints](#artifact-endpoints)
    - [GET artifact](#get-artifact)
* [Appendix: Metric Types](#metric-types)

<a name="authenticiation"/>
# Authentication

Access to the Plum API requires an authentication token.

All calls below can be used if you pass the &auth parameter as in:

```
https://api.plu.mx/g/<your_group_here>?pretty=true&auth=<your_key_here>
```

Email us at team@plumanalytics.com to get your developer key.

<a name="three-types"/>
# The three types

The API has the following three main types:
* group - represents a division of researchers (e.g. institution, department, lab)
* researcher - represents a human that does research
* artifact - represents some work that a researcher has contributed to the world-wide body of knowledge
The api provides an HTTP interface with endpoints for manipulating these three types of objects. Objects are sent and received using content type `application/json;charset=utf-8`

<a name="environments"/>
# Environments

The Plum API can be accessed at the following environments:
* demo - https://api.plu.mx

<a name="group-endpoints"/>
# Group endpoints

The group endpoints provide functions for adding, updating, retrieving and deleting groups. Group endpoints are all accessed at `http://<environment>/g/`. Group ids must match the pattern `[a-z0-9][-a-z0-9]+[a-z0-9]`. Or in other words, they must be at least three characters long, contain only lower case letters, digits and dashes, and may not start or end with a dash. The `/` (forward slash) character is special in group ids and denotes the separation between nested groups.

<a name="put-group"/>
## PUT group

You can create or update a group using an HTTP PUT method at the `/g/` endpoint. To attempt to create a new group, the JSON object should be uploaded with a `_version` of `0`. To attempt to modify an existing group the JSON object should be uploaded with a the current `_version` of the group as retrieved using a GET command or previous PUT command response.

#### Example 1 - create new group

Here is an example of creating a new group:
```
curl -H "Content-Type: application/json" -XPUT 'https://api.plu.mx/g/test-institute-1?pretty=true' -d '
```
```json
{
  "_version":0,
  "groupType":"INSTITUTE",
  "contribution":{},
  "name":"Test Institute One Display Name",
  "description":"Short description about Test Institute One Display Name"
}
```
```
'
```
Assuming all went well, the response should look like:
```json
{
  "id" : "test-institute-1",
  "_type" : "group",
  "_version" : 1
}
```

#### Example 2 - update previously created group

Here is an example of updating the summary for the group added with the previous command:
```
curl -H "Content-Type: application/json" -XPUT 'https://api.plu.mx/g/test-institute-1?pretty=true' -d '
```
```json
{
  "_version":1,
  "groupType":"INSTITUTE",
  "contribution":{},
  "name":"Test Institute One Display Name",
  "description":"Updated short description about Test Institute One Display Name"
}
```
```
'
```
Assuming all went well, the response should look like:
```json
{
  "id" : "test-institute-1",
  "_type" : "group",
  "_version" : 2
}
```

#### Example 3 - create nested group

Here is an example of creating a new group nested under the group created above:
```
curl -H "Content-Type: application/json" -XPUT 'https://api.plu.mx/g/test-institute-1/test-department-1?pretty=true' -d '
```
```json
{
  "_version":0,
  "groupType":"DEPARTMENT",
  "contribution":{},
  "name":"Test Department One Display Name",
  "description":"Short description about Test Department One Display Name"
}
```
```
'
```
Assuming all went well, the response should look like:
```json
{
  "id" : "test-institute-1/test-department-1",
  "_type" : "group",
  "_version" : 1
}
```
Note that the `id` field is optional in the PUT body and is omitted in this example. If the `id` property is included in the PUT body, no harm is done, but it is ignored, and the value from the id in the URL is used when the object is actually stored.

#### Example 4 - cascade create

If a nested group is created and it's parent groups do not exist, stubs will be generated for the parent groups. Here is an example of cascade creation using a child group with two ancestors that don't exist:
```
curl -H "Content-Type: application/json" -XPUT 'https://api.plu.mx/g/test-institute-2/test-department-2/test-lab-2?pretty=true' -d '
```
```json
{
  "_version":0,
  "groupType":"LAB",
  "contribution":{},
  "name":"Test Lab Two Display Name",
  "description":"Short description about Test Lab Two Display Name"
}
```
```
'
```
Assuming all went well, the response should look like:
```json
{
  "id" : "test-institute-2/test-department-2/test-lab-2",
  "_type" : "group",
  "_version" : 1
}
```
If you browse the objects in elastic search (use mobz head, or GET commands), you will notice that `test-institute-2` and `test-department-2` were created automatically for you.

<a name="get-group"/>
## GET group

You can retrieve a group using an HTTP GET method at the `/g/` endpoint. GET returns the raw JSON object as it was last uploaded. It will include the `_version` of the currently stored object.

#### Example 1 - get test-institute-1

Assuming the data entered in the PUT section exists, you could enter the following command:
```
curl -XGET 'https://api.plu.mx/g/test-institute-1?pretty=true'
```
Assuming all went well, the response should look like:
```json
{
  "id" : "test-institute-1",
  "_version" : 2,
  "groupType" : "INSTITUTE",
  "description" : "Updated short description about Test Institute One Display Name",
  "name" : "Test Institute One Display Name",
  "parent" : null,
  "ancestor" : [ ]
}
```

#### Example 2 - trying to retrieve data that doesn't exist

You could try and get a non-existant object using the following command:
```
curl -XGET 'https://api.plu.mx/g/this-doesnt-exist?pretty=true'
```
Assuming all went well, the response should look like:
```json
{
  "message" : "Item not found.",
  "httpStatus" : 404,
  "errorCode" : "NOT_FOUND",
  "isSuccess" : false
}
```

The GET method allows special actions to be performed besides the default action which is retrieving the raw JSON object. Actions are denoted using a suffix after the id which is prefixed with the `_` (underscore) character. 

<a name="group-display"/>
### _display action

The `_display` action fetches additional data that will be required to build a display page for the current group. The additional data includes the following:
* the JSON for any ancestors of the current group
* the JSON for any direct child groups of the current group
* the JSON for counts about artifacts that have the current group in their ancestor hierarchy
* the JSON for any artifacts that have the current group in their ancestor hierarchy
* the JSON for any researchers that have the current group in their ancestor hierarchy

#### Example - using `_display` to retrieve test-lab-2

Here is an example of retrieving test-lab-2 which shows the ancestors that are also retrieved:

```
curl -XGET 'https://api.plu.mx/g/test-institute-2/test-department-2/test-lab-2/_display?pretty=true'
```
Assuming all went well, the response should look like:
```json
{
  "id" : "test-institute-2/test-department-2/test-lab-2",
  "_version" : 1,
  "groupType" : "LAB",
  "description" : "Short description about Test Lab Two Display Name",
  "name" : "Test Lab Two Display Name",
  "parent" : "test-institute-2/test-department-2",
  "ancestor" : [ {
    "id" : "test-institute-2",
    "_version" : 1,
    "parent" : null,
    "ancestor" : [ ]
  }, {
    "id" : "test-institute-2/test-department-2",
    "_version" : 1,
    "parent" : "test-institute-2",
    "ancestor" : [ "test-institute-2" ]
  } ],
  "children" : [ ],
  "statistics" : {
    "artifactType" : {
      "total" : 0,
      "details" : {
      }
    },
    "capture" : {
      "total" : 0,
      "details" : {
      }
    },
    "usage" : {
      "total" : 0,
      "details" : {
      }
    },
    "socialMedia" : {
      "total" : 0,
      "details" : {
      }
    },
    "mention" : {
      "total" : 0,
      "details" : {
      }
    },
    "citation" : {
      "total" : 0,
      "details" : {
      }
    }
  },
  "document" : [ ],
  "researcher" : [ ]
}
```
Notice that this returns more than a raw GET request for test-lab-2 when `/_display` is added.

<a name="group-profile"/>
### _profile action

The `_profile` action fetches additional data that will be required to build the profile portion of a display page for the current group. The additional data includes the JSON for any ancestors of the current group. The `_profile` action has no parameters.

#### Example - using `_profile` to retrieve test-lab-2

Here is an example of retrieving test-lab-2 which shows the ancestors that are also retrieved:

```
curl -XGET 'https://api.plu.mx/g/test-institute-2/test-department-2/test-lab-2/_profile?pretty=true'
```
Assuming all went well, the response should look like:
```json
{
  "id" : "test-institute-2/test-department-2/test-lab-2",
  "_version" : 1,
  "groupType" : "LAB",
  "description" : "Short description about Test Lab Two Display Name",
  "name" : "Test Lab Two Display Name",
  "parent" : "test-institute-2/test-department-2",
  "ancestor" : [ {
    "id" : "test-institute-2",
    "_version" : 1,
    "parent" : null,
    "ancestor" : [ ]
  }, {
    "id" : "test-institute-2/test-department-2",
    "_version" : 1,
    "parent" : "test-institute-2",
    "ancestor" : [ "test-institute-2" ]
  } ]
}
```

<a name="group-statistics"/>
### _statistics action

The `_statistics` action fetches additional data that will be required to build the statistics portion of a display page for the current group. The additional data includes the aggregate counts of any articles that have the current group in their hierarchy. The `_statistics` action accepts the `filter` parameter which can contain a Lucene style string query ([see Lucene documentation](http://lucene.apache.org/core/3_6_1/queryparsersyntax.html)).

e.g. `https://api.plu.mx/g/test-institute-2/test-department-2/test-lab-2/_statistics?filter=usage.countType:VIEW_COUNT`

#### Example - using `_statistics` to retrieve test-lab-2

Here is an example of retrieving test-lab-2 which shows the statistics that are also retrieved:

```
curl -XGET 'https://api.plu.mx/g/test-institute-2/test-department-2/test-lab-2/_statistics?pretty=true'
```
Assuming all went well, the response should look like:
```json
{
  "statistics" : {
    "artifactType" : {
      "total" : 0,
      "details" : {
      }
    },
    "capture" : {
      "total" : 0,
      "details" : {
      }
    },
    "usage" : {
      "total" : 0,
      "details" : {
      }
    },
    "socialMedia" : {
      "total" : 0,
      "details" : {
      }
    },
    "mention" : {
      "total" : 0,
      "details" : {
      }
    },
    "citation" : {
      "total" : 0,
      "details" : {
      }
    }
  }
}
```

<a name="group-artifacts"/>
### _artifacts action

The `_artifacts` action fetches additional data that will be required to build the documents portion of a display page for the current group. The additional data includes the documents that have the current group in their hierarchy. The `_artifacts` action accepts the following parameters:
* `filter` which can contain a Lucene style string query ([see Lucene documentation](http://lucene.apache.org/core/3_6_1/queryparsersyntax.html))
* `offset` which specifies an offset into the result set
* `size` which specifies how many documents to return
* `page` which specifies which page of documents to return

If both `offset` and `page` are specified `offset` will take precedence.

e.g. `https://api.plu.mx/g/test-institute-2/test-department-2/test-lab-2/_artifacts?filter=usage.countType:VIEW_COUNT&offset=0&size=10`

#### Example - using `_artifacts` to retrieve test-lab-2

Here is an example of retrieving test-lab-2 which shows the documents that are also retrieved:

```
curl -XGET 'https://api.plu.mx/g/test-institute-2/test-department-2/test-lab-2/_artifacts?pretty=true'
```
Assuming all went well, the response should look like:
```json
{
  "total" : 0,
  "document" : [ ]
}
```

<a name="group-groups"/>
### _groups action

The `_groups` action fetches additional data that will be required to build the children portion of a display page for the current group. The additional data includes the groups that have the current group as their parent. The `_groups` action accepts the following parameters:
* `offset` which specifies an offset into the result set
* `size` which specifies how many documents to return
* `page` which specifies which page of documents to return

If both `offset` and `page` are specified `offset` will take precedence.

e.g. `https://api.plu.mx/g/test-institute-2/_groups?offset=0&size=10`

#### Example - using `_groups` to retrieve test-institute-2

Here is an example of retrieving test-institute-2 which shows the groups that are also retrieved:

```
curl -XGET 'https://api.plu.mx/g/test-institute-2/_groups?pretty=true'
```
Assuming all went well, the response should look like:
```json
{
  "total" : 1,
  "children" : [ {
    "id" : "test-institute-2/test-department-2",
    "_version" : 1,
    "parent" : "test-institute-2",
    "ancestor" : [ "test-institute-2" ]
  } ]
}
```

<a name="group-researchers"/>
### _researchers action

The `_researchers` action fetches additional data that will be required to build the researchers portion of a display page for the current group. The additional data includes the researchers that have the current group as their parent. The `_researchers` action accepts the following parameters:
* `offset` which specifies an offset into the result set
* `size` which specifies how many documents to return
* `page` which specifies which page of documents to return

If both `offset` and `page` are specified `offset` will take precedence.

e.g. `https://api.plu.mx/g/test-institute-2/test-department-2/test-lab-2/_researchers?offset=0&size=10`

#### Example - using `_researchers` to retrieve test-lab-2

Here is an example of retrieving test-institute-2 which shows the researchers that are also retrieved:

```
curl -XGET 'https://api.plu.mx/g/test-institute-2/test-department-2/test-lab-2/_researchers?pretty=true'
```
Assuming all went well, the response should look like:
```json
{
  "total" : 0,
  "researcher" : [ ]
}
```

<a name="delete-group"/>
## DELETE group

You can remove a group using an HTTP DELETE method at the `/g/` endpoint. 

#### Example 1 - remove test-department-1

Assuming the data entered in the PUT section exists, you could enter the following command:
```
curl -XDELETE 'https://api.plu.mx/g/test-department-1?pretty=true'
```
Assuming all went well, the response should look like:
```json
{
  "id" : "test-department-1",
  "_type" : "group",
  "_version" : 1
}
```

#### Example 2 - cascade delete

If a parent group has nested groups and the parent is removed using the HTTP DELETE method then all child groups of this parent will be removed. Child groups cannot exist without their ancestors because there is no way to find or navigate to them. Note that this does not remove any researchers or artifacts that may have the parent group as an ancestor. Researchers and artifacts may reference groups that do not exist, but they cannot be navigated to via the missing hierarchy. Here is an example of issuing a cascade delete:
```
curl -XDELETE 'https://api.plu.mx/g/test-institute-2?pretty=true'
```
Assuming all went well, the response should look like:
```json
{
  "id" : "test-institute-2",
  "_type" : "group",
  "_version" : 1
}
```
If you browse the objects in elastic search (use mobz head, or GET commands), you will notice that `test-department-2` and `test-lab-2` were destroyed along with `test-institute-2`.

<a name="researcher-endpoints"/>
# Researcher endpoints

The researcher endpoints provide functions for adding, updating, retrieving and deleting researchers. Researcher endpoints are all accessed at `http://<environment>/r/`. Researcher ids must match the pattern `[a-z0-9][-a-z0-9]+[a-z0-9]`. Or in other words, they must be at least three characters long, contain only lower case letters, digits and dashes, and may not start or end with a dash. Unlike groups researchers cannot be nested and they all have unique ids directly under `/r/`. Researchers may have zero or more groups as parents.

<a name="put-researcher"/>
## PUT researcher

You can create or update a researcher using an HTTP PUT method at the `/r/` endpoint. To attempt to create a new researcher, the JSON object should be uploaded with a `_version` of `0`. To attempt to modify an existing researcher the JSON object should be uploaded with a the current `_version` of the researcher as retrieved using a GET command or previous PUT command response.

#### Example 1 - create new researcher

Here is an example of creating a new researcher that is not associated with any groups:
```
curl -H "Content-Type: application/json" -XPUT 'https://api.plu.mx/r/test-researcher-1?pretty=true' -d '
```
```json
{         
  "_version":0,
  "parent":[],
  "contribution":{
    "doi": ["10.1260/095830507782616887", "10.1177/0273475306288399"]
  },   
  "personData":{
    "name": {"display": "Test Researcher1"},
    "headline":"Short biography for Test Researcher1",
    "profileLink": [{
      "uri": "http://url.of.test-institute1.com/department1/lab1/researcher1",
      "profileLinkCategory": "PORTFOLIO",
      "profileLinkType": "UNKNOWN"                                        
    }] 
  }       
} 
```
```
'
```
Assuming all went well, the response should look like:
```json
{
  "id" : "test-researcher-1",
  "_type" : "researcher",
  "_version" : 1
}
```

#### Example 2 - update previously created researcher

Assuming the data from the previous PUT researcher command, here is an example of adding the researcher to a group:
```
curl -H "Content-Type: application/json" -XPUT 'https://api.plu.mx/r/test-researcher-1?pretty=true' -d '
```
```json
{
  "_version":1,
  "type":"researcher",
  "parent":[
    "test-institute-1/test-department-1/test-lab-1"
  ],
  "contribution":[
    {
      "id":"10.1260/095830507782616887",
      "type":"doi"
    },
    {
      "id":"10.1177/0273475306288399",
      "type":"doi"
    }
  ],
  "profile":{
    "name":"Test Researcher1",
    "summary":"Short biography for Test Researcher1",
    "url":"http://url.of.test-institute1.com/department1/lab1/researcher1"
  }
}
```
```
'
```
Assuming all went well, the response should look like:
```json
{
  "id" : "test-researcher-1",
  "_type" : "researcher",
  "_version" : 2
}
```
If `test-institute-1/test-department-1/test-lab-1` existed then `test-researcher-1` would now be returned as part of it's `_display` action.

<a name="get-researcher"/>
## GET researcher

You can retrieve a researcher using an HTTP GET method at the `/r/` endpoint. GET returns the raw JSON object as it was last uploaded. It will include the `_version` of the currently stored object.

#### Example 1 - get test-researcher-1

Assuming the data entered in the PUT section exists, you could enter the following command:
```
curl -XGET 'https://api.plu.mx/r/test-researcher-1?pretty=true'
```
Assuming all went well, the response should look like:
```json
{
  "id" : "test-researcher-1",
  "_version" : 2,
  "contribution" : [ {
    "id" : "10.1260/095830507782616887",
    "type" : "doi"
  }, {
    "id" : "10.1177/0273475306288399",
    "type" : "doi"
  } ],
  "parent" : [ "test-institute-1/test-department-1/test-lab-1" ],
  "type" : "researcher",
  "ancestor" : [ "test-institute-1", "test-institute-1/test-department-1/test-lab-1", "test-institute-1/test-department-1" ],
  "profile" : {
    "summary" : "Short biography for Test Researcher1",
    "name" : "Test Researcher1",
    "url" : "http://url.of.test-institute1.com/department1/lab1/researcher1"
  }
}
```

#### Example 2 - trying to retrieve data that doesn't exist

You could try and get a non-existant object using the following command:
```
curl -XGET 'https://api.plu.mx/r/this-doesnt-exist?pretty=true'
```
Assuming all went well, the response should look like:
```json
{
  "message" : "Item not found.",
  "httpStatus" : 404,
  "errorCode" : "NOT_FOUND",
  "isSuccess" : false
}
```

<a name="researcher-display"/>
### _display action

The GET method allows special actions to be performed besides the default action which is retrieving the raw JSON object. Actions are denoted using a suffix after the id which is prefixed with the `_` (underscore) character. The only action currently supported for the GET researcher method is the `_display` action. The `_display` action fetches additional data that will be required to build a display page for the current researcher. The additional data includes the JSON for any parent groups of the current researcher. It also includes JSON for any artifacts that have the current researcher in their ancestor hierarchy.

#### Example 1 - using _display to retrieve test-researcher-1

Here is an example of retrieving `test-researcher-1` which shows the parent groups that are also retrieved:

```
curl -XGET 'https://api.plu.mx/r/test-researcher-1/_display?pretty=true'
```
Assuming all went well, the response should look like:
```json
{
  "id" : "test-researcher-1",
  "artifact" : [ ],
  "_version" : 2,
  "contribution" : [ {
    "id" : "10.1260/095830507782616887",
    "type" : "doi"
  }, {
    "id" : "10.1177/0273475306288399",
    "type" : "doi"
  } ],
  "parent" : [ "test-institute-1/test-department-1/test-lab-1" ],
  "type" : "researcher",
  "ancestor" : [ {
    "id" : "test-institute-1",
    "_version" : 1,
    "contribution" : [ ],
    "parent" : null,
    "type" : "institution",
    "ancestor" : [ ],
    "profile" : {
      "summary" : "Short description about Test Institute One Display Name",
      "name" : "Test Institute One Display Name",
      "url" : "http://url.of.test-institute1.com"
    }
  }, {
    "id" : "test-institute-1/test-department-1/test-lab-1",
    "_version" : 1,
    "contribution" : [ ],
    "parent" : "test-institute-1/test-department-1",
    "type" : "lab",
    "ancestor" : [ "test-institute-1", "test-institute-1/test-department-1" ],
    "profile" : {
      "summary" : "Short description about Test Lab One Display Name",
      "name" : "Test Lab One Display Name",
      "url" : "http://url.of.test-institute1.com/department1/lab1"
    }
  }, {
    "id" : "test-institute-1/test-department-1",
    "_version" : 1,
    "contribution" : [ ],
    "parent" : "test-institute-1",
    "type" : "department",
    "ancestor" : [ "test-institute-1" ],
    "profile" : {
      "summary" : "Short description about Test Department One Display Name",
      "name" : "Test Department One Display Name",
      "url" : "http://url.of.test-institute1.com/department1"
    }
  } ],
  "profile" : {
    "summary" : "Short biography for Test Researcher1",
    "name" : "Test Researcher1",
    "url" : "http://url.of.test-institute1.com/department1/lab1/researcher1"
  }
}
```
Notice that this returns more than a raw GET request for `test-researcher-1` when `/_display` is added.

#### Example 2 - using _display to retrieve test-researcher-1 via a hierarchy

You can retrieve a researcher via a specific group hierarchy that is in the researchers parent tree. Here is an example of retrieving `test-researcher-1` via one of it's ancestors -- `test-institute-1`:

```
curl -XGET 'https://api.plu.mx/r/test-institute-1/test-researcher-1/_display?pretty=true'
```
Assuming all went well, the response should look like:
```json
{
  "id" : "test-researcher-1",
  "artifact" : [ ],
  "_version" : 2,
  "contribution" : [ {
    "id" : "10.1260/095830507782616887",
    "type" : "doi"
  }, {
    "id" : "10.1177/0273475306288399",
    "type" : "doi"
  } ],
  "parent" : [ "test-institute-1/test-department-1/test-lab-1" ],
  "type" : "researcher",
  "ancestor" : [ {
    "id" : "test-institute-1",
    "_version" : 1,
    "contribution" : [ ],
    "parent" : null,
    "type" : "institution",
    "ancestor" : [ ],
    "profile" : {
      "summary" : "Short description about Test Institute One Display Name",
      "name" : "Test Institute One Display Name",
      "url" : "http://url.of.test-institute1.com"
    }
  } ],
  "profile" : {
    "summary" : "Short biography for Test Researcher1",
    "name" : "Test Researcher1",
    "url" : "http://url.of.test-institute1.com/department1/lab1/researcher1"
  }
}
```
Notice a few things about this:
* only the ancestors in the current parent tree context are returned
* if `test-researcher-1` did not have `test-institute-1` in it's parentage tree then this would return a 404
* it takes into account the whole tree of the parent group. In this case `test-lab-1` is the parent, but it can be retrieved via `test-institute-1` or `test-department-1` because they are ancestors of `test-lab-1`


<a name="researcher-profile"/>
### _profile action


<a name="researcher-statistics"/>
### _statistics action


<a name="researcher-artifacts"/>
### _artifacts action


<a name="delete-researcher"/>
## DELETE researcher

You can remove a researcher using an HTTP DELETE method at the `/r/` endpoint. 

#### Example - remove test-researcher-1

Assuming the data entered in the PUT section exists, you could enter the following command:
```
curl -XDELETE 'https://api.plu.mx/r/test-researcher-1?pretty=true'
```
Assuming all went well, the response should look like:
```json
{
  "id" : "test-researcher-1",
  "_type" : "researcher",
  "_version" : 3
}
```

<a name="artifact-endpoints"/>
# Artifact endpoints

Under normal operation artifacts enter the system via background crawl processes. Thus, there are no endpoints for creating, updating, or deleting artifacts. The only endpoint for artifacts is GET for retrieving them. Artifact endpoints are accessed at `http://<environment>/a/`. Artifact ids are automatically generated by the backend systems and have a form matching `[-+a-zA-Z0-9]+`. Artifacts may have zero or more groups or researchers as parents, and these relationships are managed by backend processes.

<a name="get-artifact"/>
## GET artifact

You can retrieve an artifact using an HTTP GET method at the `/a/` endpoint. GET returns the raw JSON object as it was last uploaded. It will include the `_version` of the currently stored object, but versions are unimportant for artifacts because they do not support optimistic concurrency.

#### Example 1 - get test-artifact-1

Assuming artifact `sK30+b2g-N` exists, you could enter the following command:
```
curl -XGET 'https://api.plu.mx/a/sK30+b2g-N?pretty=true'
```
Assuming all went well, the response should look like:
```json
{
  "id" : "sK30+b2g-N",
  "_version" : 1,
  "data" : {
    "summary" : "Short abstract for Journal Article One Title",
    "author" : [ {
      "lastName" : "Researcher1",
      "fullName" : "Test Researcher1",
      "firstName" : "Test"
    }, {
      "lastName" : "Kirk",
      "fullName" : "James T Kirk",
      "firstName" : "James Tiberius"
    } ],
    "issue" : "37",
    "name" : "Journal Article One Title",
    "publicationTitle" : "Fake Quarterly",
    "volume" : "vol 2",
    "counts" : [ {
      "category" : "usage",
      "source" : "Mendeley",
      "value" : 17,
      "type" : "readers"
    } ],
    "contentType" : "Journal Article",
    "publicationDate" : "2010-02",
    "doi" : "10.1177/0273475306288399",
    "url" : "http://path.to.artifact.com/doi/10.1177/0273475306288399"
  },
  "parent" : [ "test-researcher-1" ],
  "ancestor" : [ "test-researcher-1" ]
}
```

<a name="metric-types"/>
# Appendix: Metric Types

      BOOKMARK_COUNT        => Bookmarks
      FAVORITE_COUNT        => Favorites
      FOLLOWER_COUNT        => Followers
      FORK_COUNT            => Forks
      GROUP_COUNT           => Groups
      READER_COUNT          => Readers
      SUBSCRIBER_COUNT      => Subscribers
      WATCHER_COUNT         => Watchers
      LIKE_COUNT            => Likes
      PLUS_ONE_COUNT        => +1s
      RATING_COUNT          => Ratings
      RECOMMENDATION_COUNT  => Recommendations
      SCORE_COUNT           => Score
      TWEET_COUNT           => Tweets
      SHARE_COUNT           => Shares
      ABSTRACT_VIEWS        => Abstract Views
      COLLABORATOR_COUNT    => Collaborators
      DOWNLOAD_COUNT        => Downloads
      FIGURE_VIEWS          => Figure Views
      FULL_TEXT_VIEWS       => Full Text Views
      HOLDINGS_COUNT        => Holdings
      HTML_VIEWS            => HTML Views
      XML_VIEWS             => XML Views
      LINK_CLICK_COUNT      => Clicks
      PDF_DOWNLOADS         => PDF Downloads
      PDF_VIEWS             => PDF Views
      PLAY_COUNT            => Counts
      SUPPORTING_DATA_VIEWS => Data Views
      UNIQUE_IP_VIEWS       => Unique IP Views
      VIEW_COUNT            => Views
      COMMENT_COUNT         => Comments
      FORUM_TOPIC_COUNT     => Forum Topics
      GIST_COUNT            => Gists
      LINK_COUNT            => Links
      REVIEW_COUNT          => Reviews
      BLOG_COUNT            => Economics Blog Mentions
      ALL_BLOG_COUNT        => Blog Mentions
      CITED_BY_COUNT        => Citations
      CITED_BY_COUNT_NC     => Citations
      EPUB_DOWNLOAD_COUNT   => ePub Downloads
      EMAILS                => Emails
      EXPORTS               => Exports
      PRINT_OUTS            => Print-outs
      SAVES                 => Saves
      LINK_OUTS             => Link-outs
      SAMPLE_DOWNLOADS      => Sample Downloads
      EXPORTS_SAVES         => Exports-Saves
