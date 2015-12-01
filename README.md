# PocoDynamo

is a highly productive, feature-rich, typed .NET client which extends 
[ServiceStack's Simple POCO life](http://stackoverflow.com/a/32940275/85785) 
by enabling re-use of your code-first data models with Amazon's industrial strength and highly-scalable 
NoSQL [DynamoDB](https://aws.amazon.com/dynamodb/).

#### First class support for reusable, code-first POCOs

PocoDynamo is conceptually similar to ServiceStack's other code-first
[OrmLite](https://github.com/ServiceStack/ServiceStack.OrmLite) and 
[Redis](https://github.com/ServiceStack/ServiceStack.Redis) clients by providing a high-fidelity, managed client that enhances
AWSSDK's low-level [IAmazonDynamoDB client](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/UsingAWSsdkForDotNet.html), 
with rich, native support for intuitively mapping your re-usable code-first POCO Data models into 
[DynamoDB Data Types](http://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_Types.html). 

![](https://raw.githubusercontent.com/ServiceStack/Assets/master/img/aws/pocodynamo/related-customer.png)

## Features

#### Advanced idiomatic .NET client

PocoDynamo provides an idiomatic API that leverages .NET advanced language features with streaming API's returning
`IEnumerable<T>` lazily evaluated responses that transparently performs multi-paged requests 
behind-the-scenes whilst the resultset is iterated. It high-level API's provides a clean lightweight adapter to
transparently map between .NET built-in data types and DynamoDB's low-level attribute values. Its efficient batched 
API's take advantage of DynamoDB's `BatchWriteItem` and `BatchGetItem` batch operations to perform the minimum number 
of requests required to implement each API.

#### Typed, LINQ provider for Query and Scan Operations

PocoDynamo also provides rich, typed LINQ-like querying support for constructing DynamoDB Query and Scan operations, 
dramatically reducing the effort to query DynamoDB, enhancing readability whilst benefiting from Type safety in .NET. 

#### Declarative Tables and Indexes

Behind the scenes DynamoDB is built on a dynamic schema which whilst open and flexible, can be cumbersome to work with 
directly in typed languages like C#. PocoDynamo bridges the gap and lets your app bind to impl-free and declarative POCO 
data models that provide an ideal high-level abstraction for your business logic, hiding a lot of the complexity of 
working with DynamoDB - dramatically reducing the code and effort required whilst increasing the readability and 
maintainability of your Apps business logic.

It includes optimal support for defining simple local indexes which only require declaratively annotating properties 
to index with an `[Index]` attribute.

Typed POCO Data Models can be used to define more complex Local and Global DynamoDB Indexes by implementing 
`IGlobalIndex<Poco>` or `ILocalIndex<Poco>` interfaces which PocoDynamo uses along with the POCOs class structure 
to construct Table indexes at the same time it creates the tables.

In this way the Type is used as a DSL to define DynamoDB indexes where the definition of the index is decoupled from 
the imperative code required to create and query it, reducing the effort to create them whilst improving the 
visualization and understanding of your DynamoDB architecture which can be inferred at a glance from the POCO's 
Type definition. PocoDynamo also includes first-class support for constructing and querying Global and Local Indexes 
using a familiar, typed LINQ provider.

#### Resilient

Each operation is called within a managed execution which transparently absorbs the variance in cloud services 
reliability with automatic retries of temporary errors, using an exponential backoff as recommended by Amazon. 

#### Enhances existing APIs

PocoDynamo API's are a lightweight layer modeled after DynamoDB API's making it predictable the DynamoDB operations 
each API calls under the hood, retaining your existing knowledge investment in DynamoDB. 
When more flexibility is needed you can access the low-level `AmazonDynamoDBclient from the `IPocoDynamo.DynamoDb` 
property and talk with it directly.

Whilst PocoDynamo doesn't save you for needing to learn DynamoDB, its deep integration with .NET and rich support for 
POCO's smoothes out the impedance mismatches to enable an type-safe, idiomatic, productive development experience.

#### High-level features

PocoDynamo includes its own high-level features to improve the re-usability of your POCO models and the development 
experience of working with DynamoDB with support for Auto Incrementing sequences, Query expression builders, 
auto escaping and converting of Reserved Words to placeholder values, configurable converters, scoped client 
configurations, related items, conventions, aliases, dep-free data annotation attributes and more.

## Download

PocoDynamo is contained in ServiceStack's AWS NuGet package:

    PM> Install-Package ServiceStack.Aws
   
<sub>PocoDynamo has a 10 Tables [free-quota usage](https://servicestack.net/download#free-quotas) limit which can be unlocked with a [commercial license key](https://servicestack.net/pricing).</sub>
    
To get started we'll need to create an instance of `AmazonDynamoDBClient` with your AWS credentials and Region info:

```csharp
var awsDb = new AmazonDynamoDBClient(AWS_ACCESS_KEY, AWS_SECRET_KEY, RegionEndpoint.USEast1);
```

Then to create a PocoDynamo client pass the configured AmazonDynamoDBClient instance above:

```csharp
var db = new PocoDynamo(awsDb);
```

> Clients are Thread-Safe so you can register them as a singleton and share the same instance throughout your App

### [Source Code](https://github.com/ServiceStack/ServiceStack.Aws/tree/master/src/ServiceStack.Aws/DynamoDb)

The Source Code for PocoDynamo is maintained in [ServiceStack.Aws](https://github.com/ServiceStack/ServiceStack.Aws/) repository.

### Download Local DynamoDB

It's recommended to download [local DynamoDB](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Tools.DynamoDBLocal.html#Tools.DynamoDBLocal.DownloadingAndRunning)
as it lets you develop against a local DynamoDB instance, saving you needing a network connection or AWS account.

You can connect to your local DynamoDB instance by configuring the `AmazonDynamoDBClient` to point to the default url
where Local DynamoDB instance is running:

```csharp
var awsDb = new AmazonDynamoDBClient("keyId", "key", new AmazonDynamoDBConfig {
    ServiceURL = "http://localhost:8000",
});

var db = new PocoDynamo(awsDb);
```

We've found the latest version of Local DynamoDB to be a robust and fast substitute for AWS, that eliminates waiting 
times for things like creating and dropping tables whilst only slightly deviating from the capabilities of AWS where 
it doesn't always include the additional limitations imposed when hosted on AWS.

## Usage

To illustrate how PocoDynamo simplifies working with DynamoDB, we'll walk-through creating and retrieving the Simple 
[Todo model](https://github.com/ServiceStackApps/AwsApps/blob/04dea6472fd73ea2e55f1aa748fff6e8784b339c/src/AwsApps/todo/TodoService.cs#L9)
used in the [DynamoDB-powered AWS Todo Example](http://awsapps.servicestack.net/todo/) and compare it against the code
required when using AWSSDK's `IAmazonDynamoDB` client directly.

The simple `Todo` POCO is the same data model used to store TODO's in every major RDBMS's with 
[OrmLite](https://github.com/ServiceStack/ServiceStack.OrmLite), in Redis with 
[ServiceStack.Redis](https://github.com/ServiceStack/ServiceStack.Redis) as well as 
every supported [Caching provider](https://github.com/ServiceStack/ServiceStack/wiki/Caching). 

PocoDynamo increases the re-use of `Todo` again which can now be used to store TODO's in DynamoDB as well: 

```csharp
public class Todo
{
    [AutoIncrement]
    public long Id { get; set; }
    public string Content { get; set; }
    public int Order { get; set; }
    public bool Done { get; set; }
}
```

### Creating a Table with PocoDynamo

PocoDynamo enables a declarative code-first approach where it's able to create DynamoDB Table schemas from just your 
POCO class definition. Whilst you could call `db.CreateTable<Todo>()` API and create the Table directly, the recommended 
approach is instead to register all the tables your App uses with PocoDynamo on Startup, then just call `InitSchema()` 
which will go through and create all missing tables:

```csharp
//PocoDynamo
var db = new PocoDynamo(awsDb)
    .RegisterTable<Todo>();

db.InitSchema();

db.GetTableNames().PrintDump();
```

In this way your App ends up in the same state with all tables created if it was started with **no tables**, **all tables** 
or only a **partial list** of tables. After the tables are created we query DynamoDB to dump its entire list of Tables, 
which if you started with an empty DynamoDB instance would print the single **Todo** table name to the Console:

    [
        Todo
    ]
    
### Complete PocoDynamo TODO example

Before going through the details of how it all works under-the-hood, here's a quick overview of what it looks likes to 
use PocoDynamo for developing a simple CRUD App. The ServiceStack 
[TodoService](https://github.com/ServiceStackApps/AwsApps/blob/master/src/AwsApps/todo/TodoService.cs) 
below contains the full server implementation required to implement the REST API to power 
[Backbone's famous TODO App](http://todomvc.com/examples/backbone/), rewritten to store all TODO items in DynamoDB:

```csharp
//PocoDynamo
public class TodoService : Service
{
    public IPocoDynamo Dynamo { get; set; }

    public object Get(Todo todo)
    {
        if (todo.Id != default(long))
            return Dynamo.GetItem<Todo>(todo.Id);

        return Dynamo.GetAll<Todo>();
    }

    public Todo Post(Todo todo)
    {
        Dynamo.PutItem(todo);
        return todo;
    }

    public Todo Put(Todo todo)
    {
        return Post(todo);
    }

    public void Delete(Todo todo)
    {
        Dynamo.DeleteItem<Todo>(todo.Id);
    }
}
```

We can see `IPocoDynamo` is just a normal IOC dependency that provides high-level API's that work directly with POCO's 
and built-in .NET data types, enabling the minimum effort to store, get and delete data from DynamoDB.

### Creating a DynamoDB Table using AmazonDynamoDBClient

The equivalent imperative code to create the Todo DynamoDB table above would require creating executing the 
`CreateTableRequest` below:

```csharp
//AWSSDK
var request = new CreateTableRequest
{
    TableName = "Todo",
    KeySchema = new List<KeySchemaElement>
    {
        new KeySchemaElement("Id", KeyType.HASH),
    },
    AttributeDefinitions = new List<AttributeDefinition>
    {
        new AttributeDefinition("Id", ScalarAttributeType.N),
    },
    ProvisionedThroughput = new ProvisionedThroughput
    {
        ReadCapacityUnits = 10,
        WriteCapacityUnits = 5,
    }
};
awsDb.CreateTable(request);
```

DynamoDB Tables take a little while to create in AWS so we can't use it immediately, instead you'll need to 
periodically poll to check the status for when it's ready:

```csharp
//AWSSDK
var startAt = DateTime.UtcNow;
var timeout = TimeSpan.FromSeconds(60);
do
{
    try
    {
        var descResponse = awsDb.DescribeTable("Todo");
        if (descResponse.Table.TableStatus == DynamoStatus.Active)
            break;

        Thread.Sleep(TimeSpan.FromSeconds(2));
    }
    catch (ResourceNotFoundException)
    {
        // DescribeTable is eventually consistent. So you might get resource not found.
    }

    if (DateTime.UtcNow - startAt > timeout)
        throw new TimeoutException("Exceeded timeout of {0}".Fmt(timeout));

} while (true);
```

Once the table is Active we can start using it, to get the list of table names we send a `ListTablesRequest`:

```csharp
//AWSSDK
var listResponse = awsDb.ListTables(new ListTablesRequest());
var tableNames = listResponse.TableNames;
tableNames.PrintDump();
```

## Managed DynamoDB Client

As we can see using the `AmazonDynamoDBClient` directly requires a lot more imperative code, but it also ends up doing 
a lot less. We've not included the logic to query existing tables so only the missing tables are created, we've not 
implemented any error handling or Retry logic (important for Cloud Services) and we're not checking to make sure we've 
collected the entire list of results (implementing paging when necessary).

Whereas every request in PocoDynamo is invoked inside a managed execution where any temporary errors are retried using the 
[AWS recommended retries exponential backoff](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/ErrorHandling.html#APIRetries).

All PocoDynamo API's returning `IEnumerable<T>` returns a lazy evaluated stream which behind-the-scenes sends multiple
paged requests as needed whilst the sequence is being iterated. As LINQ API's are also lazily evaluated you could use 
`Take()` to only download however the exact number results you need. So you can query the first 100 table names with:

```csharp
//PocoDynamo
var first100TableNames = db.GetTableNames().Take(100).ToList();
```

and PocoDynamo will only make the minimum number of requests required to fetch the first 100 results.

## AutoIncrement Primary Keys

Once the `Todo` table is created we can start adding TODOs to it. If we were using OrmLite. the `[AutoIncrement]` 
attribute lets us use the RDBMS's native support for auto incrementing sequences to populate the Id primary key. 
Unfortunately DynamoDB lacks an auto increment feature and instead recommends the user to supply a unique key as shown
in their [DynamoDB Forum example](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DataModel.html)
where they've chosen a Forum Name as the Hash Key of the Forum and Thread tables, whilst the Reply comment uses a
concatenation of `ForumName` + `#` + `ThreadSubject` as its Hash Key and the `ReplyDateTime` for the Range Key. 

However auto incrementing Ids have a number of useful properties making it ideal for identifying data:

  - **Unique** - Each new item is guaranteed to have a unique Id that's higher than all Ids before it
  - **Sequential** - A useful property to ensure consistent results when paging or ordering
  - **Never change** - To ensure a constant key that never changes, Ids shouldn't contain data it references
  - **Easy to read** - Humans have a better chance to read and remember a number than a concatenated string
  - **Easy to reference** - It's easier to reference a predictable numeric field than a concatenated string

They're also more re-usable as most data stores have native support for integer primary keys. For these reasons we've
added support for Auto-Incrementing integer primary keys in PocoDynamo where Ids annotated with `[AutoIncrement]` 
attribute are automatically populated with the next id in its sequence.

#### ISequenceSource

The Auto Incrementing functionality is provided by the
[ISequenceSource](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.Interfaces/ISequenceSource.cs) 
interface:

```csharp
public interface ISequenceSource : IRequiresSchema
{
    long Increment(string key, int amount = 1);
    void Reset(string key, int startingAt = 0);
}
```
#### DynamoDbSequenceGenerator

The default implementation uses 
[DynamoDbSequenceGenerator](https://github.com/ServiceStack/ServiceStack.Aws/blob/master/src/ServiceStack.Aws/DynamoDb/DynamoDbSequenceGenerator.cs)
which stores sequences for each table in the `Seq` DynamoDB Table so no additional services are required. To ensure
unique incrementing sequences in DynamoDB, PocoDynamo uses UpdateItemRequest's `AttributeValueUpdate` feature to perform
atomic value updates. PocoDynamo sequences are also very efficient and only require a single DynamoDB call to populate a 
batch of Primary Key Ids which are also guaranteed to be in order (and without gaps) for batches that are stored together. 

#### RedisSequenceSource

If preferred you can instead instruct PocoDynamo to maintain sequences in Redis using 
[RedisSequenceSource](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.Server/RedisSequenceSource.cs)
or alternatively inject your own implementation which can be configured in PocoDynamo with:

```csharp
var db = new PocoDynamo(awsDb) {
    Sequences = new RedisSequenceSource(redisManager),
};
```

## Putting items with PocoDynamo

As we can take advantage of Auto Incrementing Id's, storing Items becomes as simple as creating a number of POCO's and
calling PutItems:

```csharp
//PocoDynamo
var todos = 100.Times(i => new Todo { Content = "TODO " + i, Order = i });
db.PutItems(todos);
```

## Putting items with AmazonDynamoDBClient

To do this manually with `AmazonDynamoDBClient` you'd need to create and `UpdateItemRequest` to update the counter
maintaining your TODO sequences:

```csharp
//AWSSDK
var incrRequest = new UpdateItemRequest
{
    TableName = "Seq",
    Key = new Dictionary<string, AttributeValue> {
        {"Id", new AttributeValue { S = "Todo" } }
    },
    AttributeUpdates = new Dictionary<string, AttributeValueUpdate> {
        {
            "Counter",
            new AttributeValueUpdate {
                Action = AttributeAction.ADD,
                Value = new AttributeValue { N = "100" }
            }
        }
    },
    ReturnValues = ReturnValue.ALL_NEW,
};

var response = awsDb.UpdateItem(incrRequest);
var nextSequences = Convert.ToInt64(response.Attributes["Counter"].N);
```

After you know which sequence to start with you can start putting items using a Dictionary of Attribute Values:

```csharp
//AWSSDK
for (int i = 0; i < 100; i++)
{
    var putRequest = new PutItemRequest("Todo",
        new Dictionary<string, AttributeValue> {
            { "Id", new AttributeValue { N = (nextSequences - 100 + i).ToString() } },
            { "Content", new AttributeValue("TODO " + i) },
            { "Order", new AttributeValue { N = i.ToString() } },
            { "Done", new AttributeValue { BOOL = false } },
        });

    awsDb.PutItem(putRequest);
}
```

Although even without the managed execution this still isn't equivalent to PocoDynamo's example above as to store 
multiple items efficiently PocoDynamo `PutItems()` API batches multiple Items in 4x `BatchWriteItemRequest` 
behind-the-scenes, the minimum number needed due to DynamoDB's maximum Write Batch size limit of 25 requests.

## Getting Items with PocoDynamo

Getting an item just requires the Generic Type and the primary key of the item to fetch:

```csharp
var todo = db.GetItem<Todo>(1);
todo.PrintDump();
```

Which returns the Todo item if it exists, or `null` if it doesn't.

Fetching all table items is where an understanding of DynamoDB's architecture and its limits become important. DynamoDB 
achieves its scalability by partitioning your data across multiple partitions based on its hash Key (aka Primary Key). 
This means that the only way to efficiently query across data containing multiple primary keys is to either explicitly 
create a [Global Secondary Index](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GSI.html) or perform a 
full-table Scan. 

However table scans in DynamoDB are more inefficient than full table scans in RDBMS's since it has to scan across
multiple partitions which can quickly use up your table's provisioned throughput, as such scans should be limited 
to low usage areas. 

With that said, you can do Table Scans in PocoDynamo using API's starting with `Scan*` prefix, e.g. to return
all Todo items:

```csharp
//PocoDynamo
IEnumerable<Todo> todos = db.ScanAll<Todo>();
```

As IEnumerable's are lazily executed, it only starts sending `ScanRequest` to fetch all Items once the IEnumerable is 
iterated, which it does in **batches of 1000** (configurable with `PocoDynamo.PagingLimit`). 

To fetch all items you can just call `ToList()`:

```csharp
var allTodos = todos.ToList();
allTodos.PrintDump();
```

Which incidentally is also just what `db.GetAll<Todo>()` does. 

## Getting Items with AWSSDK

To fetch the same single item with the AWSSDK client you'd construct and send a `GetItemRequest`, e.g:

```csharp
//AWSSDK
var request = new GetItemRequest
{
    TableName = "Todo",
    Key = new Dictionary<string, AttributeValue> {
        { "Id", new AttributeValue { N = "1"} }
    },
    ConsistentRead = true,
};

var response = awsDb.GetItem(request);
var todo = new Todo
{
    Id = Convert.ToInt64(response.Item["Id"].N),
    Content = response.Item["Content"].S,
    Order = Convert.ToInt32(response.Item["Order"].N),
    Done = response.Item["Done"].BOOL,
};
```

Although this is a little fragile as it doesn't handle the case when attributes (aka Properties) or the item doesn't exist.

Doing a full-table scan is pretty straight-forward although as you're scanning the entire table you'll want to implement
the paging to scan through all items, which looks like:

```csharp
//AWSSDK
var request = new ScanRequest
{
    TableName = "Todo",
    Limit = 1000,
};

var allTodos = new List<Todo>();
ScanResponse response = null;
do
{
    if (response != null)
        request.ExclusiveStartKey = response.LastEvaluatedKey;

    response = awsDb.Scan(request);

    foreach (var item in response.Items)
    {
        var todo = new Todo
        {
            Id = Convert.ToInt64(item["Id"].N),
            Content = item["Content"].S,
            Order = Convert.ToInt32(item["Order"].N),
            Done = item["Done"].BOOL,
        };
        allTodos.Add(todo);
    }
    
} while (response.LastEvaluatedKey != null && response.LastEvaluatedKey.Count > 0);

allTodos.PrintDump();
```

## Deleting an Item with PocoDynamo

Deleting an item is similar to getting an item which just needs the generic type and primary key:

```csharp
//PocoDynamo
db.DeleteItem<Todo>(1);
```

## Deleting an Item with AWSSDK

Which just sends a `DeleteItemRequest` to delete the Item:

```csharp
//AWSSDK
var request = new DeleteItemRequest
{
    TableName = "Todo",
    Key = new Dictionary<string, AttributeValue> {
        { "Id", new AttributeValue { N = "1"} }
    },
};

awsDb.DeleteItem(request);
```

## Querying

The simple Todo example should give you a feel for using PocoDynamo to handle basic CRUD operations. 
Another area where PocoDynamo adds a lot of value which can be fairly cumbersome to do without, is in creating
[Query and Scan](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/QueryAndScan.html) 
requests to query data in DynamoDB Tables.

### QueryExpressions are QueryRequests

The query functionality in PocoDynamo is available on the `QueryExpression<T>` class which is used as a typed query 
builder to construct your Query request. An important attribute about QueryExpression's are that they inherit AWSSDK's 
[QueryRequest](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/LowLevelDotNetQuerying.html) Request DTO. 

This provides a number of benefits, they're easy to use and highly introspectable since each API just populates 
different fields in the Request DTO. They're also highly reusable as QueryExpressions can be executed as-is in AWSSDK 
DynamoDB client and vice-versa with PocoDynamo's `Query` API's executing both `QueryExpression<T>` and `QueryRequest` 
DTOs. The difference with PocoDynamo's Query API is that they provide managed exeuction, lazy evaluation, paged queries 
and auto-conversion of dynamic results into typed POCOs.

### Query Usage

Query's are the efficient way to Query DynamoDB since it's limited to querying indexed fields, i.e. the Hash and 
Range Keys on your Tables or Table Indexes. Although it has the major limitation that it always needs to specify a Hash 
condition, essentially forcing the query to be scoped to a single partition. This makes it especially useless for Tables
with only a single Hash Primary Key like `Todo` as the query condition will always limit to a maximum of 1 result.

Nevertheless we can still use it to show how to perform server-side queries with PocoDynamo. To create a 
QueryExpression use the `FromQuery*` API's. It accepts a `KeyConditionExpression` as the first argument as it's a 
mandatory requirement for Query Requests which is used to identify the partition the query should be executed on:

```csharp
var q = db.FromQuery<Todo>(x => x.Id == 1);
```

PocoDynamo parses this lambda expression to return a populated `QueryExpression<Todo>` which you can inspect to find 
the `TableName` set to **Todo** and the `KeyConditionExpression` set to **(Id = :k0)** with the 
`ExpressionAttributeValues` Dictionary containing a Numeric value of **1** for the key **:k0**.

From here you can continue populating the QueryRequest DTO by calling the QueryExpression methods the names of which
are modeled after the properties they populate, e.g. the `Filter()` API populates the `FilterExpression` property:

```csharp
q.Filter(x => x.Done);
```

After you've finished populating the Request DTO you can call PocoDynamo's `Query()` API to execute the query. 
This returns a lazily executed resultset which you can use LINQ methods on to fetch the results. Given the primary key
condition we know this will only return 0 or 1 rows based on whether or not the TODO has been completed which we can
check with by calling LINQ's `FirstOrDefault()` method:

```csharp
var todo1 = db.Query(q).FirstOrDefault();
```

If `todo1` was completed it will return the populated `Todo`, otherwise it will return `null`.

#### Expression Chaining

Most `QueryExpression` methods returns itself and an alternative to calling `Query` on PocoDynamo (or AWSSDK) to execute 
the Query, you can instead call the `Exec()` alias. This allows you to create and execute your DynamoDb Query in a 
single expression which could instead be rewritten as:

```csharp
var todo1 = db.FromQuery<Todo>(x => x.Id == 1)
    .Filter(x => x.Done)
    .Exec()
    .FirstOrDefault();
```

### Scan Operations


## Supported LINQ Expressions

Another area where PocoDynamo adds a lot of value that can be fairly cumbersome to do without, is in creating and 
executing [Query and Scan](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/QueryAndScan.html) 
requests to query data in DynamoDB Tables.

### QueryExpressions are QueryRequests

The query functionality in PocoDynamo is available on the `QueryExpression<T>` class which can be used as a typed query 
builder to construct your Query request. An important attribute of QueryExpression's is that they simply inherit AWSSDK's 
[QueryRequest](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/LowLevelDotNetQuerying.html) Request DTO. 

This provides a number of benefits, they're easy to understand and highly introspectable as each API just ends up 
populating fields in the base `QueryRequest` DTO. They're highly reusable where QueryExpressions can be executed as-is 
in both AWSSDK's DynamoDB client as well as PocoDynamo's `Query*` API's which can execute both `QueryExpression<T>` and 
and AWSSDK's `QueryRequest` DTOs. The difference when executing queries in PocoDynamo are that they provide managed 
execution, lazily evaluated streaming results, paged queries and auto-conversion of unstructured results into typed POCOs.

### Query Usage

[DynamoDB Query's](http://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_Query.html) enable efficient 
querying of data in DynamoDB as it's limited to querying the indexed Hash and Range Keys on your Tables or Table Indexes. 
Although it has the major limitation that it always needs to specify a Hash condition, essentially forcing the query 
to be scoped to a single partition. This makes it fairly useless for Tables with only a single Hash Primary Key like 
`Todo` as the query condition will always limit to a maximum of 1 result.

Nevertheless we can still use it to show how to perform server-side queries with PocoDynamo. To create a 
QueryExpression use the `FromQuery*` API's. It accepts a `KeyConditionExpression` as the first argument given it's a 
mandatory requirement for Query Requests which uses it to identify the partition the query should be executed on:

```csharp
var q = db.FromQuery<Todo>(x => x.Id == 1);
```

#### Key Condition and Filter Expressions

PocoDynamo parses this lambda expression to return a populated `QueryExpression<Todo>` which you can inspect to find 
the `TableName` set to **Todo** and the `KeyConditionExpression` set to **(Id = :k0)** with the 
`ExpressionAttributeValues` Dictionary containing a Numeric value of **1** for the key **:k0**.

From here you can continue constructing the QueryRequest DTO by populating its properties directly or by calling 
`QueryExpression` high-level methods (modeled after the properties they populate), e.g. the `KeyCondition()` method
populates the `KeyConditionExpression` property, `Filter()` populates the `FilterExpression` property and any arguments 
used in any expression are automatically parameterized and added to the `ExpressionAttributeValues` collection:

```csharp
var q = db.FromQuery<Todo>()
    .KeyCondition(x => x.Id == 1) //Equivalent to: db.FromQuery<Todo>(x => x.Id == 1)
    .Filter(x => x.Done);

q.TableName                 // Todo
q.KeyConditionExpression    // (Id = :k0)
q.FilterExpression          // Done = :true
q.ExpressionAttributeValues // :k0 = AttributeValue {N=1}, :true = AttributeValue {BOOL=true}
```

Filter expressions are applied after the query is executed which enable more flexible querying as they're not just 
limited to key fields and can be used to query any field to further filter the returned resultset.

### Executing Queries

After you've finished populating the Request DTO it can be executed with PocoDynamo's `Query()`. This returns a lazily 
evaluated resultset which you can use LINQ methods on to fetch the results. Given the primary key condition we know this 
will only return 0 or 1 rows based on whether or not the TODO has been completed which we can check with by calling 
LINQ's `FirstOrDefault()` method:

```csharp
var todo1Done = db.Query(q).FirstOrDefault();
```

Where `todo1Done` will hold the populated `Todo` if it was marked done, otherwise it will be `null`.

#### Expression Chaining

Most `QueryExpression` methods returns itself and an alternative to calling `Query` on PocoDynamo (or AWSSDK) to execute 
the Query, you can instead call the `Exec()` alias. This allows you to create and execute your DynamoDb Query in a 
single expression which can instead be rewritten as:

```csharp
var todo1Done = db.FromQuery<Todo>(x => x.Id == 1)
    .Filter(x => x.Done)
    .Exec()
    .FirstOrDefault();
```

### Related Items

DynamoDB Queries are ideally suited for when the dataset is naturally isolated, e.g. multi-tenant Apps that are 
centered around Customer data so any related records are able to share the same `CustomerId` Hash Key. 

PocoDynamo has good support for maintaining related data which can re-use the same Data Annotations used to define 
POCO relationships in OrmLite, often letting you reuse your existing OrmLite RDBMS data models in DynamoDB as well.

To illustrate how to use PocoDynamo to maintain related data we'll walk through a typical Customer and Orders example:

```csharp
public class Customer
{
    [AutoIncrement]
    public int Id { get; set; }
    public string Name { get; set; }
    public CustomerAddress PrimaryAddress { get; set; }
}

public class CustomerAddress
{
    [AutoIncrement]
    public int Id { get; set; }
    public string Address { get; set; }
    public string State { get; set; }
    public string Country { get; set; }
}

[Alias("CustomerOrder")]
public class Order
{
    [AutoIncrement]
    public int Id { get; set; }

    [References(typeof(Customer))]
    public int CustomerId { get; set; }

    public string Product { get; set; }
    public int Qty { get; set; }

    [Index]
    public virtual decimal Cost { get; set; }
}
```

In order to use them we need to tell PocoDynamo which of the Types are Tables that it should create in DynamoDB which
we can do by registering them then with PocoDynamo then calling `InitSchema()` which will go through and create any
of the tables that don't yet exist in DynamoDB: 

```csharp
db = new PocoDynamo(awsDb)
    .RegisterTable<Customer>()
    .RegisterTable<Order>();

db.InitSchema();
```

`InitSchema()` will also wait until the tables have been created so they're immediately accessible afterwards. 
As creating DynamoDB tables can take upwards of a minute in AWS you can use the 
[alternative Async APIs](https://github.com/ServiceStack/ServiceStack.Aws/blob/master/src/ServiceStack.Aws/DynamoDb/IPocoDynamoAsync.cs)
if you wanted to continue to doing other stuff whilst the tables are being created in AWS, e.g:

```csharp
var task = db.InitSchemaAsync();

// do other stuff...

await task;
```

## Related Data

After the tables are created we can insert the top-level Customer record as normal:

```csharp
var customer = new Customer
{
    Name = "Customer",
    PrimaryAddress = new CustomerAddress
    {
        Address = "1 road",
        State = "NT",
        Country = "Australia",
    }
};

db.PutItem(customer);
```

Before adding the record, PocoDynamo also populates any `[AutoIncrement]` properties with the next number in the 
sequence for that Type. Any complex types stored on the `Customer` POCO like `CustomerAddress` gets persisted along
with the containing `Customer` entry and converted into a **Map** of DynamoDB Attribute Value pairs. We can view the 
[DynamoDB Web Console](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/ConsoleDynamoDB.html) 
to see how this is stored in DynamoDB:

![](https://raw.githubusercontent.com/ServiceStack/Assets/master/img/aws/pocodynamo/related-customer.png)

### Related Tables

You can define a related table using the `[References]` attribute to tell PocoDynamo what the parent table is, e.g:

```csharp
[Alias("CustomerOrder")]
public class Order
{
    [AutoIncrement]
    public int Id { get; set; }
    
    [References(typeof(Customer))]
    public int CustomerId { get; set; }   
    //...
}
```

Which PocoDynamo infers to create the table using the parent's `CustomerId` as its Hash Key, relegating its `Id` as
the Range Key for the table. This ensures the Order is kept in the same partition as all other related Customer Data, 
necessary in order to efficiently query a Customer's Orders. When both the Hash and Range Key are defined they're treated 
as the Composite Key for that table which needs to be unique for each item - guaranteed when using `[AutoIncrement]` Id's.

#### Inserting Related Data

After the table is created we can generate and insert random orders like any other table, e.g:

```csharp
var orders = 10.Times(i => new Order
{
    CustomerId = customer.Id,
    Product = "Item " + (i % 2 == 0 ? "A" : "B"),
    Qty = i + 2,
    Cost = (i + 2) * 2
});

db.PutItems(orders);
```

You can also use the alternative `PutRelatedItems()` API and get PocoDynamo to take care of populating the `CustomerId`:

```csharp
var orders = 10.Times(i => new Order
{
    Product = "Item " + (i % 2 == 0 ? "A" : "B"),
    Qty = i + 2,
    Cost = (i + 2) * 2
});

db.PutRelatedItems(customer.Id, orders);
```

Both examples results in the same data being inserted into the **CustomerOrder** DynamoDB table:

![](https://raw.githubusercontent.com/ServiceStack/Assets/master/img/aws/pocodynamo/related-customer-orders.png)

This also shows how the `[Alias]` attribute can be used to rename the `Order` Type as **CustomerOrder** in DynamoDB.

### Querying Related Tables

Now we have related data we can start querying it, something you may want to do is fetch all Customer Orders:

```csharp
var q = db.FromQuery<Order>(x => x.CustomerId == customer.Id);
var dbOrders = db.Query(q);
```

As getting related Items for a Hash Key is a popular query, it has an explicit API:

```csharp
var dbOrders = db.GetRelatedItems<Order>(customer.Id);
```

We can refine the query further by specifying a `FilterExpression` to limit the results DynamoDB returns:

```csharp
var expensiveOrders = q.Clone()
    .Filter(x => x.Cost > 10)
    .Exec();
```

> Using `Clone()` will create and modify a copy of the query, leaving the original one intact.

### [Local Secondary Indexes](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/LSI.html)

But filters aren't performed on an Index and can be inefficient if your table has millions of customer rows. By default 
only the Hash and Range Key are indexed, in order to efficiently query any other field you will need to create
a [Local Secondary Index](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/LSI.html) for it.

This is easily done in PocoDynamo by annotating the properties you want indexed with the `[Index]` attribute:

```csharp
public class Order
{
    //...    
    [Index]
    public decimal Cost { get; set; }
}
```

Which tells PocoDynamo to create a Local Secondary Index for the `Cost` property when it creates the table.

When one exists, you can query a Local Index with `LocalIndex()`:

```csharp
var expensiveOrders = q
    .LocalIndex(x => x.Cost > 10)
    .Exec();    
```

Which now performs the Cost query on an index. Although this only returns a partially populated Order, specifically
with just the Hash Key (CustomerId), Range Key (Id) and the field that's indexed (Cost):

    expensiveOrders.PrintDump();
    [
        {
            Id: 5,
            CustomerId: 1,
            Qty: 0,
            Cost: 12
        },
        //...
    ]

This is due to [Local Secondary Indexes](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/LSI.html)
being just denormalized tables behind the scenes which by default only returns re-projected fields that were defined
when the Index was created. 

One way to return populated orders is to specify a custom `ProjectionExpression` with the fields you want returned.
E.g. You can create a request with a populated `ProjectionExpression` that returns all Order fields with:

```csharp
var expensiveOrders = q
    .LocalIndex(x => x.Cost > 10)
    .Select<Order>()              //Equivalent to: SelectTableFields()
    .Exec();
```

Which now returns:

    expensiveOrders.PrintDump();
    [
        {
            Id: 5,
            CustomerId: 1,
            Product: Item A,
            Qty: 6,
            Cost: 12
        },
        //...
    ]

### Typed Local Indexes

Using a custom `ProjectionExpression` is an easy work-around, although for it to work DynamoDB needs to consult the
primary table to fetch the missing fields for each item. For large tables that are frequently accessed, the query
can be made more efficient by projecting the fields you want returned when the Index is created. 

You can can tell PocoDynamo which additional fields it should reproject by creating a **Typed Local Index** which is 
just a POCO implementing `ILocalIndex<T>` containing all the fields the index should contain, e.g:

```csharp
public class OrderCostLocalIndex : ILocalIndex<Order>
{
    [Index]
    public decimal Cost { get; set; }
    public int CustomerId { get; set; }

    public int Id { get; set; }
    public int Qty { get; set; }
}

[References(typeof(OrderCostLocalIndex))]
public class Order { ... }
```

Then use the `[References]` attribute to register the Typed Index so PocoDynamo knows which additional indexes needs 
to be created with the table. The `[Index]` attribute is used to specify which field is indexed (Range Key) whilst 
the `CustomerId` is automatically used the Hash Key for the Local Index Table.

#### Querying Typed Indexes

To query a typed Index, use `FromQueryIndex<T>()` which returns a populated Query Request with the Table and Index Name.
As `Cost` is now the Range Key of the Local Index table it can be queried together with the `CustomerId` Hash Key in 
the Key Condition expression:

```csharp
List<OrderCostLocalIndex> expensiveOrderIndexes = db.FromQueryIndex<OrderCostLocalIndex>(x => 
        x.CustomerId == customer.Id && x.Cost > 10)
    .Exec();
```

This return a list of populated indexes that now includes the `Qty` field:

    expensiveOrderIndexes.PrintDump();
    [
        {
            Cost: 12,
            CustomerId: 1,
            Id: 5,
            Qty: 6
        },
        //...
    ]

If preferred you can easily convert Typed Index into Orders by using ServiceStack's 
[built-in Auto-Mapping](https://github.com/ServiceStack/ServiceStack/wiki/Auto-mapping), e.g:

```csharp
List<Order> expensiveOrders = expensiveOrderIndexes
    .Map(x => x.ConvertTo<Order>());
```

### [Global Secondary Indexes](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GSI.html)

The major limitation of Local Indexes is that they're limited to querying data in the same partition (Hash Key). 
To efficiently query an index spanning the entire dataset, you need to instead use a 
[Global Secondary Index](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GSI.html).

Support for Global Indexes in PocoDynamo is similar to Typed Local Indexes, but instead implements `IGlobalIndex<T>`.
They also free you to choose a new Hash Key, letting you create an Index spanning all Customers. 

For example we can create a global index that lets us search the cost across all orders containing a particular product:

```csharp
public class OrderCostGlobalIndex : IGlobalIndex<Order>
{
    [HashKey]
    public string Product { get; set; }
    [Index]
    public decimal Cost { get; set; }

    public int CustomerId { get; set; }
    public int Qty { get; set; }
    public int Id { get; set; }
}

[References(typeof(OrderCostGlobalIndex))]
public class Order { ... }
```

Our Key Condition can now instead query Product and Cost fields across all Customer Orders:

```csharp
var expensiveItemAOrders = db.FromQueryIndex<OrderCostGlobalIndex>(x => 
        x.Product == "Item A" && x.Cost > 10)
    .Exec();
```

Which will print all **Item A** Orders with a **Cost > 10**:

    expensiveItemAOrders.PrintDump();
    [
        {
            Product: Item A,
            Cost: 12,
            CustomerId: 1,
            Qty: 6,
            Id: 5
        },
        //...
    ]
    
## Scan Requests

You'll want to just use queries for any frequently accessed code running in production, although the full querying 
flexibility available in full table scan requests can be useful for ad hoc querying and to speed up development cycles
by initially starting with Scan queries then when the data requirements for your App's have been finalized, rewrite 
them to use indexes and queries.

To create Scan Requests you instead call the `FromScan*` API's, e.g:

```csharp
var allOrders = db.ScanAll<Order>();

var expensiveOrders = db.FromScan<Order>(x => x.Cost > 10)
    .Exec();
```

You can also perform scans on Global Indexes, but unlike queries they don't need to be limited to the Hash Key:

```csharp
var expensiveOrderIndexes = db
    .FromScanIndex<OrderCostGlobalIndex>(x => x.Cost > 10)
    .Exec();
```

Just like `QueryExpression<T>` the populated `ScanExpression<T>` inherits from AWSSDK's `ScanRequest` enabling the same
re-use benefits for `ScanRequest` as they do for QueryRequest's.

## Query and Scan Expressions

Both Scans and Query expressions benefit from a Typed LINQ-like expression API which can be used to populate the DTO's

 - **KeyConditionExpression** - for specifying conditions on tables Hash and Range keys (only: QueryRequest)
 - **FilterExpression** - for specifying conditions to filter results on other fields
 - **ProjectionExpression** - to specify any custom fields (default: all fields)

Each `QueryRequest` needs to provide a key condition which can be done when creating the QueryExpression:

```csharp
var orders = db.FromQuery<Order>(x => x.CustomerId == 1).Exec();

// Alternative explicit API
var expensiveOrders = db.FromQuery<Order>().KeyCondition(x => x.CustomerId == 1).Exec();
```

Whilst every condition on a `ScanRequest` is added to the FilterExpression: 

```csharp
var expensiveOrders = db.FromScan<Order>(x => x.Cost > 10).Exec();

// Alternative explicit API
var expensiveOrders = db.FromScan<Order>().Filter(x => x.Cost > 10).Exec();
```

Calling `Exec()` returns a lazily executed response which transparently sends multiple paged requests to fetch the 
results as needed, e.g calling LINQ's `.FirstOrDefault()` only makes a single request whilst `.ToList()` fetches the 
entire resultset. All streaming `IEnumerable<T>` requests are sent with the configured `PagingLimit` (default: 1000).

#### Custom Limits

Several of PocoDynamo API's have overloads that let you specify a custom limit. API's with limits are instead 
executed immediately with the limit specified and returned in a concrete List:

```csharp
List<Order> expensiveOrders = db.FromScan<Order>().Filter(x => x.Cost > 10).Exec(limit:5);
```

### Custom Filter Expressions

There are also custom overloads that can be used to execute a custom expression when more flexibility is needed:

```csharp
// Querying by Custom Filter Condition with anon args
var expensiveOrders = db.FromScan<Order>().Filter("Cost > :amount", new { amount = 10 }).Exec();

// Querying by Custom Filter Condition with loose-typed Dictionary
var expensiveOrders = db.FromScan<Order>().Filter("Cost > :amount", 
        new Dictionary<string, object> { { "amount", 10 } })
    .Exec();
```

### Custom Select Projections

By default queries return all fields defined on the POCO model. You can also customize the projected fields that are 
returned with the `Select*` and `Exec*` APIs:

```csharp
// Return partial fields from anon object
var partialOrders = db.FromScan<Order>().Select(x => new { x.CustomerId, x.Cost }).Exec();

// Return partial fields from array
var partialOrders = db.FromScan<Order>().Select(x => new[] { "CustomerId", "Cost" }).Exec();

// Return partial fields defined in a custom Poco
class CustomerCost
{
    public int CustomerId { get; set; }
    public virtual decimal Cost { get; set; }
}

var custCosts = db.FromScan<Order>().Select<CustomerCost>()
    .Exec()
    .Map(x => x.ConvertTo<CustomerCost>());

// Alternative shorter version of above
var custCosts = db.FromScan<Order>().ExecInto<CustomerCost>().ToList();

// Useful when querying and index and returing results in primary Order Poco 
List<Order> expensiveOrders = db.FromScanIndex<OrderCostGlobalIndex>(x => x.Cost > 10)
    .ExecInto<Order>();

// Return a single column of fields
List<int> orderIds = db.FromScan<Order>().ExecColumn(x => x.Id).ToList();
```

### Advanced LINQ Expressions

In addition to basic predicate conditions, DynamoDB also includes support for 
[additional built-in functions](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Expressions.SpecifyingConditions.html)
which PocoDynamo also provides typed LINQ support for:

#### begins_with

Return items where string fields starts with a particular substring:

```csharp
var orders = db.FromScan<Order>(x => x.Product.StartsWith("Item A")).Exec();

// Equivalent to
var orders = db.FromScan<Order>(x => Dynamo.BeginsWith(x.Product, "Item A")).Exec();

var orders = db.FromScan<Order>().Filter("begins_with(Product, :s)", new { s = "Item A" }).Exec();
```

#### contains

Return items where string fields contains a particular substring:

```csharp
var orders = db.FromScan<Order>(x => x.Product.Contains("em A")).Exec();

// Equivalent to
var orders = db.FromScan<Order>(x => Dynamo.Contains(x.Product, "em A")).Exec();

var orders = db.FromScan<Order>().Filter("contains(Product, :s)", new { s = "em A" }).Exec();
```

#### in

Returns items where fields exist in a particular collection:

```csharp
var qtys = new[] { 5, 10 };

var orders = db.FromScan<Order>(x => qtys.Contains(x.Qty)).Exec();

// Equivalent to
var orders = db.FromScan<Order>(x => Dynamo.In(x.Qty, qtys)).Exec();

var orders = db.FromScan<Order>().Filter("Qty in(:q1,:q2)", new { q1 = 5, q2 = 10 }).Exec();
```

#### size

Returns items where the string length equals a particular size:

```csharp
var orders = db.FromScan<Order>(x => x.Product.Length == 6).Exec();

// Equivalent to
var orders = db.FromScan<Order>(x => Dynamo.Size(x.Product) == 6).Exec();

var orders = db.FromScan<Order>().Filter("size(Product) = :n", new { n = 6 }).Exec();
```

Size also works for querying the size of different native DynamoDB collections, e.g:

```csharp
public class IntCollections
{
    public int Id { get; set; }

    public int[] ArrayInts { get; set; }
    public HashSet<int> SetInts { get; set; }
    public List<int> ListInts { get; set; }
    public Dictionary<int, int> DictionaryInts { get; set; }
}

var results = db.FromScan<IntCollections>(x =>
        x.ArrayInts.Length == 10 &&
        x.SetInts.Count == 10 &&
        x.ListInts.Count == 10 &&
        x.DictionaryInts.Count == 10)
    .Exec();
```

#### between

Returns items where field values fall within a particular range (inclusive):

```csharp
var orders = db.FromScan<Order>(x => Dynamo.Between(x.Qty, 3, 5)).Exec();

// Equivalent to
var orders = db.FromScan<Order>(x => x.Qty >= 3 && x.Qty <= 5).Exec();

var orders = db.FromScan<Order>().Filter("Qty between :from and :to", new { from = 3, to = 5 }).Exec();
```

#### attribute_type

Return items where field is of a particular type:

```csharp
var orders = db.FromScan<Order>(x => 
        Dynamo.AttributeType(x.Qty, DynamoType.Number) &&
        Dynamo.AttributeType(x.Product, DynamoType.String))
    .Exec();

// Equivalent to
var orders = db.FromScan<Order>().Filter(
        "attribute_type(Qty, :n) and attribute_type(Product, :s)", new { n = "N", s = "S"})
    .Exec();
```

Valid Types: L (List), M (Map), S (String), SS (StringSet), N (Number), NS (NumberSet), B (Binary), BS, BOOL, NULL

#### attribute_exists

Return items where a particular field exists. As the schema of your data models evolve you can use this to determine
whether items are of an old or new schema:

```csharp
var newOrderTypes = db.FromScan<Order>(x => Dynamo.AttributeExists(x.NewlyAddedField)).Exec();

// Equivalent to
var newOrderTypes = db.FromScan<Order>().Filter("attribute_exists(NewlyAddedField)").Exec();
```

#### attribute_not_exists

Return items where a particular field does not exist:

```csharp
var oldOrderTypes = db.FromScan<Order>(x => Dynamo.AttributeNotExists(x.NewlyAddedField)).Exec();

// Equivalent to
var oldOrderTypes = db.FromScan<Order>().Filter("attribute_not_exists(NewlyAddedField)").Exec();
```

### Defaults and Custom Behavior

PocoDynamo is configured with the defaults below which it uses throughout its various API's when used in creating and 
querying tables:

```csharp
//Defaults:
var db = new PocoDynamo(awsDb) {
    PollTableStatus = TimeSpan.FromSeconds(2),
    MaxRetryOnExceptionTimeout = TimeSpan.FromSeconds(60),
    ReadCapacityUnits = 10,
    WriteCapacityUnits = 5,
    ConsistentRead = true,
    ScanIndexForward = true,
    PagingLimit = 1000,
};
```

If you wanted to query with different behavior you can create a clone of the client with the custom settings you want,
e.g. you can create a client that performs eventually consistent queries with:

```csharp
IPocoDynamo eventuallyConsistentDb = db.ClientWith(consistentRead:false);
```

## Table definition

To support different coding styles, readability/dependency preferences and levels of data model reuse, PocoDynamo 
enables a wide array of options for specifying a table's Hash and Range Keys, in the following order or precedence:

### Specifying a Hash Key

Using the AWSSDK's `[DynamoDBHashKey]` attribute:

```csharp
public class Table
{
    [DynamoDBHashKey]
    public int CustomId { get; set; }
}
```

This requires your models to have a dependency to the **AWSSDK.DynamoDBv2** NuGet package which can be avoided by using 
**ServiceStack.Interfaces** `[HashKey]` attribute instead which your models already likely have a reference to:

```csharp
public class Table
{
    [HashKey]
    public int CustomId { get; set; }
}
```

You can instead avoid any attributes using the explicit **HashKey** Naming convention:

```csharp
public class Table
{
    public int HashKey { get; set; }
}
```

For improved re-usability of your models you can instead use the generic annotations for defining a model's primary key:

```csharp
public class Table
{
    [PrimaryKey]
    public int CustomId { get; set; }
}
```

```csharp
public class Table
{
    [AutoIncrement]
    public int CustomId { get; set; }
}
```

Alternative using the Universal `Id` naming convention:

```csharp
public class Table
{
    public int Id { get; set; }
}
```

If preferred both Hash and Range Keys can be defined together with the class-level `[CompositeKey]` attribute:

```csharp
[CompositeKey("CustomHash", "CustomRange")]
public class Table
{
    public int CustomHash { get; set; }
    public int CustomRange { get; set; }
}
```

### Specifying a Range Key

For specifying the Range Key use can use the **AWSSDK.DynamoDBv2** Attribute:

```csharp
public class Table
{
    [DynamoDBRangeKey]
    public int CustomId { get; set; }
}
```

The **ServiceStack.Interfaces** attribute:

```csharp
public class Table
{
    [RangeKey]
    public int CustomId { get; set; }
}
```

Or without attributes, using the explicit `RangeKey` property name:

```csharp
public class Table
{
    public int RangeKey { get; set; }
}
```

## Examples

### [DynamoDbCacheClient](https://github.com/ServiceStack/ServiceStack.Aws/blob/master/src/ServiceStack.Aws/DynamoDb/DynamoDbCacheClient.cs)

We've been quick to benefit from the productivity advantages of PocoDynamo ourselves where we've used it to rewrite
[DynamoDbCacheClient](https://github.com/ServiceStack/ServiceStack.Aws/blob/master/src/ServiceStack.Aws/DynamoDb/DynamoDbCacheClient.cs)
which is now just 2/3 the size and much easier to maintain than the existing 
[Community-contributed version](https://github.com/ServiceStack/ServiceStack/blob/22aca105d39997a8ea4c9dc20b242f78e07f36e0/src/ServiceStack.Caching.AwsDynamoDb/DynamoDbCacheClient.cs)
whilst at the same time extending it with even more functionality where it now implements the `ICacheClientExtended` API.

### [DynamoDbAuthRepository](https://github.com/ServiceStack/ServiceStack.Aws/blob/master/src/ServiceStack.Aws/DynamoDb/DynamoDbAuthRepository.cs)

PocoDynamo's code-first Typed API made it much easier to implement value-added DynamoDB functionality like the new
[DynamoDbAuthRepository](https://github.com/ServiceStack/ServiceStack.Aws/blob/master/src/ServiceStack.Aws/DynamoDb/DynamoDbAuthRepository.cs)
which due sharing a similar code-first POCO approach to OrmLite, ended up being a straight-forward port of the existing
[OrmLiteAuthRepository](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.Server/Auth/OrmLiteAuthRepository.cs)
where it was able to reuse the existing `UserAuth` and `UserAuthDetails` data models.

### [DynamoDbTests](https://github.com/ServiceStack/ServiceStack.Aws/tree/master/tests/ServiceStack.Aws.DynamoDbTests)

Despite its young age we've added a comprehensive test suite behind PocoDynamo which has become our exclusive client
for developing DynamoDB-powered Apps.

### [AWS Apps](http://awsapps.servicestack.net/)

The [Live Demos](https://github.com/ServiceStackApps/LiveDemos) below were rewritten from their original RDBMS and OrmLite
backends to utilize a completely managed AWS Stack that now uses PocoDynamo and a DynamoDB-backend:

[![](https://raw.githubusercontent.com/ServiceStack/Assets/master/img/aws/pocodynamo/examples-razor-rockstars.png)](http://awsrazor.servicestack.net/)

[![](https://raw.githubusercontent.com/ServiceStack/Assets/master/img/aws/pocodynamo/examples-email-contacts.png)](http://awsapps.servicestack.net/emailcontacts/)

[![](https://raw.githubusercontent.com/ServiceStack/Assets/master/img/aws/pocodynamo/examples-todos.png)](http://awsapps.servicestack.net/todo/)

[![](https://raw.githubusercontent.com/ServiceStack/Assets/master/img/aws/pocodynamo/examples-awsauth.png)](http://awsapps.servicestack.net/awsauth/)

## IPocoClient API

```csharp
// Interface for the code-first PocoDynamo client
public interface IPocoDynamo : IPocoDynamoAsync, IRequiresSchema
{
    // Get the underlying AWS DynamoDB low-level client
    IAmazonDynamoDB DynamoDb { get; }

    // Get the numeric unique Sequence generator configured with this client
    ISequenceSource Sequences { get; }

    // Access the converters that converts POCO's into DynamoDB data types
    DynamoConverters Converters { get; }

    // How long should PocoDynamo keep retrying failed operations in an exponential backoff (default 60s)
    TimeSpan MaxRetryOnExceptionTimeout { get; }

    // Get the AWSSDK DocumentModel schema for this Table
    Table GetTableSchema(Type table);

    // Get PocoDynamo Table metadata for this table
    DynamoMetadataType GetTableMetadata(Type table);

    // Calls 'ListTables' to return all Table Names in DynamoDB
    IEnumerable<string> GetTableNames();

    // Creates any tables missing in DynamoDB from the Tables registered with PocoDynamo
    bool CreateMissingTables(IEnumerable<DynamoMetadataType> tables, TimeSpan? timeout = null);

    // Creates any tables missing from the specified list of tables
    bool CreateTables(IEnumerable<DynamoMetadataType> tables, TimeSpan? timeout = null);

    // Deletes all DynamoDB Tables
    bool DeleteAllTables(TimeSpan? timeout = null);

    // Deletes the tables in DynamoDB with the specified table names
    bool DeleteTables(IEnumerable<string> tableNames, TimeSpan? timeout = null);

    // Gets the POCO instance with the specified hash
    T GetItem<T>(object hash);

    // Gets the POCO instance with the specified hash and range value
    T GetItem<T>(object hash, object range);

    // Calls 'BatchGetItem' in the min number of batch requests to return POCOs with the specified hashes 
    List<T> GetItems<T>(IEnumerable<object> hashes);

    // Calls 'PutItem' to store instance in DynamoDB
    T PutItem<T>(T value, bool returnOld = false);

    // Calls 'BatchWriteItem' to efficiently store items in min number of batched requests
    void PutItems<T>(IEnumerable<T> items);

    // Deletes the instance at the specified hash
    T DeleteItem<T>(object hash, ReturnItem returnItem = ReturnItem.None);

    // Calls 'BatchWriteItem' to efficiently delete all items with the specified hashes
    void DeleteItems<T>(IEnumerable<object> hashes);

    // Calls 'BatchWriteItem' to efficiently delete all items with the specified hash and range pairs
    void DeleteItems<T>(IEnumerable<DynamoId> hashes);

    // Calls 'UpdateItem' with ADD AttributeUpdate to atomically increment specific field numeric value
    long Increment<T>(object hash, string fieldName, long amount = 1);

    // Polls 'DescribeTable' until all Tables have an ACTIVE TableStatus
    bool WaitForTablesToBeReady(IEnumerable<string> tableNames, TimeSpan? timeout = null);

    // Polls 'ListTables' until all specified tables have been deleted
    bool WaitForTablesToBeDeleted(IEnumerable<string> tableNames, TimeSpan? timeout = null);

    // Updates item Hash field with hash value then calls 'PutItem' to store the related instance
    void PutRelatedItem<T>(object hash, T item);

    // Updates all item Hash fields with hash value then calls 'PutItems' to store all related instances
    void PutRelatedItems<T>(object hash, IEnumerable<T> items);

    // Calls 'Query' to return all related Items containing the specified hash value
    IEnumerable<T> GetRelatedItems<T>(object hash);

    // Deletes all items with the specified hash and ranges
    void DeleteRelatedItems<T>(object hash, IEnumerable<object> ranges);


    // Calls 'Scan' to return lazy enumerated results that's transparently paged across multiple queries
    IEnumerable<T> ScanAll<T>();

    // Creates a Typed `ScanExpression` for the specified table
    ScanExpression<T> FromScan<T>(Expression<Func<T, bool>> filterExpression = null);

    // Creates a Typed `ScanExpression` for the specified Global Index
    ScanExpression<T> FromScanIndex<T>(Expression<Func<T, bool>> filterExpression = null);

    // Executes the `ScanExpression` returning the specified maximum limit of results
    List<T> Scan<T>(ScanExpression<T> request, int limit);

    // Executes the `ScanExpression` returning lazy results transparently paged across multiple queries
    IEnumerable<T> Scan<T>(ScanExpression<T> request);

    // Executes AWSSDK `ScanRequest` returning the specified maximum limit of results
    List<T> Scan<T>(ScanRequest request, int limit);

    // Executes AWSSDK `ScanRequest` returning lazy results transparently paged across multiple queries
    IEnumerable<T> Scan<T>(ScanRequest request);

    // Executes AWSSDK `ScanRequest` with a custom conversion function to map ScanResponse to results
    IEnumerable<T> Scan<T>(ScanRequest request, Func<ScanResponse, IEnumerable<T>> converter);


    // Return Live ItemCount using Table ScanRequest
    long ScanItemCount<T>();

    // Return cached ItemCount in summary DescribeTable
    long DescribeItemCount<T>();


    // Creates a Typed `QueryExpression` for the specified table
    QueryExpression<T> FromQuery<T>(Expression<Func<T, bool>> keyExpression = null);

    // Executes the `QueryExpression` returning lazy results transparently paged across multiple queries
    IEnumerable<T> Query<T>(QueryExpression<T> request);

    // Executes the `QueryExpression` returning the specified maximum limit of results
    List<T> Query<T>(QueryExpression<T> request, int limit);

    // Creates a Typed `QueryExpression` for the specified Local or Global Index
    QueryExpression<T> FromQueryIndex<T>(Expression<Func<T, bool>> keyExpression = null);

    // Executes AWSSDK `QueryRequest` returning the specified maximum limit of results
    List<T> Query<T>(QueryRequest request, int limit);

    // Executes AWSSDK `QueryRequest` returning lazy results transparently paged across multiple queries
    IEnumerable<T> Query<T>(QueryRequest request);

    // Executes AWSSDK `QueryRequest` with a custom conversion function to map QueryResponse to results
    IEnumerable<T> Query<T>(QueryRequest request, Func<QueryResponse, IEnumerable<T>> converter);


    // Create a clone of the PocoDynamo client with different default settings
    IPocoDynamo ClientWith(
        bool? consistentRead = null,
        long? readCapacityUnits = null,
        long? writeCapacityUnits = null,
        TimeSpan? pollTableStatus = null,
        TimeSpan? maxRetryOnExceptionTimeout = null,
        int? limit = null,
        bool? scanIndexForward = null);

    // Disposes the underlying IAmazonDynamoDB client
    void Close();
}

// Available API's with Async equivalents
public interface IPocoDynamoAsync
{
    Task CreateMissingTablesAsync(IEnumerable<DynamoMetadataType> tables, 
        CancellationToken token = default(CancellationToken));

    Task WaitForTablesToBeReadyAsync(IEnumerable<string> tableNames, 
        CancellationToken token = default(CancellationToken));

    Task InitSchemaAsync();
}
```

### PocoDynamo Extension helpers

To maintain a minimumal surface area for PocoDynamo, many additional API's used to provide a more DRY typed API's were moved into
[PocoDynamoExtensions](https://github.com/ServiceStack/ServiceStack.Aws/blob/master/src/ServiceStack.Aws/DynamoDb/PocoDynamoExtensions.cs)

```csharp
class PocoDynamoExtensions
{
    //Register Table
    DynamoMetadataType RegisterTable<T>();
    DynamoMetadataType RegisterTable(Type tableType);
    void RegisterTables(IEnumerable<Type> tableTypes);
    void AddValueConverter(Type type, IAttributeValueConverter valueConverter);

    //Get Table Metadata
    Table GetTableSchema<T>();
    DynamoMetadataType GetTableMetadata<T>();

    //Create Table
    bool CreateTableIfMissing<T>();
    bool CreateTableIfMissing(DynamoMetadataType table);
    bool CreateTable<T>(TimeSpan? timeout = null);

    bool DeleteTable<T>(TimeSpan? timeout = null);

    //Decrement API's
    long DecrementById<T>(object id, string fieldName, long amount = 1);
    long IncrementById<T>(object id, Expression<Func<T, object>> fieldExpr, long amount = 1);
    long DecrementById<T>(object id, Expression<Func<T, object>> fieldExpr, long amount = 1);

    List<T> GetAll<T>();
    T GetItem<T>(DynamoId id);

    //Typed API overloads for popular hash object ids
    List<T> GetItems<T>(IEnumerable<int> ids);
    List<T> GetItems<T>(IEnumerable<long> ids);
    List<T> GetItems<T>(IEnumerable<string> ids);

    void DeleteItems<T>(IEnumerable<int> ids);
    void DeleteItems<T>(IEnumerable<long> ids);
    void DeleteItems<T>(IEnumerable<string> ids);

    //Scan Helpers
    IEnumerable<T> ScanInto<T>(ScanExpression request);
    List<T> ScanInto<T>(ScanExpression request, int limit);

    //Query Helpers
    IEnumerable<T> QueryInto<T>(QueryExpression request);
    List<T> QueryInto<T>(QueryExpression request, int limit);
}
```
