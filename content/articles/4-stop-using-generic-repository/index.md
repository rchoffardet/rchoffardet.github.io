---
title: "The Generic Repository antipattern"
date: 2024-03-16T22:46:00
draft: false
summary: 
tags: 
 - C#
 - dotnet
 - design pattern
 - controversial
cover:
  image: banner.png
  hiddenInList: true
---

Before we enter this topic, here is a quick reminder of the Repository pattern (the real, non-generic, one), as found in the book [Patterns of Enterprise Application Architecture](https://martinfowler.com/books/eaa.html).
> A Repository mediates between the domain and data mapping layers using a collection-like interface for accessing domain objects.

It's a design pattern that abstracts the data access layer. It's used to both persist and retrieve data (or entities in DDD) without exposing storage implementation details. 
Considering the (simplified) entity `record Entity(Guid id, string attribute);` the pattern is expressed as an implementation of this interface:  
```csharp
interface EntityRepository
{
    void Create(Entity entity);
    List<Entity> GetAll();
    Entity GetById(Guid id);
    List<Entity> GetByAttribute(string attribute);
    // ...
}
```

This pattern was made to simplify things by hiding the complexity of retrieval and/or mapping of the returned entities. On the other hand and contrary to a popular belief amongst developers, when there is no complexity, there is no need for a repository.

The misuse of the repository pattern led to the emergence of an antipattern that defeats its original goal of simplification: The Generic Repository.  
It's a fairly common mistake made mostly by inexperienced but good-willing devs who, paradoxically, want to do things the proper way. 
I've made that mistake and I'm telling you this story so you won't make it or stop making it yourself 😉.

Let's start with an implementation of a repository that I could have made in the past, in a CRUD application, in front of an SQL database.
For the sake of the story, the following example illustrates a case when a repository is unnecessary.

Also, it's an oversimplified implementation to reduce verbosity to a maximum:
- `SqlClient` is an imaginary and convenient library
- asynchronism isn't handled
- Exceptions are not handled

```csharp
class SqlEntityRepository(SqlClient sqlClient)
{
    public void Create(Entity entity)
    {
        var statement = new SqlStatement("INSERT INTO entities (id, attribute) VALUES (@id, @attribute);")
        statement.SetParameter("id", entity.id);
        statement.SetParameter("attribute", entity.attribute);
        sqlClient.Execute(statement);
    }

    public List<Entity> GetAll()
    {
        var rows = sqlClient.Execute("SELECT * FROM entities;");
        return rows.Select(Map).ToList();
    }

    public Entity GetById(Guid id)
    {
        var rows = sqlClient.Execute("SELECT * FROM entities WHERE id = {0};", id);
        return Map(rows.Single());
    }

    public List<Entity> GetByAttribute(string attribute)
    {
        var rows = sqlClient.Execute("SELECT * FROM entities WHERE attribute = {0};", attribute);
        return rows.Select(Map).ToList();
    }

    private Entity Map(SqlRow row)
    {
        return new(
            row.Get<Guid>("id"), 
            row.Get<string>("attribute")
        );
    }
}
```

After some time, and as the application grew, I felt that I was repeating myself a lot and that I could mutualize code between my repositories. I _just_ needed to create a mapping from the entity to the table and its column. Then, I would be able to create an abstract parent class containing the common operations of my repository. 

```csharp
abstract class SqlGenericRepository<T>(SqlClient sqlClient)
{
    public void Create(T entity)
    {
        var mapping = Map(entity);
        var columnsName = string.join(", ", mapping.Keys);
        var parametersName = string.join(", ", mapping.Keys.Select(x => $"@{x}"));
        var statement = new SqlStatement($"INSERT INTO {TableName} ({columnsName}) VALUES ({valuesName});")

        foreach(var kvp in entity)
        {
            statement.SetParameter(kvp.Key, kvp.Value);
        }

        sqlClient.Execute(statement);
    }

    public List<T> GetAll()
    {
        var rows = sqlClient.Execute($"SELECT * FROM {TableName};");
        return rows.Select(Map).ToList();
    }

    public T GetById(Guid id)
    {
        var rows = sqlClient.Execute($"SELECT * FROM {TableName} WHERE {keyName} = {0};", id);
        return Map(rows.Single());
    }

    protected abstract T Map(SqlRow row);
    protected abstract Dictionary<string, object> Map(T entity);
    protected virtual string TableName { get; }
    protected virtual string keyName { get; }
}

class SqlEntityRepository(SqlClient sqlClient) : SqlGenericRepository<Entity>(sqlClient)
{
    public List<Entity> GetByAttribute(string attribute)
    {
        var rows = sqlClient.Execute("SELECT * FROM entities WHERE attribute = {attribute};", attribute);
        return rows.Select(Map).ToList();
    }

    protected override Entity Map(SqlRow row)
    {
        return new (
            row.Get<Guid>("id"), 
            row.Get<string>("attribute")
        );
    }
    
    protected override Dictionary<string, object> Map(Entity entity)
    {
        return new()
        {
            { "id", entity.id},
            { "attribute", entity.attribute}
        };
    }

    protected override string TableName => "entities";
    protected override string keyName => "id";
}
```

What a journey, to _just_ save 8 lines of code 😂.
Of course, these savings are multiplied by the number of children repositories. But was it worth it?  
No, because this simple problem became bigger and bigger:
- Some of my identifiers weren't `Guid` but were `int` instead.
  - Simple enough, I _just_ had to add a second generic parameter `SqlGenericRepository<Entity, Identifier>` and update the `GetById` method.
- I had to support composite identifier : `(string, id)` 😬
- I had to support ordering and pagination in the `GetAll` method for some entities, etc
- I had to support filtering in the `GetAll` method
- ...

Don't think that using an ORM (like entity framework) will help you in this case. An ORM is already an abstraction, having abstractions over abstractions for the sake of following some rules isn't a good idea. Just pass along the `DbContext` and leave the repository pattern alone. You'll thank me later.
  
In the end, I more or less recreated an ORM ... it was less efficient, with fewer features, with more bugs and became tired of maintaining it.  
And for what?
- I wanted to have a clean app and thus use repositories everywhere
- I got bored of repeating myself

If you are a junior developer, please be smarter than me, don't fall into this trap.  
Don't use the repository for trivial cases, you don't need it.  
Don't use the generic repository antipattern.  
Use an ORM if you want. But even then, you'll eventually find that sometimes it's just easier to write plain SQL.

If you are a more seasoned developer and are using the Generic Repository, please take to moment to weigh the pros and cons.
If you still think that it's worth it, let's discuss it 😊!
