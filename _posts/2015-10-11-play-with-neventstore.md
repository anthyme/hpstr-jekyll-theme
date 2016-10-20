---
layout: post
title:  "Play with NEventStore"
excerpt: "Today I am going to play with the NEventStore library and share some code and feelings about it.
  Briefly it is a .Net library to store events and potentially implement the Event Sourcing pattern with almost any databases as a storage."
date:   2015-10-11 00:00:00
categories: post
tags : [Event Sourcing, C#]
lang : [en]
image:
  feature: abstract-11.jpg
---
 
#Play with NEventStore
 
Today I am going to play with the [NEventStore][NEventStore] library and share some code and feelings about it.
Briefly it is a .Net library to store events and potentially implement the Event Sourcing pattern with almost any databases as a storage.
 
##Event Sourcing
 
###What is Event Sourcing?
 
Before talking about the usage of [NEventStore][NEventStore], let's explain the purpose the Event Sourcing.
 
Usually, applications used to store and update only the last state of application subjects in order to directly get them back in only one aspect. 
But there are drawbacks to this technique of storing data.
Firstly you are losing application usage data. For example data added and removed from a [domain model][DomainModel] is simply lost, as products added and removed from a basket.
Secondly application [domain models][DomainModel] sometime have really complex structure and are hard to save with transactions in relational databases. 
 
Event sourcing is an alternative technique of data storage by not storing actual state of [domain models][DomainModel] but by storing every [domain events][DomainEvents] that happen on the [domain models][DomainModel]. 
The events are immutable in an *append only* mode. They are grouped by "stream" which is identified by a "StreamId" which is, most of the time, the identifier of the [domain models][DomainModel] concerned by the events.
 
There are a lot of advantages to use this technique. 
You know everything that append to the [domain models][DomainModel] so you can go back in time to use an old state of the [domain model][DomainModel]. 
You have a "free" audit history of the [domain models][DomainModel]. 
The immutability and chronology of the events give you an easy ways to duplicate and scale the database. 
The event sourcing is really well suited for polyglot data and CQRS, allowing you to create multiples types of databases like relational, graph, NoSQL, cache, etc. optimized for each purposes of your system. 
 
###How to implement Event Sourcing?
 
When you think about it, Event Sourcing is a pattern and can be implemented (with a small effort) in many existing databases like the classic relational databases.
 
As the pattern gain in popularity, some of its fans want to create a dedicated database engine optimized for the *append only* and immutability of storing events. This is how the EventStore database emerged as the fully event oriented database.
 
So what about the [NEventStore][NEventStore] library? It is a unified API that handle the majority of event sourcing scenarios using many existing databases (more than 20). This API has the big advantage to be well and long thought by experimented developers and avoid beginners to make mistakes on the infrastructure stack as they start using Event Sourcing.
The second great thing about [NEventStore][NEventStore] vs specific databases like [EventStore][EventStore] is that you keep the ability to use the existing tools for development, administration, host, backup, ETL, etc. for the database you choose. 
 
## Let's play with NEventStore!
 
Now I will show you some test code that run [NEventStore][NEventStore]. For asserting result I use the Shouldly library that extend every object in order to test its state.
 
###Models
I must introduce some models that will be part of the tests.
 
First the event class use in the tests.
 
```csharp
public class SomethingHappened
{
    public SomethingHappened(string something)
    {
        Something = something;
    }
 
    public string Something { get; }
}
```
 
Events need some headers to be stored so I create an Event<T> class that compose all the events.
 
```csharp
public class Event<T>
{
    public Event(Guid eventId, Guid streamId, T data)
    {
        EventId = eventId;
        StreamId = streamId;
        Data = data;
    }
 
    public Guid StreamId { get; }
    public Guid EventId { get; }
    public T Data { get; }
}
```
 
And for a more convenient API, I add a factory.
 
```csharp
public class Event
{
    public static Event<T> Create<T>(Guid streamId, T data)
    {
        return Create(Guid.NewGuid(), streamId, data);
    }
 
    public static Event<T> Create<T>(Guid eventId, Guid streamId, T data)
    {
        return new Event<T>(eventId, streamId, data);
    }
}
```
 
### In memory database
 
[NEventStore][NEventStore] can connect to many databases but it can also create an in memory database. It is really pleasant for Testing!

####Configuration

A setup is only on line of fluent code.
 
```csharp
public static IStoreEvents CreateMemoryConnection()
{
    return Wireup.Init()
                .UsingInMemoryPersistence()
                .InitializeStorageEngine()
                .Build();
}
```
 
Pretty simple to setup !

####Add and retrieve an event

Now let's see how to add and retrieve one event to our store.
First I create the store with the previous method.
Then I can open the stream with the OpenStream method of the store. You have to specify the stream identifier but also the min and max revision identifier. With a 0 I choose to let it start from 0.
After that I commit and close the first opening of the stream and reopen it again to see if the event is present.
 
```csharp
var streamId = Guid.NewGuid();
var event1 = Event.Create(streamId, new SomethingHappened("data1"));
 
using (var store = CreateMemoryConnection())
{
    using (var stream = store.OpenStream(streamId, 0))
    {
        stream.Add(new EventMessage { Body = event1.Data });
        stream.CommitChanges(event1.EventId);
    }
 
    using (var stream = store.OpenStream(streamId, 0))
    {
        stream.CommittedEvents.Count.ShouldBe(1);
    }
}
```
 
It worked!

#### Concurrency handling

Now I test a classic case: 2 clients try to modify a domain model at the same time.
A correctly implemented Event Sourcing implementation use an optimistic locking technic and should raise an exception for the last one in order to avoid corruption.
 
```csharp
var streamId = Guid.NewGuid();
var event1 = Event.Create(streamId, new SomethingHappened("data1"));
var event2 = Event.Create(streamId, new SomethingHappened("data2"));
 
using (var store = CreateMemoryConnection())
using (var stream = store.OpenStream(streamId, 0))
using (var stream2 = store.OpenStream(streamId, 0))
{
    stream.Add(new EventMessage { Body = event1.Data });
    stream.CommitChanges(event1.EventId);
 
    stream2.Add(new EventMessage { Body = event2.Data });
    Assert.Throws<ConcurrencyException>(() => stream2.CommitChanges(event2.EventId));
}
```
 
This case is handled too.
 
### Storing events in an SQL Server database
 
####Configuration
 
The setup for a real database is easy too, you can create your connection with few lines of code.
 
```csharp
public const string ConnectionString = 
        "Server=(localdb)\\MSSQLLocalDB;Initial catalog=NEventStore;Integrated Security=true;";
public static IStoreEvents CreateSqlConnection()
{
    var config = new ConfigurationConnectionFactory(
        "NEventStorePoc", "system.data.sqlclient", ConnectionString);
 
    return Wireup.Init()
        .UsingSqlPersistence(config)
        .WithDialect(new MsSqlDialect())
        .InitializeStorageEngine()
        .Build();
}
```
 
There are many customizations available like data encryption, log, performance tracking, dispatching … 
 
####Commits table
 
With this configuration a Commits table will be automatically created at the first usage.

![Commits](https://dl.dropboxusercontent.com/u/44389563/blog/code/play-with-neventstore/commits.png)

####Add and retrieve an event

I put events in the SQL store connection and check if the table contains the event data.
 
```csharp
var streamId = Guid.NewGuid();
var event1 = Event.Create(streamId, new SomethingHappened("data1"));
 
using (var store = CreateSqlConnection())
using (var stream = store.OpenStream(streamId, 0))
{
    stream.Add(new EventMessage { Body = event1.Data });
    stream.CommitChanges(event1.EventId);
}
 
using (var connection = new SqlConnection(ConnectionString))
    connection
    .Query("select * from commits where streamIdOriginal = @streamId", new { streamId })
    .Count().ShouldBe(1);
 
using (var store = CreateSqlConnection())
using (var stream = store.OpenStream(streamId, 0))
{
    stream.CommittedEvents.Count.ShouldBe(1);
}
```

####Concurrency handling

Now I test the concurrency concerns like for the memory store.

```csharp
var streamId = Guid.NewGuid();
var event1 = Event.Create(streamId, new SomethingHappened("data1"));
var event2 = Event.Create(streamId, new SomethingHappened("data2"));
 
using (var store = CreateSqlConnection())
using (var stream = store.OpenStream(streamId, 0))
using (var stream2 = store.OpenStream(streamId, 0))
{
    stream.Add(new EventMessage { Body = event1.Data });
    stream.CommitChanges(event1.EventId);
 
    stream2.Add(new EventMessage { Body = event2.Data });
    Assert.Throws<ConcurrencyException>(() => stream2.CommitChanges(event2.EventId));
}
```

Still working.

####Distributed concurrency handling

But what about a scaled system? With multiples servers?
I will simulate this by creating 2 concurrent connections and create 2 different events on the streams.
 
```csharp
var streamId = Guid.NewGuid();
var event1 = Event.Create(streamId, new SomethingHappened("data1"));
var event2 = Event.Create(streamId, new SomethingHappened("data2"));
 
using (var connection1 = CreateSqlConnection())
using (var connection2 = CreateSqlConnection())
using (var stream = connection1.OpenStream(streamId, 0))
using (var stream2 = connection2.OpenStream(streamId, 0))
{
    stream.Add(new EventMessage { Body = event1.Data });
    stream.CommitChanges(event1.EventId);
 
    stream2.Add(new EventMessage { Body = event2.Data });
    Assert.Throws<ConcurrencyException>(() => stream2.CommitChanges(event2.EventId));
}
```

Still green in the unit test runner, the concurrency is handled at the database level.
 
###Handling Snapshots
 
Event storing is working well but in the Event Sourcing there is also the Snapshot pattern to handle.
The principle of snapshots is simple. If you have too many events for a stream, the stream manipulation can be slow. So when you want to optimize a stream you can create a snapshot that is a state representation at a point in time, store it and when you want to load a stream, you only load the snapshot and the following events.
 
 
####Snapshot in the database
 
The storage of the snapshot is really simple, it is a simple serialized payload stored in an auto created Snapshots table.
 
![Snapshots](https://dl.dropboxusercontent.com/u/44389563/blog/code/play-with-neventstore/snapshots.png)
 
####Snapshot model
 
You need a structure to store your snapshot. For this example as I do not have a [domain model][DomainModel] so I will put an aggregate like logic for the application of events in the snapshot in order to create it.
 
```csharp
public class SomeSnapshot
{
    public SomeSnapshot() { }
    public SomeSnapshot(IEnumerable<object> events)
    {
        foreach (var @event in events)
        {
            try
            {
                Apply((dynamic)@event);
            }
            catch (RuntimeBinderException)
            {
                //unhandled event, skip
            }
        }
    }
 
    public ICollection<string> State { get; set; } = new List<string>();
 
    public Guid Id { get; set; }
 
    public int Version { get; set; }
 
    private void Apply(SomethingHappened @event)
    {
        State = State.Union(new[] { @event.Something }).ToList();
    }
}
```
 
To test snapshot I will first create 3 events on a stream and save them to the store.
Then I will create a snapshot at this point.
Il will then get back the snapshot an reopen the stream and add d 2 more events.
And again I will reopen the stream with the snapshot and I should get only the 2 last events from the stream
 
 
```csharp
var streamId = Guid.NewGuid();
var event1 = Event.Create(streamId, new SomethingHappened("data1"));
var event2 = Event.Create(streamId, new SomethingHappened("data2"));
var event3 = Event.Create(streamId, new SomethingHappened("data3"));
 
using (var connection1 = CreateSqlConnection())
{
    using (var stream = connection1.OpenStream(streamId, 0))
    {
        stream.Add(new EventMessage { Body = event1.Data });
        stream.Add(new EventMessage { Body = event2.Data });
        stream.Add(new EventMessage { Body = event3.Data });
        stream.CommitChanges(event1.EventId);
 
        var snapshotData = new SomeSnapshot(new[] { event1.Data, event2.Data, event3.Data });
        var snapshot = new Snapshot(stream.StreamId, stream.StreamRevision, snapshotData);
        connection1.Advanced.AddSnapshot(snapshot);
        stream.CommitChanges(Guid.NewGuid());
        stream.CommittedEvents.Count.ShouldBe(3);
    }
 
    var snapshot2 = connection1.Advanced.GetSnapshot(streamId, int.MaxValue);
    ((SomeSnapshot)snapshot2.Payload).State.ShouldBe(new[] { "data1", "data2", "data3", });
 
    using (var stream = connection1.OpenStream(snapshot2, int.MaxValue))
    {
        stream.CommittedEvents.Count.ShouldBe(0);
 
        var event4 = Event.Create(streamId, new SomethingHappened("data4"));
        var event5 = Event.Create(streamId, new SomethingHappened("data5"));
        stream.Add(new EventMessage { Body = event4.Data });
        stream.Add(new EventMessage { Body = event5.Data });
        stream.CommitChanges(event4.EventId);
    }
 
    var snapshot3 = connection1.Advanced.GetSnapshot(streamId, int.MaxValue);
    ((SomeSnapshot)snapshot3.Payload).State.ShouldBe(new[] { "data1", "data2", "data3", });
 
    using (var stream = connection1.OpenStream(snapshot3, int.MaxValue))
    {
        stream.CommittedEvents.Count.ShouldBe(2);
        stream.CommittedEvents.Select(x => x.Body).OfType<SomethingHappened>().Select(x => x.Something)
            .ShouldBe(new[] { "data4", "data5" });
    }
}
```
 
##Conclusion
Working with Event Sourcing can be tricky at first but as you can see [NEventStore][NEventStore] is a really easy to use library and it can help you to start smoothly.
A memory storage is available in order to simplify testing with the same API.
You can handle the storage of event and snapshots with many databases from the relational world or even the NoSQL world.
 
You can find the source code on my [GitHub][NEventStore-Github].

[DomainEvents]: http://martinfowler.com/eaaDev/DomainEvent.html
[DomainModel]: https://en.wikipedia.org/wiki/Domain_model
[NEventStore]: https://github.com/NEventStore/NEventStore
[EventStore]: https://geteventstore.com/
[NEventStore-Github]: https://github.com/anthyme/NEventStoreTests/tree/master/NEventStorePoc
