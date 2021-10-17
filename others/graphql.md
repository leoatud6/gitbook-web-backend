# GraphQL

## Company Free Training

* hierachical
* product centric
* strong typing
* client-specified queries
* introspective

Only get what we need, not more, not less

 GraphQL--> only have one endpoint (/graphql)

 REST--> ROUTES / GRAPHQL--> QUERIES

  Can be **gateway** / **mid-layer**: between datasource and clients

 **schema** define the **types**

 **SDL**: schema definition language

#### Mutations:::

1. creating
2. updating
3. deleting

 Is only a **specification**

### **Server:**

* **a graphQL schema**
* **resolver functions**

### Client:

apollo-client:::

relay client:::

 client **libraries**: graphqa-request / apollo client / relay / urql

### Arguments & Variables

### Reading Data:::

```graphql
query{
    liftCount(status:OPEN)
}

query($status:LiftStatus){
    liftCount(status:$status)
}

query{
    liftCount(status:OPEN)
    liftCount(status:CLOSED)
}

query{
    openLifts:liftCount(status:OPEN)
    closeLifts:liftCount(status:CLOSED)
    allLifts {
        name
        liftStatus: status
    }
}

query{
    Lift(id:"panorama"){
        id
        name
        status
        trailAccess{
            id
            name
    }
}

fragment liftInfo on Lift{
    name
    status
    capacity
    night
}
```

### Writing Data

```graphql
mutation {
    setLiftStatus (id:"chenliu", status: OPEN){
        id
        name
        status
    }
}
```

 **Fields** return data in query

** Query Variables **--> pass dynamic arguments

 **mutation** --> operation used to change data

** ctrl + space** --> show all of the fields available in a query

### Objects / Types:

```graphql
ID
Float
Int
String
Boolean

all the field are non-nullable(can't return null)
```

