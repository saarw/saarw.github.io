---
layout: post
title:  "Building a collaborative React app with Flushout"
date:   2020-03-02 18:42:49 +0700
categories: dev
---
This posts demonstrates using the [Flushout](https://github.com/saarw/flushout) distributed data model to synchronize React states to build a simple collaborative todo-list app. You can run the example [in your browser](https://eager-almeida-2b573e.netlify.com/) or view the [source](https://github.com/saarw/flushout-example) of the example on Github.

## Flushout and event sourcing
Flushout is a distributed data model with a local client state model that gets updated without any network delays for fast UI updates while also synchronizing with a remote master model in the background to support collaboration. 

Flushout is based on event sourcing. Event sourcing uses sequences of commands to construct aggregate states and enables incremental replication of states by only replicating any new commands applied to a state rather than always replicating the whole state. Event sourcing also makes it easy for applications to support undo operations or update history views by inspecting and rebuilding the state from its commands.

Unlike other collaboration data structures like CRDTs, event sourcing commands must be applied in the same order to construct the same state, but event sourcing makes it easy to separate the command history from the smaller state snapshots to minimize network transfer and memory requirements for models. The consistent order also means the number of commands that have been applied to construct a state also works as a version for that state.

Flushout was created to support state synchronization between multiple devices and multi-user collaboration for SaaS apps [ImpactMiner](https://impactminer.com) and [PlotDash](https://plotdash.com).
## The example app
Clients that use Flushout for collaborative state typically retrieve a snapshot of the latest state from the backend and uses it to initialize a local Flushout proxy. The user’s input are applied as create, update, and delete commands on the proxy, which immediately updates the local state to drive the user interface while also storing the commands for later replication to the backend. 

<script charset="UTF-8" src="https://gist-it.appspot.com/github.com/saarw/flushout-example/commit/2ebceec6a6ee78b34405f3890857bef8f1850eed/src/Client.tsx?slice=75:84&footer=minimal"></script>
<small>Our example initializes the state of each client with a run-once React effect hook. The example only tracks command counts to demonstrate Flushout's inner workings in the UI, omitted in a real application.</small>

<script charset="UTF-8" src="https://gist-it.appspot.com/github.com/saarw/flushout-example/commit/2ebceec6a6ee78b34405f3890857bef8f1850eed/src/Client.tsx?slice=69:72&footer=no"></script>
...
<script charset="UTF-8" src="https://gist-it.appspot.com/github.com/saarw/flushout-example/commit/2ebceec6a6ee78b34405f3890857bef8f1850eed/src/Client.tsx?slice=117:125&footer=minimal"></script>
<small>When the user clicks to add a new todo, we apply a Create command to the local proxy.</small>

### Synchronizing with the master model
To demonstrate Flushout's operations, our application requires the user to manually trigger synchronization with the master model by pressing the Flush button. This would normally happen automatically in the background when the user interacts with the client or when the client gets notified of server updates.

A flush is a list of commands that have been applied to the local proxy but have not been transmitted to the master. The master's response to a flush updates the local state of the proxy to that of the master, including any changes performed on the master by other clients.
<script charset="UTF-8" src="https://gist-it.appspot.com/github.com/saarw/flushout-example/commit/2ebceec6a6ee78b34405f3890857bef8f1850eed/src/Client.tsx?slice=132:135&footer=minimal"></script>

## The backend
Our application runs completely in the browser so the client works against a simple backend API that is implemented locally but uses asynchronous operations that could easily be implemented with a remote backend over a REST or RPC API.
<script charset="UTF-8" src="https://gist-it.appspot.com/github.com/saarw/flushout-example/commit/2ebceec6a6ee78b34405f3890857bef8f1850eed/src/types.ts?slice=11:15&footer=minimal"></script>


Our backend implementation demonstrates applying flushes from clients to a master model and also includes an interceptor to demonstrate setting values that are computed when commands are applied to the master model. 

<script charset="UTF-8" src="https://gist-it.appspot.com/github.com/saarw/flushout-example/commit/2ebceec6a6ee78b34405f3890857bef8f1850eed/src/Backend.ts?slice=14:25&footer=minimal"></script>


<small>Interceptors can also be used to check model-dependent validation, such as whether a user has reached the maximum number of todos they are allowed to create.</small>

Command operations that do not depend on the state of the model, such as validating or attaching the session user ID to each command, can be done by the backend before applying the commands on the master.

### Real-world backend implementation
The backend in our example does not persist its model and does not keep its command history, so the updates it returns when the client flushes the model will always be a full state snapshot if other clients have modified the master's state. 

A simple backend implementation may just store the snapshot as JSON with its command count in a separate column of a database. When a flush arrives from a client, the backend can initialize a master from the database snapshot, apply the flush and optimistically store the new snapshot and command count in the database if the command count has not changed (do UPDATE ... WHERE command_count = *command count that I read*). If the update fails because another client updated the same model and changed the command count, redo the whole read-apply-update cycle until it succeeds.

A more advanced backend may store the master snapshot and all or some of its commands (to support historical views of updates, or a number of undo steps) and may use caching for all or some of the data. Storing command history can reduce network traffic to clients as it allows Flushout to use incremental updates to keep clients synchronized.

Database and cache access in NodeJS happens asynchronously which may cause multiple flush operations from different clients to be processed concurrently on the same data model. Backends can use library [async-sequential-runner](https://github.com/saarw/async-sequential-runner) to make sure updates in a single server run sequentially for each data model.

### How do we scale this thing?
While a Node server will handle quite a lot of traffic, you may eventually end up needing multiple instances if the single-threaded event loop gets saturated with JSON encoding/decoding work or to achieve high-availability. Having many users on different servers update the same Flushout master model may result in contention using the simple optimistic update scheme described above.

A resilient and scalable solution is to publish all the flushes for each model on its own key in a queue that supports keyed partitioning, such as Apache Kafka. A stream processor, such as Kafka streams, or a subscriber-producer service, can then maintain and update master model states and publish flush responses to clients on the queue. This eliminates contention as Kafka message processing can be done sequentially for each key and may remove the need to store model state in a database or use caching. 

Unfortunately, serverless functions on common cloud queues such as AWS Lambda on Kinesis or Google Cloud Functions on Pubsub have poor support for key-partitioned processing and are likely to process commands in parallell for the same model.

Other scaling alternatives is to separate your Flushout master operations into a micro-service with multiple instances behind a proxy that distributes commands for each model to a particular instance, for example using URL-based load balancing. The Flushout master processing code is also small enough to be easily ported to a language such as Java or Rust that can utilize powerful multi-core servers better than Node (JVM-based JavaScript runtime [ES4X](https://reactiverse.io/es4x/) may be able to support this even without porting).
