# Phoenix & Elixir

## Pre-reqs

* Elixir 1.1.0 or greater
* Hex
* Postgres
* Node version 5.3.0 or greater

```    ​​
​​brew update
brew install nodejs
brew install postgres
brew install elixir
mix local.hex
```

### Vim

For vim support you can use: https://github.com/elixir-lang/vim-elixir. If you're using `vim-elixir` then you should remove the support for elixir in syntastic.

You might want to add `_build` and `deps` to your custom ignore:

`let g:ctrlp_custom_ignore = '\v[\/](node_modules|target|dist|tmp|log|_build|deps)|(\.(swp|ico|git|svn))$'`


## Install Phoenix

Get the latest:

```
mix archive.install https://github.com/phoenixframework/archives/raw/master/phoenix_new.ez
```
​
## Hello World

```
mix​​ ​​phoenix.new​​ ​​hello​
cd hello
mix ecto.create
mix ecto.migrate
mix phoenix.server
```

> If you get the error **"\*\* (Mix) The database for Hello.Repo couldn't be created: FATAL (invalid_authorization_specification): role "postgres" does not exist"** then `createuser -s postgres`


## Run tests

```
mix test
```

# Questions

* What is scrub_params
* Why is :empty deprecated
