---
layout: post
title: Creating a Finite State Machine Distributed Workflow with Service Fabric Reliable Actors
date: "2017-11-21"
categories:
  - service fabric
  - csharp
---

<p class="message">The accompanying source code can be found at <a href="https://github.com/jsturtevant/azure-service-fabric-actor-fsm">github.com/jsturtevant/azure-service-fabric-actor-fsm</a>.
</p>

I recently had to manage a long running workflow inside Azure Service Fabric solution.  I remembered reading about using a [Finite State Machine](https://en.wikipedia.org/wiki/Finite-state_machine) (FSM) with Actors in the book [Programming Microsoft Azure Service Fabric](https://www.amazon.com/Programming-Microsoft-Service-Developer-Reference/dp/1509301887/) by Haishi Bai.  When I went looking for examples of how I might use a FSM with the [Service Fabric Actor Programming model](https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-reliable-actors-introduction) I wasn't able to find one, so I built [an example of using a FSM with Service Fabric Actors](https://github.com/jsturtevant/azure-service-fabric-actor-fsm).

Although, Finite State Machines might sound scary they are not to hard to implement or us, once you learn some basic [terminology](https://en.wikipedia.org/wiki/Finite-state_machine#Concepts_and_terminology).  FSM's are great for many types of scenarios but keeping tack of long running workflows with a predetermined sequence of actions is a great use case.  

In Haishi's book, he walks through the example of how it might be used in an ordering and shipping process to track items as they move through the ordering, manufacturing, shipping and invoicing product stages.  In my case I needed to manage creating and configuring multiple resources within my Service Fabric cluster (applications and services) and [Azure](https://azure.microsoft.com/en-us/) (IP addresses and Load balancers, etc).  Each individual step required the previous step to be complete and could take anywhere from a few seconds to 10 minutes. 

In such a distributed workflow, where there are many different services implementing each of the individual steps you it can be challenge to track and manage all of the different steps.  A single Actor for each instance of the workflow simplifies the process and simplifies the scaling of the workflow service.  For each instance of a new workflow a new actor is created and the state for the workflow is managed by that actor.  

An example of a Actor managing multiple states and services might look like:

![Finite State Machine Workflow using Reliable Actors and multiple distributed services.]({{ site.url }}/assets/fsm-actor-distributed-workflow.png)

A few of the benifits of using a Reliable Actor with a FSM in this case are:

- FSM will centralize the logic for the workflow into one location
- the workflow service will scale well as each actor has independent state and logic for a given workflow instance
- can use timed [Reminders](https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-reliable-actors-timers-reminders) on the actor to trigger workflow timeouts and notify other services something is wrong.
- monitoring status of the workflow is simplified 
- canceling a workflow is simplified (if in correct state)

A notable trade off of using Reliable Actors are they are inheritly single threaded.  If your workflow requires a high number of reads and writes then you might find that the Actors will ***not*** be a good choice do to throughput performance.  In both of the use cases noted above this is not an issue so the models maps nicely to the Reliable Actors.

## Create the Finite State Machine (FSM) with Service Fabric Actor
Instead of implementing a FSM myself I used the very popular [Stateless FSM library](https://github.com/dotnet-state-machine/stateless).  I chose this library because of its popularity and it comes with the ability to externalize the state of the FSM.  In this case I wanted to [persist the state using a Service Fabric Stateful Reliable Actor](https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-reliable-actors-state-management).  

You can install the library via Nuget `Install-Package Stateless` and checkout some of the [examples to get an understanding](https://github.com/dotnet-state-machine/stateless/tree/dev/example) of to use the library. 

I adapted the [BugTracker example](https://github.com/dotnet-state-machine/stateless/tree/dev/example/BugTrackerExample) to create the Service Fabric code samples below. 

For the remainder of the article we will walk through some of the important parts of the solution.

### Working with external State inside the Actor
Stateless uses a simple enum to track the state the FSM is in at any given time.  We can use Reliable Actors `StateManager` to save this value durably when ever the State Machine changes state.  

Using external state in Stateless is fairly easy:

```csharp
//Private member Variable
enum State { Open, Assigned, Deferred, Resolved, Closed }

//initializing Stateless StateMachine with external state.
machine = new StateMachine<State, Trigger>(() => this.state, s => this.state = s);
```  

Saving the State durably (replicated across multiple nodes for failover) using Reliable Actors is also straight forward. Within any method of the Actor we can pass a `key` and `value`:

```csharp
await this.StateManager.SetStateAsync("state", machine.State);
```

Now let's see how we can use this to create a FSM using Service Fabric Reliable Actors.

### Initializing the State of the Actor 
Reliable Actors are *virtual*, which means they always "exist" and the lifetime of an actor is not tied to their in-memory representation. An individual actor can also be "Garbage Collected" (deactivated) from the system at if not used for a period of time, although the state will be persisted.  This means two things when an working with the FSM in the actor framework:

- The first time an actor is activated we need to set the initial FSM state and configure the FSM
- When a new request comes in for a Actor that has been deactivated we will need to load the state and re-configure the FSM

The code to handle both first time activation and re-activation happens in the  `OnActivateAsync` method and will look something like:

```csharp
protected override async Task OnActivateAsync()
{
    // Handle Activation for first time and re-activation after being garbage collected
    ActorEventSource.Current.ActorMessage(this, $"Actor activated: {this.GetActorId()}");

    var savedState = await this.StateManager.TryGetStateAsync<State>("state");
    if (savedState.HasValue)
    {
        // has started processing
        this.state = savedState.Value;
    }
    else
    {
        // first load ever - initialize
        this.state = State.Open;
        await this.StateManager.SetStateAsync<State>("state", this.state);
    }

    ActorEventSource.Current.ActorMessage(this, $"Actor state at activation: {this.GetActorId()}, {this.state}");

    //Configure the Stateless StateMachine with the state.
    machine = new StateMachine<State, Trigger>(() => this.state, s => this.state = s);
    assignTrigger = machine.SetTriggerParameters<string>(Trigger.Assign);
    machine.Configure(State.Open)
        .Permit(Trigger.Assign, State.Assigned);
    machine.Configure(State.Assigned)
        .SubstateOf(State.Open)
        .OnEntryFromAsync(assignTrigger, assignee => OnAssigned(assignee))
        .PermitReentry(Trigger.Assign)
        .Permit(Trigger.Close, State.Closed)
        .Permit(Trigger.Defer, State.Deferred)
        .OnExitAsync(() => OnDeassigned());
    machine.Configure(State.Deferred)
        .OnEntryAsync(() => this.StateManager.SetStateAsync<string>("assignee", null))
        .Permit(Trigger.Assign, State.Assigned);
}
```

You will notice that the state machine gets saved in a local member variable.  From here on out we will interact with our state machine in memory and persist the state object using the `StateManager` when ever the FSM Transitions from one state to another.

### Triggering a new state from external system
In the case of managing a distributed workflow, the external services that are doing the work of at each step of the workflow will need a way to notify the workflow actor that there work has either completed or failed.  We can create methods on the actor that will enable the clients to interact with the actor and workflow.  If any service requests a transition of state outside of the workflow, an error will be returned to the client notifying them that the action is not available at this time.

The first step is to modify the interface that Actor implements:

```csharp
public interface IFSM : IActor
{
    Task Close(CancellationToken cancellationToken);
    Task Assign(string assignee, CancellationToken cancellationToken);
    Task<bool> CanAssign(CancellationToken cancellationToken);
    Task Defer(CancellationToken cancellationToken);
}

[StatePersistence(StatePersistence.Persisted)]
internal class FSM : Actor, IFSM
{
  /// implement members and methods like OnActivateAsync here
}
```

The next step is to implement the methods and save the FSM state as the workflow transitions.  Since the FSM is in memory when an actor is active we can work with the FSM's actual state:

```csharp
public async Task Assign(string assignee, CancellationToken cancellationToken)
{
    //trigger transition to next workflow state.
    await machine.FireAsync(assignTrigger, assignee);

    //check that local state is same as machine state then save.
    Debug.Assert(state == machine.State);
    await this.StateManager.SetStateAsync("state", machine.State);
}
``` 

This creates a transition in the FSM (`assignTrigger`) and fires an event which could then call out to another service to kick of the next step of the workflow.  Now that the next step in the workflow has been activated we can save the state of the machine.  Calling the `Assign` method from an external service is simple:

```csharp
ActorId actorId = new ActorId(Guid.NewGuid().ToString());
IFSM actor = ActorProxy.Create<IFSM>(actorId, new Uri("fabric:/ActorFSM/FSMActorService"));
var token = new CancellationToken();
await actor.Assign("Joe", token);
```

In the `assignTrigger` of the FSM we can also save the the actual assignee durably using the Actors `StateManager`.  This means that we can not only query the actor to get the current step of the workflow but also who is assigned.  The event wired to `FireAsync` looks like:

```csharp
async Task OnAssigned(string assignee)
{
    var _assignee = await this.StateManager.TryGetStateAsync<string>("assignee");
    if (_assignee.HasValue && assignee != _assignee.Value)
      await SendEmailToAssignee("Don't forget to help the new employee!");

    await this.StateManager.SetStateAsync("assignee", assignee);
    await SendEmailToAssignee("You own it.");
}
```

### Getting current State of a workflow
A system out side our workflow might want to know what state a given workflow instance is in.  This is again is made simple through the Actor.  What is interesting to note is that since the actor is single threaded, you will always get the latest state of the Workflow.

Implement the current status as a method on the Actor:

```csharp
public async Task<BugStatus> GetStatus()
{
  var statusHistory = await this.StateManager.TryGetStateAsync<StatusHistory>("statusHistory");
  var assignee = await this.StateManager.TryGetStateAsync<string>("assignee");

  var status = new BugStatus(this.machine.State);
  status.History = statusHistory.HasValue ? statusHistory.Value : new StatusHistory(machine.State);
  status.Assignee = assignee.HasValue ? assignee.Value : string.Empty;

  return status;
}
```

Getting the current status:

```csharp
var actor1Status = await actor.GetStatus();
Console.WriteLine($"Actor 1 with {actorId} has state {actor1Status.CurrentStatus}");
```

Knowing the history of all the states a workflow has gone through will almost certainly be a requirement.  Updating the workflow history as it move throught the FSM is straight forward using the machines `machine.OnTransitionedAsync(LogTransitionAsync)` event and wiring up a function to log it:

```csharp
private async Task LogTransitionAsync(StateMachine<State, Trigger>.Transition arg)
{
    var conditionalValue = await this.StateManager.TryGetStateAsync<StatusHistory>("statusHistory");

    StatusHistory history;
    if (conditionalValue.HasValue)
    {
        history = StatusHistory.AddNewStatus(arg.Destination, conditionalValue.Value);
    }
    else
    {
        history = new StatusHistory(arg.Destination);
    }

    await this.StateManager.SetStateAsync<StatusHistory>("statusHistory", history);
}
```

You can see that the `GetStatus` method previously implemented already returns that value.  

## Testing state persistance
As was discussed earlier, the Reliable Actor can be Garbage collected from the system if not used with in a given time period.  I wanted to test this to make sure the Activation code worked properly with the FSM.  The actor framework gives you some configuration settings that you can modify to tweak your actor's behavior.  I used these  settings to aggressively garbage collect the Actors to test the state.  

> Note: Only modify these Actor settings is you have data to backup making a change to the default settings.  In other words you should have sufficient monitoring set up that points to your actors being garbage collected to early or to late for your scenario.

During registration of the Actor with Service Fabric in the program.cs you can adjust the ActorGarbageCollectionSettings.  The first number is scan interval (how often it checks the system) and the second number is the idle time out (how long an actor must be idle for):

```csharp
internal static class Program
{
  private static void Main()
  {
      try
      {
          ActorRuntime.RegisterActorAsync<FSM>(
              (context, actorType) => new ActorService(context, actorType, settings: new ActorServiceSettings()
              {
                  ActorGarbageCollectionSettings = new ActorGarbageCollectionSettings(10, 5)
              })).GetAwaiter().GetResult();

          Thread.Sleep(Timeout.Infinite);
      }
      catch (Exception e)
      {
          ActorEventSource.Current.ActorHostInitializationFailed(e.ToString());
          throw;
      }
  }
}
```

Then during my test I waited period of time before working with the actors and used the Service Fabric management API's to make sure the Actors were deactivated:

```csharp
ActorId actorId2 = new ActorId(Guid.NewGuid().ToString());
IFSM actor2 = ActorProxy.Create<IFSM>(actorId2, new Uri("fabric:/ActorFSM/FSMActorService"));
await actor2.Assign("Sue", token);

// Wait for actors to get garbage collected
await Task.Delay(TimeSpan.FromSeconds(20));

//Query management API's to verify
var actorService = ActorServiceProxy.Create(new Uri("fabric:/ActorFSM/FSMActorService"), actorId);
ContinuationToken continuationToken = null;
IEnumerable<ActorInformation> inactiveActors;
do
{
    var queryResult = await actorService.GetActorsAsync(continuationToken, token);
    inactiveActors = queryResult.Items.Where(x => !x.IsActive);
    continuationToken = queryResult.ContinuationToken;
} while (continuationToken != null);

foreach (var actorInformation in inactiveActors)
{
    Console.WriteLine($"\t {actorInformation.ActorId}");
}
```

## Conclusion
Creating a distributed workflow with multiple steps and variable times to complete can be a difficult problem.  Using Service Fabric's Reliable Actors with a FSM machine library like Stateless not only simplifies the code that needs to be written but also ensures the scalablity of the solution because of the partitioned nature of Reliable Actors.  If you don't need lots of concurrent reads and writes then the Reliable Actors could be a great solution.

You can find the complete sample code for this solution at https://github.com/jsturtevant/azure-service-fabric-actor-fsm.