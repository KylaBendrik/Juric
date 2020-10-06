# 11 Working with components
In this chapter, we are going to go from small applications to OTP-compliant applications and webservers, as well as learning about how to work with dependencies and how to configure applications.
## 11.1 OTP applications
And OTP aplication is able to be started with a single function call, even though it uses multiple modules and may depend on other systems.
### 11.1.1 Creating applications with the mix tool
The mix tool uses? creates? a file with the application's name and version, description, a list of modules in the application, dependencies, and an optional callback module. You can use the Application module to start and stop the application.
mix new hello_world --sup creates a folder with the minimum mix project skeleton. The --sup parameter makes the mix tool generate the callback module and start the empty supervisor.
### 11.1.2 The application behavior
HelloWorld.Application.start/2 is the function that will start the application. This starts the top-level process of the system, usually a supervisor, and returns a tuple of {:ok, pid} or {:error, reason}
### 11.1.3 Starting the application
You can start the application by calling Application.start(:hello_world), and it may return an error if the application is already running. This is extra info that you won't usually need to use, because mix automatically starts the application with its dependencies. You can't start multiple instances of a single application. You can stop using Application.stop(:hello_world). This will exit only the specified app, in a polite way.
### 11.1.4 Library applications
Library applications are components that don't need supervision trees. Usually simpler apps, such as a JSON parser. There's no callback module, so no top-level process to be started. It's only a toolbox that can be used by other applications. It doesn't do much on its own. ?
### 11.1.5 Creating a to-do application
In this section, we're going to take the OTP app we've been making and turn it into a full-blown application with a callback module and supervision tree. Simplified system startup! Yay!
#### Listing 11.1 Specifying appliction parameters (todo_app/mix.exs)
This listing shows the mix.exs file from our previous todo app, with some slight changes. Specifically, it adds line 17 to application/0 : 
```elixir
  mod: {Todo.Application, []}
```
This adds the callback module.
#### Listing 11.2 Implementing the application module (todo_app/lib/todo/application.ex)
We're now adding the callback module, creating a new file `application.ex`, defining one function: `start/2`
Now, all it takes to start the Todo app is to run iex.bat -S mix
(it also pops up memory_usage info, which is interesting)
#### Listing 11.3 Testing server_process (todo_app/test/todo_cache_test.exs)
The only change I'm seeing here, is that we no longer have to pass Todo.Cache.server_process the cache. Not sure why.
```elixir
  test "server_process" do
    {:ok, cache} = Todo.Cache.start()
    bob_pid = Todo.Cache.server_process(cache, "bob")

    assert bob_pid != Todo.Cache.server_process(cache, "alice")
    assert bob_pid == Todo.Cache.server_process(cache, "bob")
  end
```
Turns into
```elixir
  test "server_process" do
    bob_pid = Todo.Cache.server_process("bob")

    assert bob_pid != Todo.Cache.server_process("alice")
    assert bob_pid == Todo.Cache.server_process("bob")
  end
```
It says "inside Todo.CacheTest you needed to manually start the supporting processes, such aas cache. With the system turned into a proper OTP application, this is not the case anymore, as you can see..." But doesn't server_process still expect a `cache` argument?
### 11.1.6 The application folder structure
Mix projects use three environments: dev, test, and prod. You can specify an environment with `MIX_ENV`
- dev: 
  - in a version compiled for development, you may want extra debug logging
  - in most mix tasks, default environment is dev.
- prod:
  - in production, you don't want debug logging
  - To compile the code for prod, you can invoke `MIX_ENV=prod mix compile`
- test:
  - you may want a different database to prevent tests from polluting your dev database.
  - test is the default environment when you invoke `mix test`
## 11.2 Working with Dependencies
It's helpful to use third-party libraries, such as web frameworks, JSON parsers, or database drivers. In this section, we are going to clean up our implementation of database workers and use a proven process pool library instead.
### 11.2.1 Adding a dependency
We'll be working with the Poolboy library, which provides an implementation of a process pool. I created a new branch in my todo project named `poolboy`
#### Listing 11.4 Adding an external dependency (todo_poolboy/mix.exs)
In the `mix.exs` file, add `:poolboy` to the list of dependencies. There are probably example dependencies listed as comments in this function, but probably no real ones yet.
### 11.2.2 Adapting the pool
This part describes how to use the Poolboy toolkit to fit our needs. You use checkout and checkin to ask for and "return" a worker process, like a library book. This way, the manager can know which worker processes are being used at any time. This also lets the manager start a new worker process if one crashes.
#### Listing 11.5 Starting a Poolboy-powered pool (todo_poolboy/lib/todo/database.ex)
In the good old database file, we need to add a couple of lines telling the poolboy module to start working. The `child_spec/1` function changes dramatically.
:poolboy.child_spec/3 
1. ID of child
2. pool options:
  1. :name states that the manager process should be locally registered
  2. :worker_module option specifies the module that will power each worker process
  3. :size specifies the pool size (I used the @pool_size we made earlier. The book doesn't)
3. list of arguments passed to the start_link of each worker when they're being started.

For some reason, this is giving me some issues...It says that :poolboy.child_spec/3 breaks the contract. ?
#### Listing 11.6 Adapted operation functions (todo_poolboy/lib/todo/database.ex)
Here we change store/2 and get/1 to work with the new Poolboy interface
#### Listing 11.7 Adapted worker interface functions (todo_poolboy/lib/todo/database.ex)
Here we change the worker interface functions to work with the new Poolboy toolbox. Mostly, we get ride of the viatuple stuff.
### 11.2.3 Visualizing the system
We can visualize the application using the observer.
#### Figure 11.1 Observing the application
You can see the top level supervisor, the three children, and the to-do servers.
## 11.3 Building a web server
Time to create the HTTP interface. (a la Pragmatic Studio?). We're going to focus on only `entries` and `add-entry` requests, not including update or delete. I assume we'll work on those in the future, or will be expected to make those for the exercise.
### 11.3.1 Choosing dependencies
Unfortunately, we won't be touching Phoenix at this time. We won't be using Cowboy directly (I think we did in an earlier book?). We'll be using Plug. Plug uses "middleware modules" that can be injected into the request-handling pipe.
#### Listing 11.8 External dependencies for the web server (todo_web/mix.exs)
This one shows how to add `:cowboy` and `:plug` dependencies.
### 11.3.2 Starting the server
Use Plug.Adapters.Cowboy.child_spec/1 to start the server. It returns a child specification describing how to start the processes responsible for the HTTP server.
#### Listing 11.9 HTTP server specification (todo_web/lib/todo/web.ex)
Let's create a new module: Todo.Web
This will hold our server specification.
#### Listing 11.10 Starting the HTTP server (todo_web/lib/todo/system.ex)
Add .Web to the ever growing list of things to go in the supervision tree.
### 11.3.3 Handling requests
add_entry will be a POST request. The implementation of the request handler consists of three parts:
1. decode the input parameters
2. invoke the code that performs some action in the system
3. respond to the client

Phoenix is not your application

The HTTP server is an interface.
#### Listing 11.11 Setting up a route for add_entry (todo_web/lib/todo/web.ex)
Set up a route in Todo.Web. use Plug.Router
Below this listing, it explains the basics of how the Plug.Router macros work.
#### Listing 11.12 Implementing the add_entry request (todo_web/lib/todo/web.ex)
Fill in the add_entry request. The `conn` variable is generated by the post macro. The conn variable holds the connection.
#### Listing 11.13 Implementing the entries request (todo_web/lib/todo/web.ex)
Same story as last time, only now its a `get` request. So this time, instead of just returning `(200, "OK")`, we're returning `(200, formatted entries)`
### 11.3.4 Reasoning about the system
Explaining how an HTTP server works
#### Figure 11.2 Requests are handled in separate processes
Each request spawns a new process to talk to the Todo system. 
#### Figure 11.3 Independent simultaneous requests are handled concurrently
Independent requests won't block each other, but multiple requests on the same todo list are synchrnized, handled one at a time
#### Figure 11.4 Using casts reduces the certainty of your reponses
casts are fire-and-forget
call expects a response

If you use a cast, you will immediately get a success response, but it may not accurately reflect whether or not the outcome of the request was successful.
#### Figure 11.5 Using calls promotes consistency but reduces the responsiveness of the system
Using calls will be more accurate, but slower.
#### Figure 11.6 Queueing the request and sending a notification about the status
You can resolve some of the slowness by adding an intermidate process, queing the requests, so you can immediately reply that the request has been queued, but not slow the client down with a call. This isn't necessary for small applications. Only usefull for high-load operations.
## 11.4 Configuring applications
You can provide application environment values in config script files.
### 11.4.1 Application environment
Instead of hard-coding the port # in `web`, let's make assign it in the application environment.
#### Listing 11.14 Configuring the HTTP port (todo_env/config/config.exs)
We need to make a new folder (config) and a new file (config.exs), and put in these two lines. We can now use `Application.get_env/2` to get this environment variable. (How are these related to OS environment variables?)
#### Listing 11.15 Fetching the http_port setting (todo_env/lib/todo/web.ex)
`Use Application.get_env/2` to get the environment variable we set up earlier.
### 11.4.2 Varying configuration
If we keep it set up this way, we can't run tests if the system is running.
#### Listing 11.16 Mix environment-specific settings (todo_env/config/config.exs)
`import_config "#{Mix.env()}.exs"` imports the configuration specific to the current mix environmen. Mix.env() returns the environment as an atom (:dev, :test, :prod) (see section 11.1.6)
#### Listing 11.17 Overriding configuration (todo_env/config/test.exs)
Here we are setting up a different HTTP port for the test environment. Easily missed is the line that says we need to set up dev.exs and prod.exs. I'm not going to set up prod.exs at the moment, because I'm not sure if we're supposed to...
### 11.4.3 Config script considerations
The config files are evaluated before the project is compiled, so we're limited in what we can do in such scripts. You can't make runtime decisions in config scripts.
