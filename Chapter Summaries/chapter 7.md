# 7 Building a concurrent system
Elixir/Erlang systems are powered by a multitude of processes, many of which are stateful server processes.
Ultimate goal is to build a distributed HTTP server that can handle many end users who are simultaneously manipulating many to-do lists.
## 7.1 Working with the mix project
`mix new projectname` is how you generate a new blank elixir project
## 7.2 Managing multiple to-do lists
### 7.2.1 Implementing a cache
#### Listing 7.1 Cache initialization (todo_cache/lib/todo/cache.ex)
#### Listing 7.2 Handling the `server_process` request (todo_cache/lib/todo/cache.ex)
#### Listing 7.3 Interface functions (todo_cache/lib/todo/cache.ex)
### 7.2.2 Writing tests
#### Listing 7.4 Test file skeleton (todo_cache/test/todo_cache_test.exs)
#### Listing 7.5 Testing server_process (todo_cache/test/todo_cache_test.exs)
#### Listing 7.6 Testing to-do server operations (todo_cache/test/todo_cache_test.exs)
### 7.2.3 Analyzing process dependencies
#### Figure 7.1 Cooperation of processes
## 7.3 Persisting data
### 7.3.1 Encoding and persisting
#### Listing 7.7 Database process (persistable_todo_cache/lib/todo/database.ex)
### 7.3.2 Using the database
#### Listing 7.8 Starting the database (persistable_todo_cache/lib/todo/cache.ex)
#### Listing 7.9 Persisting the data (persistable_todo_cache/lib/todo/server.ex)
#### Listing 7.10 Reading data (persistable_todo_cache/lib/todo/server.ex)
### 7.3.3 Analyzing the system
#### Figure 7.2 Process dependencies
### 7.3.4 Addressing the process bottle neck
#### Figure 7.3 Handling requests concurrently
#### Figure 7.4 Pooling database operations
### 7.3.5 Exercise: pooling and synchronizing
## 7.4 Reasoning with processes