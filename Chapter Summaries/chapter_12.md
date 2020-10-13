# Building a distributed system
To have a reliable, fault-tolerant server you need clusters
of machines sharing the work. These clusters have Nodes which
handle the incoming requests as needed. By horizontal scaling
you can prevent crashing the server and server overload.

## 12.1 Distibution primitives
BEAM-powered distributed systems are built by connecting multiple nodes into a cluster.
A node is a BEAM instance that has a name associated with it.

### 12.1.1 Starting a cluster
To start a cluster you need some Nodes
Using --sname turns your BEAM instance into a node with the name node1@localhost.
The part before the @ character is a prefix that uniquely identifies a node on a single
machine. The second part (localhost) identifies the host machine. If you omit the
host part, the host machine’s name is automatically used.

### 12.1.2 Communicating between nobes
Once you have some nodes started and connected, you can make them cooperate. A
simple way to try this is to use Node.spawn/2, which receives a node name (an atom)
and a lambda. The function then spawns a new process on the target node and runs
the lambda in that process.

#### Listing 12.1 Spawning a process on another node
Group leader process: Something unexpected is happening in listing 12.1. Even though the lambda has been
executed on node2, the output is printed in the shell of node1. How is this possible? The
reason lies in how Erlang does standard I/O operations.

### 12.1.3 Process discovery
A client process must obtain the server’s pid. A client sends a message to the server.
Even in a single-node system, you must somehow find the target process’s pid. This
doesn’t change in a distributed setting, but you must use another means of discovery
because Registry isn’t cluster-aware and works only in the scope of a local node.
Note that lookups are local. When a registration is being performed, all nodes are
contacted, and they cache the registration information in their local ETS table


### 12.1.4 Links and monitors
Links and monitors work even if processes reside on different nodes.
A process receives
an exit signal or a :DOWN notification message (in the case of a monitor) if any of the
following events occurs:
Crash of a linked or monitored process
Crash of a BEAM instance or the entire machine where the linked or monitored
process is running
Network connection loss

### 12.1.5 Others distrbution
Other interesting services are provided as part of the Erlang standard library.
Occasionally you’ll need to issue function calls on other nodes.
More often, you’ll want to obtain the result of a remote function call or invoke a
function on multiple nodes and collect all the results.
For example, to call a function on another node and get its result, you can use
:rpc.call/4, which accepts a node and an MFA identifying the function to be called
remotely
## 12.2 Building a fault-tolerant cluster
Most of your work here will be based on the GenServer abstraction. This shouldn’t
come as a surprise, given that message passing is the main distribution tool in BEAM.

### 12.2.1 Cluster design
The cluster will consist of multiple nodes, all of which are powered by the same
code and provide the same service (a web interface for managing multiple to-do
lists).
Network partitions:
A partition is a situation in which a communication channel between two nodes
is broken and the nodes are disconnected. In this case, you may end up with a “split brain”
situation: the cluster gets broken into two (or more) disconnected smaller clusters, all of
which work and provide service.

### 12.2.2 The distributed to-do cache
When you want to modify a to-do list, you ask the to-do cache to provide the corresponding
to-do server process for you. This to-do server then acts as a synchronization
point for a single to-do list. All requests for Bob’s list go through that process, which
ensures consistency and prevents race conditions.

#### Listing 12.2 Global registration of to-do servers (todo_distributed/lib/todo/server.ex)
This is the single thing you need to change to make your to-do cache distributed. As
you’ll see, the changes are reasonably straightforward.
#### Listing 12.3 Discovering to-do servers (todo_distributed/lib/todo/server.ex)
With this single simple change, you’ve switched to distributed registration and discovery.
There’s no need to change anything else; the system will work properly.
#### Listing 12.4 Optimized process discovery (todo_distributed/lib/todo/cache.ex)
It’s worth repeating that :global.whereis_name/1 doesn’t lead to any cross-node
chatting. This function only makes a single lookup to a local ETS table. Therefore,
you can expect pretty good and stable performance from the Todo.Server.whereis/1
function.
### 12.2.3 Implementing a replicated database
Obviously the database needs to be replicated so that data can survive node crashes.
The simplest (although not the most efficient) way of preserving data is to replicate it in
the entire cluster. To deal with this situation properly, you need to implement a two-phase commit
protocol and a log-based database implementation. This would allow you to safely roll
back in case of a partial failure. For the sake of simplicity and brevity.


#### Listing 12.5 Storing data on all nodes (todo_distributed/lib/todo/database.ex)
Here you rely on :rpc.multicall/4 to make a function call on all nodes in the cluster.
multicall accepts MFA and a timeout. The target function is then invoked on all
nodes in the cluster, all results are collected, and you get a tuple in the form {results,
bad_nodes}: a list of results and a list of nodes that didn’t respond in the given time.

### 12.2.4 Testing the system
You need to start a few nodes, connect them, and see
how the cluster works. But recall that in chapter 11 you made the web server listen on
port 5454. You can’t have two nodes listening on the same port, so you need to change
this. Luckily, in section 11.4 you made the web port configurable via the application
environment, so it’s possible to change the default port from the command line.

### 12.2.5 Detecting partitions
When you decide to go distributed, partitions are a problem you’ll have to deal with,
one way or another. Network partitions shouldn’t be ignored in a distributed system,
so even if you reach for a third-party product (such as an external database) to handle
clustering and replication, you should understand how that product behaves when
partitions occur.

### 12.2.6 Highly available systems
The main tool for high availability is the BEAM
concurrency model. Relying on processes provided many nice properties and made it
possible to come close to having a proper highly available system.
When you set out to implement a highly available system that must serve a multitude of users continuously, these are the properties
you’ll need to achieve, and processes are the main tool that can take you there.
## 12.3 Network considerations
So far, you’ve been running nodes locally. This is fine for making local experiments
and doing development-time testing. But in production you’ll usually want to run different
nodes on different machines. When running a cross-host cluster, you need to
consider some additional details.
### 12.3.1 Node names
A node name plays an important role when establishing a connection. Recall that a
name uses the form arbitrary_prefix@host (short name) or arbitrary_prefix@
host.domain (long name). This name obviously identifies a BEAM instance on a
machine. The second part of the name (host or host.domain) must be resolvable to
the IP address of the machine where the instance is running. When you attempt to
connect to node2@some_host.some_domain from node1, the node1 host must be able to
resolve some_host.some_domain to the IP address of the host machine.
### 12.3.2 Cookies
Notice that the cookie is internally represented as an atom. A node running on
another machine will have a different cookie, so connecting two nodes on different
machines won’t work by default; you need to somehow make all nodes use the same
cookie. Cookies provide a bare minimum of security and also help prevent a fully connected
cluster where all nodes can directly talk to each other
### 12.3.3 Hidden nodes
A simple example is starting a local node that acts as a
remote shell to another node. Another example is an instrumentation tool — a node
that connects to another node, collects all sorts of metrics from it, and presents the
results in a GUI.
### 12.3.4 Firewalls
When one node wants to connect to
another node on a different machine, it needs to communicate with two different components.
The first component, the Erlang Port Mapper Daemon (EPMD), is an OS process
that’s started automatically when you start the first Erlang node on the host machine.
Each node listens on a random port that needs to be accessible as well,
because it’s used to establish the connection between two nodes. Obviously it’s not particularly
helpful that the node’s listening port is random, because it’s not possible to
define firewall rules.
### Summary
¡ Distributed systems can improve fault-tolerance, eliminating the risk of a single
point of failure.
¡ Clustering lets you scale out and spread the total load over multiple machines.
¡ BEAM-powered clusters are composed of nodes, which are named BEAM instances
that can be connected and can communicate.
¡ Two nodes communicate via a single TCP connection. If this connection is broken,
the nodes are considered disconnected.
¡ The main distribution primitive is a process. Sending a message works the same
way, regardless of the process location. A remotely registered process can be
accessed via {alias, node_name}.
¡ Many useful higher-level services built on top of those primitives are available in
the :global, :rpc, and GenServer modules.
¡ When communicating between nodes, use calls rather than casts.
¡ Always consider and prepare for netsplit scenarios.
