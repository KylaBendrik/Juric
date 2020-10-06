# 11 Working with components
We are learning in this chapter about OTP-Apps. 
They will be helpful tools for making our systums reuseable.
The to-do list is getting upgraded to handle 3nd party 
libs and HTTP interface.  

## 11.1 OTP applications
OTP-Apps are a component that has mulitiple modules.
OTP-Apps can be started with a single function call 
and are easy to turn into an application as we will se in the next section.

### 11.1.1 Creating applications with the mix tool
Applications are an OTP-specific construct.
Using the mix tool is a great way to start an Application.
Mix creates the application resource flies nessisary to begin a project 
and also compilies it. The Mix project can also be run in iex. 
You as the progemer give it the nessisary information by making new files, giving 
it a name, writting tests, and filling out the mix.exs file.

### 11.1.2 The application behavior
The Module description starts the application.
To be able to work with Application, you must implement your own callback module
and define some callback functions.
The callback module must contain the start/2 function.

### 11.1.3 Starting the application
To start the application call Application.start/1.
When start is called the app interprets the contents of the file, verifies 
which dependent apps are started, and then calls back the apps start/2 function. 
You can’t start multiple instances of a single application much like a singleton.
Stopping an application terminates its top-level process.
Application.stop/1 stops only the specified application, leaving dependencies
(other applications) running.

### 11.1.4 Library applications
There’s no application callback module, which means there is no top-level
process to start but it still is a OTP process so it can start and stop.
This technique is used for library applications:
components that don’t need to create their own supervision tree. As the name indicates,
these are usually simpler libraries; a typical example is a JSON parser

### 11.1.5 Creating a to-do application
The to-do system runs a set of its own processes under a supervision tree, it
makes sense to turn it into a full-blown application.
Once you properly implement the application callback module,
the system can be automatically started as soon as you run iex -S mix.
Inside TodoCacheTest you needed to
manually start the supporting processes, such as cache.

#### Listing 11.1 Specifying appliction parameters (todo_app/mix.exs)
#### Listing 11.2 Implementing the application module (todo_app/lib/todo/application.ex)
#### Listing 11.3 Testing server_process (todo_app/test/todo_cache_test.exs)
### 11.1.6 The application folder structure
Mix environments: This is a compile-time option that can be used to affect the shape of the compiled code.
Mix projects use three environments: dev, test, and prod. These three environments
produce slight variations in the compiled code.

The compiled code structure: Because dev is the default environment, if you run mix compile or iex -S mix, you
get binaries in the _build/dev folder. In addition to your application, the lib folder contains your compile-time dependencies.
When you attempt to start the application, the generic
application behavior looks for the resource file in the load paths (the same paths that are
searched for compiled binaries).

## 11.2 Working with Dependencies


### 11.2.1 Adding a dependency
#### Listing 11.4 Adding an external dependency (todo_poolboy/mix.exs)
### 11.2.2 Adapting the pool
#### Listing 11.5 Starting a Poolboy-powered pool (todo_poolboy/lib/todo/database.ex)
#### Listing 11.6 Adapted operation functions (todo_poolboy/lib/todo/database.ex)
#### Listing 11.7 Adapted worker interface functions (todo_poolboy/lib/todo/database.ex)
### 11.2.3 Visualizing the system
#### Figure 11.1 Observing the application
## 11.3
### 11.3.1 Choosing dependencies
#### Listing 11.8 External dependencies for the web server (todo_web/mix.exs)
### 11.3.2 Starting the server
#### Listing 11.9 HTTP server specification (todo_web/lib/todo/web.ex)
#### Listing 11.10 Starting the HTTP server (todo_web/lib/todo/system.ex)
### 11.3.3 Handling requests
#### Listing 11.11 Setting up a route for add_entry (todo_web/lib/todo/web.ex)
#### Listing 11.12 Implementing the add_entry request (todo_web/lib/todo/web.ex)
#### Listing 11.13 Implementing the entries request (todo_web/lib/todo/web.ex)
### 11.3.4 Reasoning about the system
#### Figure 11.2 Requests are handled in separate processes
#### Figure 11.3 Independent simultaneous requests are handled concurrently
#### Figure 11.4 Using casts reduces the certainty of your reponses
#### Figure 11.5 Usinng calls promotes consistency but reduces the responsiveness of the system
#### Figure 11.6 Queueing the request and sending a notification about the status
## 11.4 Configuring applications
### 11.4.1 Application environment
#### Listing 11.14 Configuring the HTTP port (todo_env/config/config.exs)
#### Listing 11.15 Fetching the http_port setting (todo_env/lib/todo/web.ex)
### 11.4.2 Varying configuration
#### Listing 11.16 Mix environment-specific settings (todo_env/config/config.exs)
#### Listing 11.17 Overriding configuration (todo_env/config/test.exs)
### 11.4.3 Config script considerations
