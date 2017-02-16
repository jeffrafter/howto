# Building a Quiz Game in Phoenix

> At the bottom of this guide are a set of links that represent how I learned how all of this works.

## Requirements: 

* Erlang (19+)
* Elixir (1.4+)
* Hex (0.15.0+)
* Postgres (9.2+)
* Node (5.3.0+)

### Erlang and Elixir (and mix)

Use homebrew to install or upgrade Erlang and Elixir.

```bash
brew install erlang elixir
```

Next get the latest version of Hex:

```bash
mix local.hex
mix archive.install https://github.com/phoenixframework/archives/raw/master/phoenix_new.ez

```



### Postgres

You can check your current installation with:

```bash
psql --version
```

Again, use homebrew to install or upgrade:

```bash
brew install postgres
brew services start postgresql
initdb /usr/local/var/postgres
/usr/local/Cellar/postgresql/<version>/bin/createuser -s postgres
```

### Node

You can check your current installation with:

```bash
node --version
```

Again, use homebrew to install or upgrade:

```bash
brew install node
```

## Getting started

Create the project:

```bash
mix phoenix.new quiz_game
```

You should see the following output:

```bash
* creating quiz_game/config/config.exs
* creating quiz_game/config/dev.exs
* creating quiz_game/config/prod.exs
* creating quiz_game/config/prod.secret.exs
* creating quiz_game/config/test.exs
* creating quiz_game/lib/quiz_game.ex
* creating quiz_game/lib/quiz_game/endpoint.ex
* creating quiz_game/test/views/error_view_test.exs
* creating quiz_game/test/support/conn_case.ex
* creating quiz_game/test/support/channel_case.ex
* creating quiz_game/test/test_helper.exs
* creating quiz_game/web/channels/user_socket.ex
* creating quiz_game/web/router.ex
* creating quiz_game/web/views/error_view.ex
* creating quiz_game/web/web.ex
* creating quiz_game/mix.exs
* creating quiz_game/README.md
* creating quiz_game/web/gettext.ex
* creating quiz_game/priv/gettext/errors.pot
* creating quiz_game/priv/gettext/en/LC_MESSAGES/errors.po
* creating quiz_game/web/views/error_helpers.ex
* creating quiz_game/lib/quiz_game/repo.ex
* creating quiz_game/test/support/model_case.ex
* creating quiz_game/priv/repo/seeds.exs
* creating quiz_game/.gitignore
* creating quiz_game/brunch-config.js
* creating quiz_game/package.json
* creating quiz_game/web/static/css/app.css
* creating quiz_game/web/static/css/phoenix.css
* creating quiz_game/web/static/js/app.js
* creating quiz_game/web/static/js/socket.js
* creating quiz_game/web/static/assets/robots.txt
* creating quiz_game/web/static/assets/images/phoenix.png
* creating quiz_game/web/static/assets/favicon.ico
* creating quiz_game/test/controllers/page_controller_test.exs
* creating quiz_game/test/views/layout_view_test.exs
* creating quiz_game/test/views/page_view_test.exs
* creating quiz_game/web/controllers/page_controller.ex
* creating quiz_game/web/templates/layout/app.html.eex
* creating quiz_game/web/templates/page/index.html.eex
* creating quiz_game/web/views/layout_view.ex
* creating quiz_game/web/views/page_view.ex

Fetch and install dependencies? [Yn] y
* running mix deps.get
* running npm install && node node_modules/brunch/bin/brunch build

We are all set! Run your Phoenix application:

    $ cd quiz_game
    $ mix phoenix.server

You can also run your app inside IEx (Interactive Elixir) as:

    $ iex -S mix phoenix.server

Before moving on, configure your database in config/dev.exs and run:

    $ mix ecto.create
```

Change into the newly created project folder:

```bash
cd quiz_game
```

Get setup (you may need to open `config/dev.exs` and remove or change the postgres user password first):

```bash
mix ecto.create
```

At this point you can run your project and check it out:

```bash
mix phoenix.server
```

### Git

Now that we have the basics of our server, let's setup the basics of our version control. Create a new file `.gitignore`:

```bash
# App artifacts
/_build
/db
/deps
/*.ez
doc

# Generated on crash by the VM
erl_crash.dump

# The config/prod.secret.exs file by default contains sensitive
# data and you should not commit it into version control.
#
# Alternatively, you may comment the line below and commit the
# secrets file as long as you replace its contents by environment
# variables.
/config/prod.secret.exs

# Extra
/priv/static/
/priv/gettext/
node_modules/
/rel

# Keep it secret!
.env
```

Now setup the repository:

```bash
git init
git add .
git commit -a -m "Initial commit"
```

# Game outline

Our game will have multiple players trying to answer challenging questions. There will be 5 questions per game, then we'll declare a winner.

* Welcome
* Create a Game (host-only)
* Join a Game (player-only)
* Game Start
* Game Turn
  - Question
  - Buzz in
  - Answer
* Check for winner

In order to accomplish this we'll need a number of `structs`:

* Game
* Player
* Turn
* Question

## Welcome

Players need a place to view what is happening, we'll start with a very basic welcome screen. By default, Phoenix creates a `PageController`:

```elixir
defmodule QuizGame.PageController do
  use QuizGame.Web, :controller

  def index(conn, _params) do
    render conn, "index.html"
  end
end
```

Let's modify this to auto-generate a new player id:

```elixir
defmodule QuizGame.PageController do
  @moduledoc """
  A basic controller to show the primary landing pages.
  """
  use QuizGame.Web, :controller

  @doc """
  Players view the welcome screen and every time they do, we generate
  a new random player id that will be used to identify them when they
  join and play games.
  """
  def index(conn, _params) do
    player_id = conn.cookies["player"] || QuizGame.generate_player_id

    conn
    |> put_resp_cookie("player", player_id, max_age: 24*60*60)
    |> render("index.html", id: player_id)
  end
```

We'll assume for now that only players will be visiting the welcome page. The host (an AppleTV application) will be creating games via a separate interface. Notice that we are storing the `id` in a cookie (that lasts for 1 day; or 24 hours * 60 minutes * 60 seconds). This will allow us to build some basic reconnect logic later and will keep the player's `id` consistent when code-reloading updates all of our pages and severs our socket connections.

Let's make the `generate_player_id` function:


```elixir
# lib/quiz_game.ex

defmodule QuizGame do
  use Application

  @id_length Application.get_env(:quiz_game, :id_length)
  # ...
  # ...

  @doc """
  Each player needs a unique id, this creates a random one
  """
  def generate_player_id do
    @id_length
    |> :crypto.strong_rand_bytes
    |> Base.url_encode64()
    |> binary_part(0, @id_length)
  end

  # ...
end
```

Our `generate_player_id` function simply generates a random code a specific length. In a production application this would be woefully inadequet. We would likely use a JWT here (using `Joken`) instead or perhaps do an OAuth layer combined with an existing authentication system. At the very least we would store these somewhere (perhaps in a `GenStage`, or Redis or Memcache) and check for duplicates. For now, it is random numbers. 

Great. Now we need to use this id. To keep things simple we'll bind this to a global JavaScript variable in the view (in a production level app you might have a full page framework responsible for managing the variables). Replace the contents of `web/templates/page/index.html.eex` with:

```html
<script>
  window.playerId = '<%= @id %>';
</script>

<div class="jumbotron">
  <h2 id="welcome" class="animated pulse">Welcome to<br>Quiz Game!</h2>
</div>

<div class="center">
  <a class="btn" href="/host">Host a game</a>
  <span class="or">or</span>
  <a class="btn" href="/lobby">Join a game</a>
</div>

<div class="meta">
  Player id: <%= @id %>
</div>
```

Eventually we'll want to consider player accounts and authentication and the ability to re-use an existing `id`. For now this should work great.

#### A quick stylistic detour

Without going overboard, let's tweak some styles. In `web/templates/layout/app.html.eex` we can remove some boilerplate code, set the title, and add some more fun fonts:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <meta name="description" content="">
    <meta name="author" content="">

    <title>Quiz Game!</title>
    <link href="https://fonts.googleapis.com/css?family=Fontdiner+Swanky|Londrina+Solid|Open+Sans" rel="stylesheet">
    <link rel="stylesheet" href="<%= static_path(@conn, "/css/app.css") %>">
  </head>
  <body>
    <div class="container">
      <header class="header">
        <nav role="navigation">
          <ul class="nav nav-pills pull-right">
          </ul>
        </nav>
      </header>

      <p class="alert alert-info" role="alert"><%= get_flash(@conn, :info) %></p>
      <p class="alert alert-danger" role="alert"><%= get_flash(@conn, :error) %></p>

      <main role="main">
        <%= render @view_module, @view_template, assigns %>
      </main>

    </div> <!-- /container -->
    <script src="<%= static_path(@conn, "/js/app.js") %>"></script>
  </body>
</html>
```

I've picked two fun fonts from Google Fonts. In `web/static/css/app.css` add:

```css
/* This file is for your main application css. */

/*
font-family: 'Fontdiner Swanky', cursive; <- Welcome font; quirky, heavier, better apos
font-family: 'Londrina Solid', cursive; <- Question and answer font (check out the sketch version)
*/

body {
  background: purple;
  color: white;
  min-height: 100%;
}

.container {
  min-height: 100%;
  width: 100%;
  max-width: 100%;
}

.jumbotron {
  background: none;
}

.header {
  margin-bottom: 30px;
  min-height: 92px;
  border-bottom: none;
}

.header ul {
  margin-top: 0;
}

.header ul li {
  background: white;
}

.center {
  text-align: center;
}

main {
  height: 100%;
  width: 100%;
  margin: 0;
  padding: 0;
}

h2#welcome {
  font-size: 128px;
  font-family: 'Fontdiner Swanky', cursive;
}

.meta {
  position: absolute;
  left: 0;
  bottom: 0;
  padding-bottom: 20px;
  text-transform: uppercase;
  width: 100%;
  text-align: center;
}

.or {
  margin: 26px;
  font-size: 26px;
  font-family: 'Londrina Solid', cursive;
}

.btn {
  background-color: white;
  font-family: 'Londrina Solid', cursive;
  color: purple;
  font-size:36px;
}

.btn.orange {
  background-color: orange;
  color: white;
}

/*
 * Animation
*/

.animated {
  -webkit-animation-duration: 1s;
  animation-duration: 1s;
  -webkit-animation-fill-mode: both;
  animation-fill-mode: both;
  -webkit-animation-iteration-count: infinite;
  animation-iteration-count: infinite;
}

/* https://webdesignerhut.com/5-animations-using-css-keyframes/ */

/*
 * Pulse animation
*/

@-webkit-keyframes pulse {
  0% { -webkit-transform: scale(1); }
  50% { -webkit-transform: scale(1.1); }
  100% { -webkit-transform: scale(1); }
}
@keyframes pulse {
  0% { transform: scale(1); }
  50% { transform: scale(1.1); }
  100% { transform: scale(1); }
}
.pulse {
  -webkit-animation-name: pulse;
  animation-name: pulse;
}
```

Most of the code in this file is intended to hide the default headers, introduce some fun fonts and provide some basic animation via CSS transitions. Make sure your server is running (`mix phoenix.server`) and check it out:

![](https://rpl.cat/uploads/oT5FJ2iRMT9459aDMyTP5HInWLjuQoilugZLh93QVq4/public.png)

## Games and Processes

Before we go much further with the front-end we'll need to build out the back-end to manage the current games and their state. This gets a little confusing, mostly because there are multiple classes and multiple processes all talking to each other. Until we have written all of the code we won't really be able to test how it integrates. By the end, though, the separation starts to make sense. Here is a quick list of the classes involved in the communication:

1. **Game** - This class is a `GenServer` it manages getting and setting the state of game from a child process (one for each game)
2. **Game.Supervisor** - This class is a `Supervisor` and is in charge of all of the child `Game` servers (and their child processes)
3. **UserSocket** - Basic communication class for working with web sockets
4. **LobbyChannel** - A channel which allows connected players to see the available games
5. **HostChannel** - A channel which allows hosts to start new games and perform host actions for the games they start (like ask questions and announce the winner)
6. **PlayerChannel** - A channel which allows players to join games, submit answers and see updates as the game changes state

Lets start with the backend pieces.

### Game Supervisor

By default, Elixir and Phoenix don't store state. But what if you need to store state? Often this is done in a separate process (see also [http://dantswain.herokuapp.com/blog/2015/01/06/storing-state-in-elixir-with-processes/](http://dantswain.herokuapp.com/blog/2015/01/06/storing-state-in-elixir-with-processes/)). 

We need to store the state of each game that is in progress and which players are connected. Becuase of this we are going to make each game a separate child process. In order to manage all of the processes for the many simultaneous Quizzes happening we'll need a `Supervisor` (see also [https://hexdocs.pm/elixir/Supervisor.html](https://hexdocs.pm/elixir/Supervisor.html_)). It will be responsible for starting new `Game` processes and communicating with them, getting and setting their state and handling any errors along the way.

Create a new file called `lib/quiz_game/game/supervisor.ex`:

```elixir
defmodule QuizGame.Game.Supervisor do
  use Supervisor

  @doc """
  We define how new processes are started and name them based on the
  module (QuizGame.Game.Supervisor) name so that they can be accessed
  without using pids later. We'll be using a simple_one_for_one
  supervision strategy so we'll only ever have one name and one kind
  of child that is supervised.
  """
  def init(_) do
    children = [
      worker(QuizGame.Game, [], restart: :temporary)
    ]

    supervise(children, strategy: :simple_one_for_one, name: __MODULE__)
  end

  def start_link, do: Supervisor.start_link(__MODULE__, [], name: __MODULE__)

  def create_game(id) do
    Supervisor.start_child(__MODULE__, [id])
  end

  @doc """
  Leverage the supervisor to inspect all child processes and fetch data
  from them.
  """
  def games do
    __MODULE__
    |> Supervisor.which_children
    |> Enum.map(&data/1)
  end

  defp data({_id, pid, _type, _modules}) do
    pid
    |> GenServer.call(:get_data)
  end
end
```

Now that we have our Supervisor we need our application to start it. In `lib/quiz_game.ex` you need to add it to the list of children:

```elixir
defmodule QuizGame do
  # ...
  
  def start(_type, _args) do
    import Supervisor.Spec

    # Define workers and child supervisors to be supervised
    children = [
      # ...
      supervisor(QuizGame.Game.Supervisor, []),
    ]

    # ...
  end
 
  # ...
end
```

With the Supervisor complete, we now need some processes for it to supervise. These will all be `Game` processes. 

### Game

Our `Game` needs to hold state information about the questions, answers, score, and players. We'll need to manage this as a separate process, and will therefore use a `GenServer`. The `GenServer` pattern abstracts away starting a process, managing it and getting and setting state information. Our `Game` module will implement both sides of the that communication (the client and the server).

Create a new file: `lib/quiz_game/game.ex`

```elixir
defmodule QuizGame.Game do
  use GenServer

  @typedef """
  The basic state of the game
  """
  defstruct [
    id: nil,
    host: nil,
    questions: [],
    players: [],
    turns: [],
    winner: nil,
    state: nil,
  ]

  # Within our Phoenix application we can call this function and when 
  # we do the GenServer will look up the associated process for 
  # our game and send the :join message. The process is also running
  # this class and will receive a corresponding handle_call which 
  # we handle lower in this module.
  def join(id, player) do
    GenServer.call(via_tuple(id), {:join, player})
  end

  def init(id) do
    state = %__MODULE__{
      id: id,
      host: nil,
      questions: QuizGame.Questions.questions,
      players: [],
      turns: [],
      winner: nil,
      state: :waiting
    }

    {:ok, state}
  end

  def start_link(id) do
    GenServer.start_link(__MODULE__, id, name: via_tuple(id))
  end

  def handle_call({:join, player}, _from, game) do
    {:reply, {:ok, game.state}, %{game | players: [player | game.players]}}
  end

  def handle_call(:get_data, _from, game) do
    {:reply, game, game}
  end

  defp via_tuple(id) do
    {:via, Registry, {:game_registry, id}}
  end
end
```

Our `Game` has a couple of additional features. When it is created it generates a set of questions for the whole game and adds this to the returned game struct. It can also return the data when requested because it responds to a `handle_call` for `:get_data`. 

We'll add quite a bit more logic into this module as we implement our game logic. 

From within our primary Phoenix application (specifically the `PlayerChannel`) we will call `join/2`. This will simply fire off a message which the `GenServer` will pass to our running child process. That child process will be running the same `Game` class and will receive the message in `handle_call`. We have a matching `def` for `{:join, player}`. When the call is handled a `:reply` will be returned with a new version of the game state.

The `GenServer` knows how to find the child process using a "Via Tuple". Generally, there are three ways that a `GenServer` can find their associated child processes:

1. _by `pid`_ - this works great, but if the process gets restarted the `pid` will change and then things get lost. It is the most basic strategy and breaks down
2. _by name_ - as we saw with earlier you can name a process. However if you have more than one instance (for example, we will have many instances of `Game`) then you need to append something to make the names unique. Unfortunately the name is an atom, and those are not collected... so if you start a large number of short lived child processes you can run out of memory.
3. _by Registry_ - a `Registry` (new as of Elixir 1.3) offers the best of both worlds, it watches and manages changes to the `pid` for a process and allows you to specify a name and a key. This is done using a "Via Tuple" which looks like: `{:via, Registry, {:game_registry, id}}`. You can name the registry anything you like.

Our `Game` relies on a `Registry` to look up the associated processes. In order for this work we need to do two things: we have to register the process using the "Via" tuple when we start it and we have to make sure our application is managing the Registry itself (which is, you guessed it, another process). Within the `Game` class when we start a child process we use our `via_tuple` helper to generate the registry name:

```elixir
def start_link(id) do
  GenServer.start_link(__MODULE__, id, name: via_tuple(id))
end
```

All that's left is to add the registry to our primary supervisor in `lib/quiz_game.ex`:

```elixir
defmodule QuizGame do
  use Application

  # ...

  def start(_type, _args) do
    import Supervisor.Spec

    # Define workers and child supervisors to be supervised
    children = [
      supervisor(QuizGame.Repo, []),
      supervisor(QuizGame.Endpoint, []),
      supervisor(QuizGame.Game.Supervisor, []),
      supervisor(Registry, [:unique, :game_registry]),
    ]

    opts = [strategy: :one_for_one, name: QuizGame.Supervisor]

    Supervisor.start_link(children, opts)
  end
  
  # ...
end
```

Notice that we have used the same name for our Registry that we used when generating our "Via" tuple: `:game_registry`. 

#### Questions

Any decent quiz game needs quiz questions. In our case I found a small handful of questions and answers and modified them so that we can use for our Quiz Game. Add a new file `lib/quiz_game/questions.ex`:

```elixir
defmodule QuizGame.Questions do
  def questions do
    # http://www.quiz-questions.net/literature.php
    [
      ["Who killed the Minotaur?", "Theseus"],
      ["Who is the giant with 100 eyes according to Greek mythology?", "Argus"],
      ["What is the name of the winged horse in Greek mythology?", "Pegasus"],
      ["On which island did Ernest Hemingway stay and write?", "Cuba"],
      ["In which country did Shakespeare's Hamlet live?", "Denmark"],
      ["Who was the wife of Othello?", "Desdemona"],
      ["Who was the imaginary love of Don Quixote?", "Dulcinea"],
      ["Which book did Mary Shelley write when she was 19?", "Frankenstein"],
      ["Who said 'I think therefore I am?'", "Descartes"],
    ]
  end
end
```

## Sockets, Channels and Communications

When you created your application using `mix phoenix.new quiz_game`, Phoenix generated a user socket module for your application. All of our communication (hosts, players, games) will use the default user socket. Phoenix allows us to build custom *channels* on top of base socket. Ultimately, our user socket will have three channels:

1. _Lobby_ - This will list the available games
2. _Host_ - Hosts (in our case the AppleTV application) will send and receive game messages on this channel
3. _Player_ - Players will send and receive game messages on this channel

Let's start with the Lobby.

### Lobby

The lobby is a place where you can see all of the current games and players and you can join a game (or be matched into one, eventually). To make this, change `web/channels/user_socket.ex` as follows:

```elixir
defmodule QuizGame.UserSocket do
  use Phoenix.Socket

  ## Channels
  channel "lobby", QuizGame.LobbyChannel

  ## Transports
  transport :websocket, Phoenix.Transports.WebSocket

  # We expect the socket to connect with an id
  def connect(%{"id" => id}, socket) do
    {:ok, assign(socket, :id, id)}
  end

  # If you try to connect with empty params, it is an error
  def connect(_, _socket), do: :error

  # By specifying an id we can broadcast to this socket by id later
  def id(socket), do: "users_socket:#{socket.assigns.id}"
end
```

Out Lobby is simple, we expect a player to connect with their id and we setup a `LobbyChannel` to handle the interactions. For now, the lobby is really just an updating list of games. Create a new file `web/channels/lobby_channel.ex`:

```elixir
defmodule QuizGame.LobbyChannel do
  use QuizGame.Web, :channel

  @doc """
  When you join the lobby, we'll send back a list of the current games
  """
  def join("lobby", _msg, socket) do
    {:ok, games, socket}
  end

  @doc """
  If you send a `games` message, we'll reply with the current list of games
  """
  def handle_in("games", _params, socket) do
    {:reply, {:ok, games}, socket}
  end

  @doc """
  Send a list of the current games to all sockets listening on the lobby channel
  """
  def broadcast_current_games do
    QuizGame.Endpoint.broadcast("lobby", "games", games)
  end

  defp games do
    %{games: QuizGame.Game.Supervisor.games}
  end
end
```

The lobby channel handles a single incoming message `games` and can broadcast a single message to all clients. 

#### Lobby Page

Users will want a specific page where they can see available games, invite friends and be invited. We've already included a link to a `/lobby` endpoint on our Welcome page. To make that work, start by adding the route to the router in `web/router.ex`:

```elixir
defmodule QuizGame.Router do
  # ...
  
  scope "/", QuizGame do
    pipe_through :browser # Use the default browser stack

    get "/", PageController, :index
    get "/lobby", PageController, :lobby
  end
  
  # ...
end
```

Next, we'll need to add the matching action in `web/controllers/page_controller.ex`:

```elixir
defmodule QuizGame.PageController do
  # ...
  
  @doc """
  Available games
  """
  def lobby(conn, _params) do
    render(conn, "lobby.html", id: conn.cookies["player"])
  end

  # ...
end

```

This action assumes there is a template called `lobby.html`, you should create that now at `web/templates/page/lobby.html.eex`:


```html
<script>
  window.playerId = '<%= @id %>';
</script>

<div class="jumbotron">
  <h2 id="welcome" class="animated pulse">Join a game!</h2>
</div>

<div style="text-align:center">
  <ul id="games">
  </ul>
  <a class="btn" id="update" href="#">Update</a>
</div>

<div class="meta">
  Player id: <%= @id %>
</div>
```

This page is very similar to the Welcome page. We output the player id to a global `window` variable so that it is broadly accessible. 

#### Lobby JavaScript

We've skipped a deep dive into React or Elm and gone with basic JavaScript. For the main work of the lobby create a new file: 

```javascript
// We'll need a socket, luckily Phoenix provides a lot of the plumbing for us
import {Socket} from 'phoenix'

let Lobby = {
  channel: null,

  init() {
    Lobby.connect()
    Lobby.attachEvents()
  },

  connect() {
    // To connect we need to create a new socket and pass our unique id
    // By default, Phoenix has setup a /socket route for us
    let socket = new Socket("/socket", {params: {id: window.playerId}})
    socket.connect()

    // Connect to our lobby channel and call the join method
    let channel = socket.channel('lobby')

    // Join fulfills promises on completion
    channel.join()
      .receive('ok', reply => {
        // If we successfully joined, we should have received a current 
        // list of games to display
        Lobby.renderGames(reply.games)
      })
      .receive('error', reply => {
        // It is not the best user experience to just log errors...
        console.log(`Sorry, you can't join because ${reply.reason}`)
      })

    // If we receive a push message with the key "games" we are getting 
    // an update from the server about a change in the list of available
    // games and need to update the display
    channel.on('games', message => {
      Lobby.renderGames(message.games)
    })

    // We'll want to save this for later
    Lobby.channel = channel
  },

  renderGames(games) {
    // Pretty basic render function, maps each game object in the games message
    let ul = document.getElementById('games')
    ul.innerHTML = `
      <ul>
        ${games.map(game => `<li><a href="/play?id=${game.id}" class="btn orange">Join ${game.id}</a></li>`)}
      </ul>
    `
  },

  attachEvents() {
    let button = document.getElementById("update")
    button.onclick = Lobby.fetchGames
  },

  fetchGames(e) {
    e.preventDefault()
    e.stopPropagation()

    // When the user clicks update we'll send a "games" message to the
    // server indicating that we want to receive an updated list of games
    // The name of this message is coincidentally the same as our earlier
    // listener, the two are completely separate. We've named them the same
    // because they accomplish the same goal
    Lobby.channel.push('games', {})
      .receive('ok', reply => {
        Lobby.renderGames(reply.games)
      })
      .receive('error', reply => {
        console.log(`Sorry, you can't fetch games because ${reply.reason}`)
      })
  },

}
export default Lobby
```

Unfortunately, our `lobby.js` is never used. By default, Phoenix bundles up all of your JavaScript (using Brunch) and executes the code in the top-level `app.js`. Open `web/static/js/app.js` and add:

```javascript
import Lobby from "./lobby.js"

if (window.location.pathname === '/lobby') {
  Lobby.init()
}
```

This is obviously a little hacky, but should initialize our new `Lobby` class only when the current path is `/lobby`. 

At this point you should be able to go to the Welcome page and then click "Join a Game" which will take you to the lobby. 

![](https://rpl.cat/uploads/xU1uK5bjlh4Ov9j4IR7GxRIQ1fzdWjMo33ru9PGXSGM/public.png)


Amazing! But wait. There are no games available. Even if you click update there are no games! At this point there is no way for anyone to start a new game for us to join. Let's move on to the host.

### Host

The "Host" of the game is like a game show host. They decide when the game starts and stops and they are in charge of asking the questons. They also inject some witty banter to keep things fun. Our long-term plan is to make the host an Apple TV application so we can play our Quiz Game in our living room with friends. To keep things simple for now we'll start with a host page similar to our lobby page. 

Like the lobby, the host will communicate from a web browser to our Phoenix application via the user socket. To make that happen we need to declare it in `web/channels/user_socket.ex`:

```elixir
defmodule QuizGame.UserSocket do
  use Phoenix.Socket

  ## Channels
  channel "lobby", QuizGame.LobbyChannel
  channel "host:*", QuizGame.HostChannel

  # ...
end
```

Notice that the channel declaration for our host is `host:*`. When we built the lobby we wanted all users and sockets joining the same channel. This meant we could broadcast games to all of the connected users at once. For our host it is a little different, we might have multiple games going on at the same time. Because of this we added a wildcard so that we can add multiple host channels at a time; each with a unique name.

#### Host Channel

The Host will eventually have a lot of responsibility. Initially though, the host will need to:

* Join a channel
* Create a new game
* Tell all of the users waiting in the lobby about the new game
* Update the host web browser with the current game state

Cretae a new file `web/channels/host_channel.ex`:

The complete host channel looks like:

```elixir
defmodule QuizGame.HostChannel do
  use QuizGame.Web, :channel

  # When the host joins, check to see if the game is already
  # running. If it is, that is our game. If not, create a
  # new one. If the game is created, tell everyone in the lobby.
  def join("host:" <> id, _msg, socket) do
    pid =
      case QuizGame.Game.Supervisor.create_game(id) do
        {:ok, pid} -> pid
        {:error, {:already_started, pid}} -> pid
      end

    game_data = GenServer.call(pid, :get_data)

    status = game_data.state

    socket =
      socket
      |> assign(:game_id, id)
      |> assign(:status, status)

    send self(), {:after_join, status}

    {:ok, game_data, socket}
  end

  def handle_info({:after_join, _status}, socket) do
    QuizGame.LobbyChannel.broadcast_current_games

    {:noreply, socket}
  end
end
```
Let's break this down. The host needs be able to join the channel so we start with `join` method:

```elixir
  def join("host:" <> id, _msg, socket) do 
    # ...
  end
```

This method will match any join message that starts with `host:` and will grab what comes after the `:` into an `id` param. 

Within the `join` method we need to start the game. To do that we need to ask our `Supervisor` to create a new game process for the `id`. It's possible that a game has already been started for the specified `id` in which case the `Supervisor` will just respond with that `pid`. 

```elixir
    pid =
      case QuizGame.Game.Supervisor.create_game(id) do
        {:ok, pid} -> pid
        {:error, {:already_started, pid}} -> pid
      end
```

Why would the game already be started? Sockets can easily become disconnected because of a spotty wifi or mobile connection. Additionally, if the user refreshes the web page the socket will immediately disconnect and reconnect. We could ignore this case and just end the game if the host disconnects, but it is a much better experience to handle it gracefully.

> A `pid` is a process id. Every process your computer starts has a unique process id. Elixir uses the process id to communicate between processes. If a process gets restarted, the `pid` will change, but the `Supervisor` and `Registry` we setup keep track of process changes.

Next we need to grab the current state of the game so we can send if back to the host:

```elixir
    game_data = GenServer.call(pid, :get_data)
    status = game_data.state
```

We assign the `id` and the `status` to the new host socket in case we need them later:

```elixir
    socket =
      socket
      |> assign(:game_id, id)
      |> assign(:status, status)
```

We wanted the host to tell users about the new game but instead of doing that directly in our `join` method (which we want to return quickly) we post a message to handle that after the `join` is complete:

```elixir
    send self(), {:after_join, status}
```

Finally, we return `:ok` and the current game data to the host socket:

```elixir
    {:ok, game_data, socket}
```

Lastly, we handle the `:after_join` message we passed from inside `join`:

```elixir
  def handle_info({:after_join, _status}, socket) do
    QuizGame.LobbyChannel.broadcast_current_games

    {:noreply, socket}
  end
```

Eventually, this channel will do much more; for now though this is enough to start a game and tell everyone waiting in the lobby about it. 

#### Host Page

Before we make the AppleTV client we'll have a basic web page to test things out. We'll need a page that the "Host" can visit to start a new game. We've already included a link to a `/host` endpoint on our Welcome page. To make that work, start by adding the route to the router in `web/router.ex`:

```elixir
defmodule QuizGame.Router do
  # ...
  
  scope "/", QuizGame do
    pipe_through :browser # Use the default browser stack

    get "/", PageController, :index
    get "/lobby", PageController, :lobby
    get "/host", PageController, :host
  end
  
  # ...
end
```

Next, we'll need to add the matching action in `web/controllers/page_controller.ex`:

```elixir
defmodule QuizGame.PageController do
  # ...
  
  @doc """
  When a host views the host page we generate a new random host id that
  will be used to identify them as a host and their game using the same
  id.
  """
  def host(conn, _params) do
    host_id = conn.cookies["host"] || QuizGame.generate_host_id

    conn
    |> put_resp_cookie("host", host_id, max_age: 24*60*60)
    |> render("host.html", id: host_id)
  end
  
  # ...
end

```

This action assumes there is a template called `host.html`, you should create that now at `web/templates/page/host.html.eex`:

```html
<script>
    window.hostId = '<%= @id %>';
</script>

<div class="state" data-state="start">
  <div class="jumbotron">
    <h2 id="welcome" class="animated pulse">Start a new game!</h2>
  </div>

  <div style="text-align:center">
    <a class="btn" id="start" href="#">Start</a>
  </div>
</div>

<div class="state" data-state="waiting">
  <div class="jumbotron">
    <h2 id="welcome" class="animated pulse">Waiting for <br>players!</h2>
  </div>

  <div style="text-align:center">
    <a class="btn" id="ready" href="#">Ready</a>
  </div>
</div>

<div style="text-aling:center">
  <ul id="players">
  </ul>
</div>

<div class="meta">
  Host id: <%= @id %>
</div>
```

This page is very similar to the Welcome and Lobby pages. We output the host id to a global `window` variable so that it is broadly accessible.  This time however, we have a couple `state` blocks in our HTML. As the state of the game changes we'll hide and show the different blocks.

#### Host JavaScript

The script needs to handle the basics of connecting the socket to our phoenix application and handling any changes to the state. Add the following in `web/static/js/host.js`:

```javascript
import {Socket} from 'phoenix'

let Host = {
  game: null,

  channel: null,

  init() {
    Host.attachEvents()
    Host.state('start')
  },

  attachEvents() {
    let button = document.getElementById("start")
    button.onclick = Host.start
  },

  // Handle state transitions and update the interface
  state(status) {
    document.querySelectorAll('.state').forEach(el => {
      if (el.getAttribute('data-state') === status) {
        el.className = 'state active'
      } else {
        el.className = 'state'
      }
    })
    Host.renderPlayers()
  },

  // When the user clicks the Start Game button
  start(e) {
    if (e) {
      e.preventDefault()
      e.stopPropagation()
    }
    Host.connect()
  },

  connect() {
    let socket = new Socket("/socket", {params: {id: window.hostId}})
    socket.connect()

    let channel = socket.channel('host:'+window.hostId)

    // When you first join the channel, the game will be looked up
    // or created. If the connection to the host is severed the game will
    // continue to run and the host can reconnect.
    channel.join()
      .receive('ok', game => {
        console.log('ok', game)
        Host.game = game
        Host.state(game.state)
      })
      .receive('error', reply => {
        error(`Sorry, you can't join because ${reply.reason}`)
      })

    Host.channel = channel
  },

  renderPlayers() {
    if (!Host.game) return

    let ul = document.getElementById('players')
    ul.innerHTML = `
      <ul>
        ${Host.game.players.map(player => `<li>${player}</li>`).join('')}
      </ul>
    `
  },

  error(message) {
    console.log(message)
  },
}
export default Host
```

Open `web/static/js/app.js` and add:

```javascript
import Host from "./host.js"

if (window.location.pathname === '/host') {
  Host.init()
}
```



### Player

### Playing the game

### Buzz

### Scoring

### Winning the game


# Lifecycle of a game

* There are no games, the server is waiting
* Player 1 goes to the welcome page, gets a random player id
    - Player 1 connects to the user socket, nothing happens
* Player 1 goes to the lobby page
    - Player 1 connects to the user socket, then connects to the lobby channel
    - A list of games is returned, it is empty
* Host 1 goes to the host page, gets a random host id
    - Host 1 connects to the user socket and joins the host channel
    - If the host already has a running game, it connects to that game
    - If there is no running game for the host a game is created (by the Supervisor)
    - The current state of the game (existing or created) is fetched and then returned to the host
    - After joining a message is sent to broadcast all games to the lobby
* Player 1 is still waiting in the lobby and receives a game broadcast
    - Player 1 sees the newly created game and clicks on the join button to join the game
    - After join, a message is sent to all players and the host about the new player (and game state)
* Host 1 sees that there is a player and can start the first round
    - Host clicks start which sends a message to start the game
    - The first round starts and the host starts a timer
    - If Player 1 buzzes in the timer is paused
        - Player 1 sends a guess
            - If right, Player 1 gets points and the turn ends
            - If wrong, Player 1 loses points and cannot buzz anymore
            - If there are no remaining players to buzz the turn ends
    - If Player 1 does not buzz in and the timer expires the turn ends
    - Scores are totaled, and an update is sent to everyone
    - Host 1 clicks "next round" (this could be automated)

# Testing

Opening half-a-dozen browser tabs and refreshing is fun for a while...

# When Rad Turns Bad

* What happens when the host's network connection drops?
* What happens when the player's network connection drops?
* What happens when a player leaves?
* What happens if you want to kick a player out of the game?
* What happens when you want to stop playing the current game?
* What happens when a message doesn't make it to or from the server?
* What happens when the game process crashes?
* What happens when the application is restarted?
* What happens when you want to deploy a new version?
* How many concurrent games can be played?
* How many players can join a game?
* Do we need security?

# Resources

Some really useful links:

## Using GenServer and Processes for State

* [Storing State in Elixir With Processes](http://dantswain.herokuapp.com/blog/2015/01/06/storing-state-in-elixir-with-processes/) by *Dan Swain* - if you are trying to understand why or how (recursion?!) Elixir uses processes to store state this should clarify quite a bit.

## Sockets and Channels in Phoenix

* [Building Phoenix Battleship](http://codeloveandboards.com/blog/2016/04/29/building-phoenix-battleship-pt-1/) by *Ricardo García Vega* - and [Part 2](http://codeloveandboards.com/blog/2016/04/29/building-phoenix-battleship-pt-2), [Part 3](http://codeloveandboards.com/blog/2016/04/29/building-phoenix-battleship-pt-3), [Part 4](http://codeloveandboards.com/blog/2016/04/29/building-phoenix-battleship-pt-4), [Part 5](http://codeloveandboards.com/blog/2016/04/29/building-phoenix-battleship-pt-5) - is a complete walk-through of how to build a real-time multiplayer game with a lobby. It gives the broadest overview of all of the OTP citizens you should use without focusing on the front-end code.

* [Creating a Game Lobby System in Phoenix with Websockets](https://quickleft.com/blog/creating-game-lobby-system-phoenix-websockets/) by *Alex Jensen* - has a really simple set of patterns and demonstrates authentication with Phoenix.Token and shows how to intercept outgoing pushes to create an invite system.

* [LearnPhoenix.tv's Tic-Tac-Toe Example](https://github.com/learnphoenixtv/tic_tac_toe) - Really straightforward and clean use of `GenServer` to manage game state.

* [Simple Chat Example](https://github.com/chrismccord/phoenix_chat_example) by *Chris McCord* - another small project demonstrating basic sockets and channels.

* [Building Multiplayer Games with Phoenix and Phaser (ElixirConfEU 2016)](https://www.youtube.com/watch?v=I5L9_cXwBcU) by *Keith Salisbury* - great intro video into some basic connections between Phaser (the JavaScript web gaming framework) and Phoenix channels and sockets. The section on [Message Flow](https://youtu.be/I5L9_cXwBcU?t=1251) and the visual representation of it is fantastic.

## Registry

* [Process registry with Elixir](https://m.alphasights.com/process-registry-in-elixir-a-practical-example-4500ee7c0dcc#.wejx8widr) by *Brian Thomas Storti* - excellent build up to a hand-built Registry (which predates the Elixir 1.4 Registry module). This is a perfect article to start with if you are trying to understand the **why** of `Registry`.

* [Using the Registry in Elixir 1.4](https://medium.com/@adammokan/registry-in-elixir-1-4-0-d6750fb5aeb#.nzwzoli9i) by *Adam Mokan* - quick introduction to using the new Registry pattern with a `GenServer` and `Supervisor` (see [the code](https://github.com/amokan/registry_sample)).

## Presence

I didn't use `Presence` in my work mostly because I didn't need the CRDT aspects. Additionally, when seeing those parts of `Presence` in action it felt too slow.

* [Building a Chat App with Elixir and Phoenix Presence](http://work.stevegrossi.com/2016/07/11/building-a-chat-app-with-elixir-and-phoenix-presence/) by *Steve Grossi* - how to tackle online user tracking through a supervised Presence.

* [Phoenix Presence sneak peek – step-by-step walkthrough](https://www.youtube.com/watch?v=9dALrnCOLNE) by *Chris McCord* - Video walkthrough of how `Presence` works with demonstration. (see also [The DockYard Blog](https://dockyard.com/blog/2016/03/25/what-makes-phoenix-presence-special-sneak-peek))

## Elixir & Phoenix

* [`GenServer`](https://hexdocs.pm/elixir/GenServer.html#content)
* [`GenEvent`](https://hexdocs.pm/elixir/GenEvent.html#content)
* [`Supervisor`](https://hexdocs.pm/elixir/Supervisor.html#content)
* [`Agent`](https://hexdocs.pm/elixir/Agent.html#content)
* [`Phoenix.Channel`](https://hexdocs.pm/phoenix/Phoenix.Channel.html)
* [`Phoenix.Socket`](https://hexdocs.pm/phoenix/Phoenix.Socket.html)





