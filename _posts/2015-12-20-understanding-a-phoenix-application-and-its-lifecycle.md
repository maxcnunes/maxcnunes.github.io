---
layout: post
title: Understanding a Phoenix application and it's lifecycle
tags:
- elixir
- phoenix
---

> It is a post for beginners in Elixir and Phoenix. Has intention to help understanding how a Phoenix application works and a bit of its lifecycle.

I started the day thinking in write an chat application to apply what I have learned so far of Elixir. But even after reading the whole [Elixir - Getting Started](http://elixir-lang.org/getting-started/introduction.html) and [Phoenix - Up And Running](http://www.phoenixframework.org/docs/up-and-running). I was not really feeling prepared to write an application on my own.

> P.S. I didn't read the Phoenix framework documentation before start the application. I'm sure if I had read the docs, everything would makes sense. But I was just too excited to start coding :D.

So instead of writing a chat app from scratch. What we gonna do is understand how the [chat app](https://github.com/chrismccord/phoenix_chat_example) built by Chris McCord works.


Before understanding, lets see the application running:

  * Clone the repo [chrismccord/phoenix_chat_example](https://github.com/chrismccord/phoenix_chat_example), then cd to the new directory
  * Install dependencies with `mix deps.get`
  * (optional) Install npm dependencies to customize the ES6 js/Sass `npm install`
  * Start Phoenix router with `mix phoenix.server`

Now you can visit localhost:4000 from your browser.


### mix.exs

Contains the definition for all dependencies and configures the OTP application.

> Check out [elixir/Application](http://elixir-lang.org/docs/stable/elixir/Application.html) to understand what is an OTP application.  And the presentation [OTP Has Done It - by Nick DeMonner](https://www.youtube.com/watch?v=yBReonQlfL4) also gives an idea about it.

```elixir
defmodule Chat.Mixfile do
  use Mix.Project

  def project do
    [app: :chat,
     version: "0.0.1",
     elixir: "~> 1.0",
     elixirc_paths: ["lib", "web"],
     compilers: [:phoenix] ++ Mix.compilers,
     deps: deps]
  end

  # Configuration for the OTP application
  #
  # Type `mix help compile.app` for more information
  def application do
    [mod: {Chat, []},
     applications: [:phoenix, :phoenix_html, :cowboy, :logger]]
  end

  # Specifies your project dependencies
  #
  # Type `mix help deps` for examples and options
  defp deps do
    [{:phoenix, "~> 1.0.0"},
     {:phoenix_html, "~> 2.1"},
     {:phoenix_live_reload, "~> 1.0", only: :dev},
     {:phoenix_ecto, "~> 1.1"},
     {:postgrex, ">= 0.0.0"},
     {:cowboy, "~> 1.0"}]
  end
end
```

In the `application` function is defined the `Chat` as the startup module. And also is defined all applications your application depends on at runtime.


### lib/chat.ex

```elixir
defmodule Chat do
  use Application

  # See http://elixir-lang.org/docs/stable/elixir/Application.html
  # for more information on OTP Applications
  def start(_type, _args) do
    import Supervisor.Spec, warn: false

    children = [
      # Start the endpoint when the application starts
      supervisor(Chat.Endpoint, []),
      # Start the Ecto repository
      worker(Chat.Repo, []),
      # Here you could define other workers and supervisors as children
      # worker(Chat.Worker, [arg1, arg2, arg3]),
    ]

    # See http://elixir-lang.org/docs/stable/elixir/Supervisor.html
    # for other strategies and supported options
    opts = [strategy: :one_for_one, name: Chat.Supervisor]
    Supervisor.start_link(children, opts)
  end

  # Tell Phoenix to update the endpoint configuration
  # whenever the application is updated.
  def config_change(changed, _new, removed) do
    Chat.Endpoint.config_change(changed, removed)
    :ok
  end
end
```

In `lib/chat.ex` is defined the OTP Application. As we can see the `Chat.Endpoint` is started as `supervisor`. Which will start the endpoint when the application starts and it will be restarted in case it crashes.

And `Chat.Repo` is started as `worker`. Which will run the repository in a different process. Allowing this way to keep the state (e.g. connection pool) between different requests. Otherwise would be necessary establish a new DB connection for every request.

### lib/chat/endpoint.ex

```elixir
defmodule Chat.Endpoint do
  use Phoenix.Endpoint, otp_app: :chat

  # Requests coming at "/socket" path will be handled by
  # UserSocket (web/channels/user_socket.ex)
  socket "/socket", Chat.UserSocket


  # Serve at "/" the given assets from "priv/static" directory
  plug Plug.Static,
    at: "/", from: :chat,
    only: ~w(css images js favicon.ico robots.txt)

  # Code reloading will only work if the :code_reloader key of
  # the :phoenix application is set to true in your config file.
  if code_reloading? do
    socket "/phoenix/live_reload/socket", Phoenix.LiveReloader.Socket
    plug Phoenix.CodeReloader
    plug Phoenix.LiveReloader
  end

  # Log the requests
  plug Plug.Logger

  # Configure parsers
  plug Plug.Parsers,
    parsers: [:urlencoded, :multipart, :json],
    pass: ["*/*"],
    json_decoder: Poison

  plug Plug.MethodOverride
  plug Plug.Head

  plug Plug.Session,
    store: :cookie,
    key: "_chat_key",
    signing_salt: "LH6XmqGb",
    encryption_salt: "CIPZg4Qo"

  # Only after passing through all the previous Plug
  # the request will be handled by the Chat.Router (web/router.ex)
  plug Chat.Router
end
```

In `lib/chat/endpoint.ex` is used a lot `Plug`. It allows compose modules between web applications. With `Plug` is possible to change the request and response data through the connection lifecycle. It is comparable to a middleware in Node JS.

> Check out [understanding-plug](http://www.phoenixframework.org/docs/understanding-plug) and [plug project](https://github.com/elixir-lang/plug).

### web/router.ex

```elixir
defmodule Chat.Router do
  use Phoenix.Router

  pipeline :browser do
    plug :accepts, ["html"]
    plug :fetch_session
    plug :fetch_flash
    plug :protect_from_forgery
  end

  pipeline :api do
    plug :accepts, ["json"]
  end

  scope "/", Chat do
    pipe_through :browser # Use the default browser stack

    get "/", PageController, :index
  end
end
```

In `Chat.Router` we see the definition of `pipeline`. Which is a simple way to pipe a series of plug before passing the request ahead to a controller. That can be used for different type of requests. For example: an API request must be handled differently of a browser (page) request.

> Once a request arrives at the Phoenix router, it performs a series of transformations through pipelines until the request is dispatched to a desired end-point.

> Such transformations are defined via plugs, as defined in the Plug specification. Once a pipeline is defined, it can be piped through per scope.

> http://hexdocs.pm/phoenix/Phoenix.Router.html

### web/controllers/page_catroller.ex

```elixir
defmodule Chat.PageController do
  use Chat.Web, :controller

  def index(conn, _params) do
    render conn, "index.html"
  end
end
```

After a request passing through all previous plug. It will be handled by the controller. For instance a `GET /` will respond with `index.html` page.

### channels/user_socket.ex

As we saw before in the `Chat.Endpoint` the socket connections will be handled by the `Chat.UserSocket`.

```elixir
defmodule Chat.UserSocket do
  use Phoenix.Socket

  channel "rooms:*", Chat.RoomChannel

  transport :websocket, Phoenix.Transports.WebSocket
  transport :longpoll, Phoenix.Transports.LongPoll

  def connect(_params, socket) do
    {:ok, socket}
  end

  def id(_socket), do: nil
end
```

Basically the `Chat.UserSocket` creates a channel for topics matching `rooms:*`. And add support for both web socket and log pool connections.

### channels/room_socket.ex

```elixir
defmodule Chat.RoomChannel do
  use Phoenix.Channel
  require Logger

  @doc """
  Authorize socket to subscribe and broadcast events on this channel & topic

  Possible Return Values

  `{:ok, socket}` to authorize subscription for channel for requested topic

  `:ignore` to deny subscription/broadcast on this channel
  for the requested topic
  """
  def join("rooms:lobby", message, socket) do
    # Exit signals arriving to a process are converted to {'EXIT', From, Reason} messages,
    # which can be received as ordinary messages
    Process.flag(:trap_exit, true)
    :timer.send_interval(5000, :ping)
    send(self, {:after_join, message})

    {:ok, socket}
  end

  def join("rooms:" <> _private_subtopic, _message, _socket) do
    {:error, %{reason: "unauthorized"}}
  end

  def handle_info({:after_join, msg}, socket) do
    broadcast! socket, "user:entered", %{user: msg["user"]}
    push socket, "join", %{status: "connected"}
    {:noreply, socket}
  end
  def handle_info(:ping, socket) do
    push socket, "new:msg", %{user: "SYSTEM", body: "ping"}
    {:noreply, socket}
  end

  def terminate(reason, _socket) do
    Logger.debug"> leave #{inspect reason}"
    :ok
  end

  def handle_in("new:msg", msg, socket) do
    broadcast! socket, "new:msg", %{user: msg["user"], body: msg["body"]}
    {:reply, {:ok, %{msg: msg["body"]}}, assign(socket, :user, msg["user"])}
  end
end
```

Pretty simple, it handles:
  * new users join the channel
  * broadcast new users in the chat
  * send ping messages
  * broadcast a user's message

> Check out [Phoenix.Channel](http://hexdocs.pm/phoenix/Phoenix.Channel.html) for a further explanation.

The rest is just a HTML page loading a CSS script (source in Sass) and a Javascript script (source in ES6) which consumes the socket provided by the Chat channel.

> P.S. I have just started learning Elixir and Phoenix. Let me know if I had misunderstood something.
