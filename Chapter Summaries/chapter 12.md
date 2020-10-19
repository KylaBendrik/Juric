# 12 Building a distributed system
Let's make this system more reliable. In this chapter, "distributed" may refer more to using multiple "nodes" running the system, rather than packaging the app and sending it to customers.
#### Figure 12.1 To to-do system as a cluster
Clients can speak to one of three nodes, each of which is running Todo.Server and Todo.Database. 
## 12.1 Distribution primitives
BEAM-powered systems are built by connecting multiple nodes into a cluster
### 12.1.1 Starting a cluster
To set up a cluster, you need to start a couple of nodes. Use --sname parameter while starting the shell. The --sname turns the BEAM instance into a node with the name provided.
 - node() gives you the name of the node (in this case, :node1@localhost)
 - Node.connect() lets you connect the two nodes. Use the atom style above ^ It will automatically connect connected nodes.
 - Node.monitor/1 lets you register and recived notifications when a node is disconnected.
> *Detecting disconnected nodes*
Node.monitor/1 lets you register and recived notifications when a node is disconnected.
You can also use :net_kernal.monitor_nodes. Each node periodically tries to check to see if they're connected to the other nodes.
### 12.1.2 Communicating between nodes
Node.spawn/2 recieves a node name and a lambda. It spawns a new process on the target node and runs the lambda in that process.
#### Listing 12.1 Spawning a process on another node
Use Node.spawn/2 to tell a process to run on another node.
> *Group leader process*
I noticed while doing this that the IO.puts was printing on the node I inputted Node.spawn in, instead of the one I told to run the lambda. All standard IO calls are forwarded to the *group leader*. Basically, it'll use the other nodes to run the behind the scenes stuff, but use the node you're obviously able to communicate with to print stuff for the user.

> *Avoid spawnign lamdas or sending them to different nodes*
### 12.1.3 Process discovery
> *How global registration works*

> *Recognizing remote processes*

**Groups of processes**
### 12.1.4 Links and monitors
### 12.1.5 Other distribution services
> *Message passing is the core distribution primitive*
## 12.2 Building a fault-tolerant cluster
### 12.2.1 Cluster design
> *Network Partitions*
### 12.2.2 The distributed to-do cache
#### Figure 12.2 Using a to-do cache
**Discovering to-do servers**
> *Process Registrations*
#### Listing 12.2 Global registration of to-do servers (todo_distributed/lib/todo/server.ex)
#### Listing 12.3 Discovering to-do servers(todo_distributed/lib/todo/server.ex)
#### Listing 12.4 Optimized process discovery (todo_distributed/lib/todo/cache.ex)
**Alternative discovery**
### 12.2.3 Implementing a replicated database
#### Figure 12.3 Replicating the database
#### Listing 12.5 Storing data on all nodes (todo_distributed/lib/todo/database.ex)
> *Always Provide Timeouts*
### 12.2.4 Testing the System
### 12.2.5 Detecting partitions
### 12.2.6 Highly available systems
 - *Responsiveness*
 - *Scalability*
 - *Fault-tolerance*
## 12.3 Network considerations
### 12.3.1 Node names
### 12.3.2 Cookies
### 12.3.3 Hidden nodes
### 12.3.4 Firewalls
#### Figure 12.4 Connecting to a remote node
> *Security*