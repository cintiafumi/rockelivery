# Rockelivery

## Introdução do Módulo 4

API para pedidos de um restaurante.

- Usuário pode se cadastrar e gerenciar sua conta
- Items do restaurante podem ser cadastrados
- Usuário pode realizar pedidos
  - Para realizar pedidos, usuário tem que se logar
  - CEP do usuário deve ser válido
- Semanalmente, um relatório de pedidos por usuário deve ser gerado

## Setup inicial do projeto

Instalar o [Phoenix](https://hexdocs.pm/phoenix/installation.html#phoenix), nosso framework web.

```bash
$ mix archive.install hex phx_new 1.5.9
```

Precisamos do [PostgreSQL](https://hexdocs.pm/phoenix/installation.html#postgresql) instalado também. (Fiz pelo docker)

Para usuários de linux tem que instalar [inotify-tools (for linux users)](https://hexdocs.pm/phoenix/installation.html#inotify-tools-for-linux-users) para o hot reload funcionar.

Agora com tudo instalado, vamos rodar

```bash
$ mix phx.new rockelivery --no-html --no-webpack
```

que vai ser uma api em json e não precisamos de nada de html e webpack.

Vamos adicionar a dependência do Credo em `mix.exs`

```elixir
{:credo, "~> 1.5", only: [:dev, :test], runtime: false}
```

Rodar o comando para gerar o `.credo.exs` e desabilitar o `ModuleDoc`

```bash
$ mix credo.gen.config
* creating .credo.exs
```

Vamos nos arquivos de `config/dev.exs` e `config/test.exs`, e verificar se as configurações do postgres está correta. Deixar o postgres do docker rodando e rodar o setup para gerar o database

```bash
$ mix ecto.setup
Compiling 11 files (.ex)
Generated rockelivery app
The database for Rockelivery.Repo has been created

20:23:19.708 [info]  Migrations already up
```

E vamos rodar o servidor

```bash
$ mix phx.server
[info] Running RockeliveryWeb.Endpoint with cowboy 2.9.0 at 0.0.0.0:4000 (http)
[info] Access RockeliveryWeb.Endpoint at http://localhost:4000
```

E abrimos no navegador `http://localhost:4000/dashboard`

## Conhecendo a estrutura de diretórios

Todo projeto phoenix terá duas pastas em `lib`. Uma com o nome do projeto `rockelivery`, onde fica a lógica de negócios e comunicação com banco de dados (arquivo `repo.ex`). E uma `rockelivery_web` onde fica tudo relacionado a web, como `controllers`, `views`, `router.ex` onde já está a rota `/dashboard`. É então um framework MVC. Apesar de não ter `models`, temos `schemas`.

Na pasta `config` estão as variáveis para cada ambiente. Em `config.exs` ficam as variáveis que valem para todos ambientes.

A pasta `_build` é tudo que foi compilado.

A pasta `priv` é tudo que queremos que vá para produção no pacotinho binário, mas que não estejam na code base, como assets, configurações de tradução `gettext`, as `migrations` e `seeds` para o banco de dados.

O arquivo `mix.lock` define as versões que estão sendo executadas de cada lib dando um `lock` na versão.

## Criando nossa primeira rota

No arquivo de rotas vamos adicionar a rota, o nome do controller que vai tratar a rota e a action que vai tratar essa rota.

```elixir
  scope "/api", RockeliveryWeb do
    pipe_through :api

    get "/", WelcomeController, :index
  end
```

Vamos criar o controller `welcome_controller.ex`, onde o módulo terá a estrutura `defmodule` com o nome da aplicação com sufixo `Web` pois estamos no contexto web, `RockeliveryWeb.NomeDoController`.

Todo controller vai ter o `use RockeliveryWeb, :controller`. Dessa forma estamos definindo que esse arquivo é um controller e estamos trazendo todas as funcionalidades de um controller para dentro desse módulo.

E como no `router.ex` criamos uma função `:index`, precisamos criar essa action, que terá como parâmetros a conexão e os parâmetros que essa rota recebe.

```elixir
defmodule RockeliveryWeb.WelcomeController do
  use RockeliveryWeb, :controller

  def index(conn, params) do
    conn
    |> put_status(:ok)
    |> text("Welcome =)")
  end
end
```

Deixar o docker com postgres ligado, iniciar o servidor e verificar a rota pelo navegador. Como queremos dar um `GET` em `/` e estamos na rota de `/api`, acessamos `http://localhost:4000/api/`. E com status `200` na aba de network do navegador.

Antes do `put_status`, adicionamos um `IO.inspect()` e vemos no terminal a struct da `conn`. Podemos perceber que o `status` está `nil`. Se adicionar o `IO.inspect()` depois do `put_status`, então o status está `200`. O `resp_body` é `nil` por default.

```elixir
[info] GET /api/
[debug] Processing with RockeliveryWeb.WelcomeController.index/2
  Parameters: %{}
  Pipelines: [:api]
%Plug.Conn{
  adapter: {Plug.Cowboy.Conn, :...},
  assigns: %{},
  before_send: [#Function<0.11227428/1 in Plug.Telemetry.call/2>],
  body_params: %{},
  cookies: %{},
  halted: false,
  host: "localhost",
  method: "GET",
  owner: #PID<0.495.0>,
  params: %{},
  path_info: ["api"],
  path_params: %{},
  port: 4000,
  private: %{
    RockeliveryWeb.Router => {[], %{}},
    :phoenix_action => :index,
    :phoenix_controller => RockeliveryWeb.WelcomeController,
    :phoenix_endpoint => RockeliveryWeb.Endpoint,
    :phoenix_format => "json",
    :phoenix_layout => {RockeliveryWeb.LayoutView, :app},
    :phoenix_request_logger => {"request_logger", "request_logger"},
    :phoenix_router => RockeliveryWeb.Router,
    :phoenix_view => RockeliveryWeb.WelcomeView,
    :plug_session_fetch => #Function<1.123471702/1 in Plug.Session.fetch_session/1>
  },
  query_params: %{},
  query_string: "",
  remote_ip: {127, 0, 0, 1},
  req_cookies: %{},
  req_headers: [
    {"accept",
     "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9"},
    {"accept-encoding", "gzip, deflate, br"},
    {"accept-language", "en"},
    {"cache-control", "max-age=0"},
    {"connection", "keep-alive"},
    {"host", "localhost:4000"},
    {"sec-ch-ua",
     "\" Not;A Brand\";v=\"99\", \"Google Chrome\";v=\"91\", \"Chromium\";v=\"91\""},
    {"sec-ch-ua-mobile", "?0"},
    {"sec-fetch-dest", "document"},
    {"sec-fetch-mode", "navigate"},
    {"sec-fetch-site", "none"},
    {"sec-fetch-user", "?1"},
    {"upgrade-insecure-requests", "1"},
    {"user-agent",
     "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.77 Safari/537.36"}
  ],
  request_path: "/api/",
  resp_body: nil,
  resp_cookies: %{},
  resp_headers: [
    {"cache-control", "max-age=0, private, must-revalidate"},
    {"x-request-id", "FocP0PmoTzIHh8QAAAUh"}
  ],
  scheme: :http,
  script_name: [],
  secret_key_base: :...,
  state: :unset,
  status: nil
}
[info] Sent 200 in 30ms
```

Os `params` são os parâmetros que passamos na rota. Se tivermos uma rota `/:id` que espera o parâmetro `:id`, podemos fazer o inspect e ver

```elixir
IO.inspect(params, label: "VALOR ---->>>")
```

Iremos ver no terminal:

```bash
VALOR ---->>>: %{"id" => "1234"}
%Plug.Conn{
  ...
  params: %{"id" => "1234"},
  ...
}
```

## Criando a migration do User

A primeira ação que iremos fazer do CRUD do usuário é a criação no banco de dados. Para criar a tabela no nosso banco, digitar no terminal

```bash
$ mix ecto.gen.migration create_users_table
Compiling 2 files (.ex)
* creating priv/repo/migrations/20210612022008_create_users_table.exs
```

O arquivo de migration mapeia como o banco de dados está sendo construído. Quais tabelas, quais campos nas tabelas e qual a ordem que elas devem ser criadas. Pelo comando no terminal, é gerado um timestamp pelo qual o Ecto saberá qual ordem seguir para criar as tabelas.

Dentro da função `change` que criamos as especificações da nossa tabela e do nosso banco de dados. Tudo que criarmos dentro de `change` já é feita a criação e o rollback.

Iremos salvar no banco o campo `password_hash`, que será a senha criptografada do nosso usuário

Adicionamos o `timestamps()` que adiciona na tabela os campos `inserted_at` e `updated_at`, que mantém o tracking de data completa quando um registro foi inserido e quando foi atualizado, principalmente, para criar relatórios e ordenação.

Por fim, vamos criar dois index, pois não queremos usuários com mesmo cpf e nem com mesmo e-mail.

Por default, os `id` são inteiros. Para alterar para uuid, temos que alterar nas configurações do projeto `config.exs` e adicionamos

```elixir
config :rockelivery, Rockelivery.Repo,
  migration_primary_key: [type: :binary_id],
  migration_foreign_key: [type: :binary_id]
```

`:binary_id` é o uuid do tipo uuid4.

Vamos resetar o banco

```bash
$ mix ecto.reset
Compiling 12 files (.ex)
Generated rockelivery app
The database for Rockelivery.Repo has been dropped
The database for Rockelivery.Repo has been created

23:39:29.281 [info]  == Running 20210612022008 Rockelivery.Repo.Migrations.CreateUsersTable.change/0 forward

23:39:29.290 [info]  create table users

23:39:29.319 [info]  create index users_cpf_index

23:39:29.329 [info]  create index users_email_index

23:39:29.350 [info]  == Migrated 20210612022008 in 0.0s
```

Ou podemos rodar assim

```bash
$ mix ecto.drop
The database for Rockelivery.Repo has been dropped

$ mix ecto.create
The database for Rockelivery.Repo has been created

$ mix ecto.migrate

23:41:04.760 [info]  == Running 20210612022008 Rockelivery.Repo.Migrations.CreateUsersTable.change/0 forward

23:41:04.765 [info]  create table users

23:41:04.794 [info]  create index users_cpf_index

23:41:04.804 [info]  create index users_email_index

23:41:04.823 [info]  == Migrated 20210612022008 in 0.0s
```

E se rodar de novo, já estão executando:

```bash
$ mix ecto.migrate

23:42:29.354 [info]  Migrations already up
```
