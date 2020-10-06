# 12 Building a distributed system
#### Figure 12.1 To to-do system as a cluster
## 12.1 Distribution primitives
### 12.1.1 Starting a cluster
> *Detecting disconnected nodes*
### 12.1.2 Communicating between nodes
#### Listing 12.1 Spawning a process on another node
> *Group leader process*

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