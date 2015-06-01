
###start with a shell script###
'mongo --shell UpdAndDelTest.js'

###load data###

'''
S:\MongoDB>mongo --shell UpdAndDelTest.js
MongoDB shell version: 3.0.3
connecting to: test
type "help" for help
> prepareTestData()
Inserted 20 documents in updAndDelTest
> db.updAndDelTest.find({}, {_id:0})
{ "x" : 1, "y" : 1 }
{ "x" : 1, "y" : 2 }
{ "x" : 1, "y" : 3 }
{ "x" : 1, "y" : 4 }
{ "x" : 1, "y" : 5 }
{ "x" : 1, "y" : 6 }
{ "x" : 1, "y" : 7 }
{ "x" : 1, "y" : 8 }
{ "x" : 1, "y" : 9 }
{ "x" : 1, "y" : 10 }
{ "x" : 2, "y" : 1 }
{ "x" : 2, "y" : 2 }
{ "x" : 2, "y" : 3 }
{ "x" : 2, "y" : 4 }
{ "x" : 2, "y" : 5 }
{ "x" : 2, "y" : 6 }
{ "x" : 2, "y" : 7 }
{ "x" : 2, "y" : 8 }
{ "x" : 2, "y" : 9 }
{ "x" : 2, "y" : 10 }



> db.updAndDelTest.update({x:1}, {$set:{y:0}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.updAndDelTest.find({x:1}, {_id:0})
{ "x" : 1, "y" : 0 }
{ "x" : 1, "y" : 2 }
{ "x" : 1, "y" : 3 }
{ "x" : 1, "y" : 4 }
{ "x" : 1, "y" : 5 }
{ "x" : 1, "y" : 6 }
{ "x" : 1, "y" : 7 }
{ "x" : 1, "y" : 8 }
{ "x" : 1, "y" : 9 }
{ "x" : 1, "y" : 10 }
>
'''
'db.updAndDelTest.update({x:1}, {$set:{y:0}})', it only updates the first document that matches the query provided as the first parameter. 

The update function has the ***db.<collection name>.update(query, update object, isUpsert, isMulti)*** format.

Removals, on other hand, behave differently. By default, the remove operation deletes all the documents that match the provided query. However, if we wish to delete only one document, we will explicitly pass the second parameter as true.

###Index###
We will now create an index on the state and pincode fields as follows:
> db.postalCodes.ensureIndex({state:1, pincode:1})

> db.postalCodes.find({state:'Maharashtra'}).explain()

```

       "queryPlanner" : {
               "plannerVersion" : 1,
               "namespace" : "test.postalCodes",
               "indexFilterSet" : false,
               "parsedQuery" : {
                       "state" : {
                               "$eq" : "Maharashtra"
                       }
               },
               "winningPlan" : {
                       "stage" : "FETCH",
                       "inputStage" : {
                               "stage" : "IXSCAN",
                             ***  "keyPattern" : {
                                       "state" : 1,
                                       "pincode" : 1
                               },
                             ***  "indexName" : "state_1_pincode_1", ***
                               "isMultiKey" : false,
                               "direction" : "forward",
                               "indexBounds" : {
                                       "state" : [
                                               "[\"Maharashtra\", \"Maharashtra\"]"
                                       ],
                                       "pincode" : [
                                               "[MinKey, MaxKey]"
                                       ]
                               }
                       }
               },
               "rejectedPlans" : [ ]
       },
       "serverInfo" : {
               "host" : "eagleii",
               "port" : 27017,
               "version" : "3.0.3",
               "gitVersion" : "b40106b36eecd1b4407eb1ad1af6bc60593c6105"
       },
       "ok" : 1
```


Some of the operators that do not use the index efficiently are the $where, $nin, and $exists operators. 
Whenever these operators are used in the query, one should bear in mind a possible performance bottleneck when the data size increases. 
Similarly, the $in operator must be preferred over the $or operator, as both can be more or less used to achieve the same result.


db.postalCodes.find({state:'Maharashtra'}, {pincode:1, _id:0}).explain()

###Create an index in the background###
Create the index again but, this time, in the background:
> db.indexTest.ensureIndex({value:1}, {background:true})
