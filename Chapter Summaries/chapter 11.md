# 11 Working with components
## 11.1 OTP applications
### 11.1.1 Creating applications with the mix tool
### 11.1.2 The application behavior
### 11.1.3 Starting the application
### 11.1.4 Library applications
### 11.1.5 Creating a to-do application
#### Listing 11.1 Specifying appliction parameters (todo_app/mix.exs)
#### Listing 11.2 Implementing the application module (todo_app/lib/todo/application.ex)
#### Listing 11.3 Testing server_process (todo_app/test/todo_cache_test.exs)
### 11.1.6 The application folder structure
## 11.2 Working with Dependencies
### 11.2.1 Adding a dependency
#### Listing 11.4 Adding an external dependency (todo_poolboy/mix.exs)
### 11.2.2 Adapting the pool
#### Listing 11.5 Starting a Poolboy-powered pool (todo_poolboy/lib/todo/database.ex)
#### Listing 11.6 Adapted operation functions (todo_poolboy/lib/todo/database.ex)
#### Listing 11.7 Adapted worker interface functions (todo_poolboy/lib/todo/database.ex)
### 11.2.3 Visualizing the system
#### Figure 11.1 Observing the application
## 11.3 Building a web server
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
#### Figure 11.5 Using calls promotes consistency but reduces the responsiveness of the system
#### Figure 11.6 Queueing the request and sending a notification about the status
## 11.4 Configuring applications
### 11.4.1 Application environment
#### Listing 11.14 Configuring the HTTP port (todo_env/config/config.exs)
#### Listing 11.15 Fetching the http_port setting (todo_env/lib/todo/web.ex)
### 11.4.2 Varying configuration
#### Listing 11.16 Mix environment-specific settings (todo_env/config/config.exs)
#### Listing 11.17 Overriding configuration (todo_env/config/test.exs)
### 11.4.3 Config script considerations
