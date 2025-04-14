<h2 id="chapter-iv" >Chapter IV</h2>
<h3 id="ex00">Task 00: Scalability</h3>

After some time, the board was covered with writings.

"Command line access — there should be a separate application that provides the REPL and connects to a running instance over the network, even if it's just a local host and port."

"We should be able to kill any instance (process) of the database and it should continue to run and provide answers to queries. This means that one of the configurable parameters, for example, should be a replication factor, i.e. how many copies of the same document we store. For testing purposes, 2 is probably enough."

"The client should perform heartbeats to check if the current database instance is available. If it stops responding, it should automatically switch to another instance."

"Also, for simplicity, let's assume for now that being scalable means that the client should be aware of all other nodes. Any heartbeat response from a current node should include all currently known instance addresses and ports along with the current replication factor."

So here we need to implement two programs — one is the client and one is an instance of a database. Whenever you start a new instance, you should be able to point it to an existing instance, so that after receiving a heartbeat it will send its host and port to all other running nodes, and everyone will know the new guy.

If the instance node is started with a different replication factor than existing nodes, it should detect this and automatically fail without joining the cluster. This means that the replication factor should probably be included in the heartbeat as well.

You can use any network protocol for this — HTTP, gRPC, etc.

Whenever the replication factor is more than a number of running nodes, information about this problem should be included in a heartbeat and explicitly displayed in each connected client. You can see an example of a user session in Task 01.

Actual work with documents will be implemented in the next task.

<h2 id="chapter-v" >Chapter V</h2>
<h3 id="ex01">Task 01: Balancing and Queries</h3>

"Okay, so let's use UUID4 strings as artifact keys. We also need to implement some balancing to provide fault tolerance..."

Our simple database should only support three operations — GET, SET and DELETE. 

Here's what a typical session should look like, with comments (starting with #):

```
~$ ./warehouse-cli -H 127.0.0.1 -P 8765
Connected to a database of Warehouse 13 at 127.0.0.1:8765
Known nodes:
127.0.0.1:8765
127.0.0.1:9876
127.0.0.1:8697
> SET 12345 '{"name": "Chapayev's Mustache comb"}'
Error: Key is not a proper UUID4
> SET 0d5d3807-5fbf-4228-a657-5a091c4e497f '{"name": "Chapayev's Mustache comb"}'
Created (2 replicas)
> GET 0d5d3807-5fbf-4228-a657-5a091c4e497f
'{"name": "Chapayev's Mustache comb"}'
> DELETE 0d5d3807-5fbf-4228-a657-5a091c4e497f
Deleted (2 replicas)
> GET 0d5d3807-5fbf-4228-a657-5a091c4e497f
Not found
>
# if current instance is stopped in the background
Reconnected to a database of Warehouse 13 at 127.0.0.1:8697
Known nodes:
127.0.0.1:9876
127.0.0.1:8697
> 
# if another current instance is stopped in the background
Reconnected to a database of Warehouse 13 at 127.0.0.1:9876
Known nodes:
127.0.0.1:9876
WARNING: cluster size (1) is smaller than a replication factor (2)!
>
```

If a key specified in SET already exists in a database, the value should be overwritten. If it doesn't, then the SET operation should provide read-after-write consistency, meaning that an immediate read should return the correct value.

When updating or deleting an existing value, eventual consistency should be implemented, meaning that immediate (dirty) reads may (but not "should"!) give you old results, but after a few seconds the data should be updated to a proper new state.

You can implement key-hash-based balancing, so your client can explicitly compute for each entry the list of nodes where it should be stored according to a replication factor. This is also useful for deletion.

If a current node is killed while writing, your client should automatically re-request to another available node. The only case where the user should see an error like "Failed to write/read an entry" is when ALL instances are dead.

<h2 id="chapter-vi" >Chapter VI</h2>
<h3 id="ex02">Task 02: Long Live the King</h3>

Let's update the logic from Tasks 00/01. We now introduce the concepts of a Leader and a Follower node. This leads to a list of important changes:

* From now on, the client ONLY interacts with a Leader node. The hash function to determine where to write replicas is now *on* the Leader, *not* in the client.
* All nodes (Leader and Follower) keep sending each other heartbeats with a full list of nodes. If a node doesn't respond to heartbeats for a certain configurable timeout (for testing purposes you should set it to 10 seconds by default).
* If the Leader is stopped, the remaining Followers should be able to choose a new Leader from among them. For simplicity, each of them can just sort the list of nodes by some other unique identifier (numeric id, port, etc.) and pick the top one. From that moment on, all heartbeats will contain a new elected Leader.
* If a client is unable to connect to a known Leader, they should try to connect to Followers to get a heartbeat from them. If a Leader is killed, this heartbeat will contain a new elected Leader.

<h2 id="chapter-vii" >Chapter VII</h2>
<h3 id="ex03">Task 03: Consensus</h3>

**NOTE: this task is completely optional. It is only graded as a bonus part**

You may have noticed that a lot of things can go wrong in a schema provided above, specifically race conditions and the possibility of losing some data because replicas are not automatically resynchronized between instances after some of them die.

You can try to fix this for some extra credit, either by using an existing solution or by writing a workaround yourself. Here are some options:

