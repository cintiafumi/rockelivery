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

## Criando o schema do User pt 1

A diferença principal de uma migration para um schema é que a migration modela o modelo de dados, ou seja, modelo como os dados são salvos no banco de dados, como é a tabela e quais as colunas dela. O schema é uma representação desse modelo num código que o elixir pode entender. O schema vai devolver uma struct para representar esses dados, com validações e com cast de dados.

Em `rockelivery` vamos criar um novo arquivo `user.ex`. Todos arquivos que representam um schema, criamos o schema na raiz dessa pasta e uma pasta com o nome do contexto no plural contendo todas as ações e lógicas ao redor do usuário fica dentro dessa pasta `users`.

De novo, temos que definir que `id` é do tipo `:binary_id`

```elixir
defmodule Rockelivery.User do
  use Ecto.Schema
  import Ecto.Changeset

  @primary_key {:id, :binary_id, autogenerate: true}

  schema "users" do
    field :address, :string
    field :age, :integer
    field :cep, :string
    field :cpf, :string
    field :email, :string
    field :name, :string
    field :password_hash, :string

    timestamps()
  end
end
```

E no `iex` podemos definir um schema que retorna a struct

```elixir
iex> %Rockelivery.User{}
%Rockelivery.User{
  __meta__: #Ecto.Schema.Metadata<:built, "users">,
  address: nil,
  age: nil,
  cep: nil,
  cpf: nil,
  email: nil,
  id: nil,
  inserted_at: nil,
  name: nil,
  password_hash: nil,
  updated_at: nil
}
```

O `Changeset` recebe uma struct do tipo User e consegue tanto fazer cast de dados para inserir nessa struct, como também fazer validações e modificações. E a função tem que chamar `changeset` por default do Ecto. A função `cast` pega os params e vai tentar fazer o cast nos campos da struct, e como segundo argumento temos que definir uma lista de campos para cast `@required_params`.

```elixir
  def changeset(params) do
    %__MODULE__{}
    |> cast(params, @required_params)
  end
```

No `iex`, criamos o mapa `user_params`

```elixir
iex> user_params = %{address: "Rua das bananeiras", age: 36, cep: "1234", cpf: "123", email: "email", name: "Cintia", password_hash: "123"}
%{
  address: "Rua das bananeiras",
  age: 36,
  cep: "1234",
  cpf: "123",
  email: "email",
  name: "Cintia",
  password_hash: "123"
}

iex> alias Rockelivery.Use
Rockelivery.User

iex> User.changeset(user_params)
#Ecto.Changeset<
  action: nil,
  changes: %{
    address: "Rua das bananeiras",
    age: 36,
    cep: "1234",
    cpf: "123",
    email: "email",
    name: "Cintia",
    password_hash: "123"
  },
  errors: [],
  data: #Rockelivery.User<>,
  valid?: true
>
```

Agora temos um Changeset do Ecto que é uma struct especial que valida os dados, faz cast dos dados e essa struct vai ser mandada para o banco. O Repo só vai inserir no banco se o Changeset for válido.

## Criando o schema do User pt 2

Vamos criar a validação pela função `validate_required` que também recebe uma lista.

```elixir
  def changeset(params) do
    %__MODULE__{}
    |> cast(params, @required_params)
    |> validate_required(@required_params)
  end
```

No `iex`, vamos remover o `name` de `user_params` e tentar criar o changeset novamente

```elixir
iex> user_params = %{address: "Rua das bananeiras", age: 36, cep: "1234", cpf: "123", email: "email", password_hash: "123"
%{
  address: "Rua das bananeiras",
  age: 36,
  cep: "1234",
  cpf: "123",
  email: "email",
  password_hash: "123"
}

iex> User.changeset(user_params)
#Ecto.Changeset<
  action: nil,
  changes: %{
    address: "Rua das bananeiras",
    age: 36,
    cep: "1234",
    cpf: "123",
    email: "email",
    password_hash: "123"
  },
  errors: [name: {"can't be blank", [validation: :required]}],
  data: #Rockelivery.User<>,
  valid?: false
>
```

E vamos adicionar mais validações

```elixir
  def changeset(params) do
    %__MODULE__{}
    |> cast(params, @required_params)
    |> validate_required(@required_params)
    |> validate_length(:password_hash, min: 6)
    |> validate_length(:cep, is: 8)
    |> validate_length(:cpf, is: 11)
    |> validate_number(:age, greater_than_or_equal_to: 18)
    |> unique_constraint([:email])
    |> unique_constraint([:cpf])
  end
```

E testar no `iex`

```elixir
iex> user_params = %{address: "Rua das bananeiras", age: 16, cep: "1234", cpf: "123", email: "email", name: "Cintia", password_hash: "123"}
%{
  address: "Rua das bananeiras",
  age: 16,
  cep: "1234",
  cpf: "123",
  email: "email",
  name: "Cintia",
  password_hash: "123"
}

iex> User.changeset(user_params)
#Ecto.Changeset<
  action: nil,
  changes: %{
    address: "Rua das bananeiras",
    age: 16,
    cep: "1234",
    cpf: "123",
    email: "email",
    name: "Cintia",
    password_hash: "123"
  },
  errors: [
    age: {"must be greater than or equal to %{number}",
     [validation: :number, kind: :greater_than_or_equal_to, number: 18]},
    cpf: {"should be %{count} character(s)",
     [count: 11, validation: :length, kind: :is, type: :string]},
    cep: {"should be %{count} character(s)",
     [count: 8, validation: :length, kind: :is, type: :string]},
    password_hash: {"should be at least %{count} character(s)",
     [count: 6, validation: :length, kind: :min, type: :string]}
  ],
  data: #Rockelivery.User<>,
  valid?: false
>

iex> user_params = %{address: "Rua das bananeiras", age: 36, cep: "12345678", cpf: "12345678900", email: "email", name: "Cintia", password_hash: "123456"}
%{
  address: "Rua das bananeiras",
  age: 36,
  cep: "12345678",
  cpf: "12345678900",
  email: "email",
  name: "Cintia",
  password_hash: "123456"
}

iex> User.changeset(user_params)
#Ecto.Changeset<
  action: nil,
  changes: %{
    address: "Rua das bananeiras",
    age: 36,
    cep: "12345678",
    cpf: "12345678900",
    email: "email",
    name: "Cintia",
    password_hash: "123456"
  },
  errors: [],
  data: #Rockelivery.User<>,
  valid?: true
>
```

Podemos fazer a validação por regex, como no email

```elixir
    |> validate_format(:email, ~r/@/)
```

O campo `password_hash` existe para salvar no banco a senha com um hash e para quem usa, é estranho expor esse hash. A primeira coisa que faremos é criar um campo virtual que existe no nosso schema mas não existe no banco.

```elixir
    field :password, :string, virtual: true
```

E também alteramos para `@required_params` como `:password`

```elixir
  @required_params [:address, :age, :cep, :cpf, :email, :name, :password]
```

Se passar por todas validações do Ecto e recebermos um changeset com valid true, vamos adicionar o passo de criptografar esse `password` e salvar no campo `password_hash`. Caso seja inválido, só devolve o changeset.

Vamos criar a função privada `put_password_hash` que recebe um struct `%Changeset{}` que se tiver um campo `valid` que esteja `true`, vamos ler o campo `password` e vamos chamar a função `change` do `Ecto.Changeset`, que recebe um dado e um novo mapa, aplicando as novas modificações ao Changeset.

Precisamos passar um novo map com o nome do campo e o novo valor. Para isso, usaremos uma lib `Pbkdf2` que faz isso já. Na função `add_hash` que recebe uma string, criptografa.

Vamos adicionar a lib no `mix.exs` nas dependências

```elixir
      {:pbkdf2_elixir, "~> 1.4"}
```

A função `add_hash` recebe a string, criptografa e cria o map da forma que o Ecto espera e com o nome do campo `password_hash`.

```elixir
iex> Pbkdf2.add_hash("123456")
%{
  password_hash: "$pbkdf2-sha512$160000$ts.ScEst971Z7I/EIG/elA$UZJ/X5cbuMqRUhGexkcp7U7z.weO13lmp5zpN0mObsyBWOGItmNJU3bXjJKd6ydDlkLb9h0yZOOqj2Gsmrv36g"
}
```

E com isso, a função `change` receberá isso como parâmetro

```elixir
defmodule Rockelivery.User do
  use Ecto.Schema
  import Ecto.Changeset

  alias Ecto.Changeset

  @primary_key {:id, :binary_id, autogenerate: true}

  @required_params [:address, :age, :cep, :cpf, :email, :name, :password]

  schema "users" do
    field :address, :string
    field :age, :integer
    field :cep, :string
    field :cpf, :string
    field :email, :string
    field :name, :string
    field :password, :string, virtual: true
    field :password_hash, :string

    timestamps()
  end

  def changeset(params) do
    %__MODULE__{}
    |> cast(params, @required_params)
    |> validate_required(@required_params)
    |> validate_length(:password_hash, min: 6)
    |> validate_length(:cep, is: 8)
    |> validate_length(:cpf, is: 11)
    |> validate_number(:age, greater_than_or_equal_to: 18)
    |> validate_format(:email, ~r/@/)
    |> unique_constraint([:email])
    |> unique_constraint([:cpf])
    |> put_password_hash()
  end

  defp put_password_hash(%Changeset{valid?: true, changes: %{password: password}} = changeset) do
    change(changeset, Pbkdf2.add_hash(password))
  end
end
```

E no `iex` podemos ver como fica

```elixir
iex> user_params = %{address: "Rua das bananeiras", age: 36, cep: "12345678", cpf: "12345678900", email: "cintia@banana.com", name: "Cintia", password: "123456"}
%{
  address: "Rua das bananeiras",
  age: 36,
  cep: "12345678",
  cpf: "12345678900",
  email: "cintia@banana.com",
  name: "Cintia",
  password: "123456"
}

iex> User.changeset(user_params)
#Ecto.Changeset<
  action: nil,
  changes: %{
    address: "Rua das bananeiras",
    age: 36,
    cep: "12345678",
    cpf: "12345678900",
    email: "cintia@banana.com",
    name: "Cintia",
    password: "123456",
    password_hash: "$pbkdf2-sha512$160000$HhKMxBhAFwTmSUmMOT3ZWQ$hz99Qg76qUzoQSGK9RorlAAgniZuaCrYfPXBKbC5rpkR8jJksdcF6ST1DmhMy4FITXP4Q6ExHx75LwmUz./Byw"
  },
  errors: [],
  data: #Rockelivery.User<>,
  valid?: true
>
```

O campo `password` ainda aparece por ser virtual, mas na hora de salvar no banco não vai existir essa coluna.

Quando o changeset for inválido, devemos retornar o próprio changeset, então, adicionamos outra função privada para capturar a exceção

```elixir
  defp put_password_hash(changeset), do: changeset
```

## Inserindo o User no banco

O [`Ecto.Repo`](https://hexdocs.pm/ecto/Ecto.Repo.html) define um repositório e mapeia os dados que temos no elixir e o nosso repositório físico que é o nosso banco de dados. O Adapter implementa os callbacks. Vamos testar no `iex`

```elixir
iex> alias Rockelivery.Repo
Rockelivery.Repo

iex> user_params = %{address: "Rua das bananeiras", age: 36, cep: "12345678", cpf: "12345678900", email: "cintia@banana.com", name: "Cintia", password: "123456"}
%{
  address: "Rua das bananeiras",
  age: 36,
  cep: "12345678",
  cpf: "12345678900",
  email: "cintia@banana.com",
  name: "Cintia",
  password: "123456"
}

iex> alias Rockelivery.User
Rockelivery.User

iex> user_params |> User.changeset() |> Repo.insert()
[debug] QUERY OK db=12.2ms decode=6.5ms queue=4.5ms idle=1195.8ms
INSERT INTO "users" ("address","age","cep","cpf","email","name","password_hash","inserted_at","updated_at","id") VALUES ($1,$2,$3,$4,$5,$6,$7,$8,$9,$10) ["Rua das bananeiras", 36, "12345678", "12345678900", "cintia@banana.com", "Cintia", "$pbkdf2-sha512$160000$kFH06jbBH6dkaRuIgIuHXA$fxhwqSV5YMI.rIPrnWUbU4W5gBR9db5avdhTdaBeyfXdNQjvvFY4BKHZavWOfabIzpTz/64/xCEhG6Bn.bgMLg", ~N[2021-06-14 01:09:04], ~N[2021-06-14 01:09:04], <<192, 138, 157, 52, 250, 184, 74, 14, 153, 225, 99, 249, 75, 38, 226, 244>>]
{:ok,
 %Rockelivery.User{
   __meta__: #Ecto.Schema.Metadata<:loaded, "users">,
   address: "Rua das bananeiras",
   age: 36,
   cep: "12345678",
   cpf: "12345678900",
   email: "cintia@banana.com",
   id: "c08a9d34-fab8-4a0e-99e1-63f94b26e2f4",
   inserted_at: ~N[2021-06-14 01:09:04],
   name: "Cintia",
   password: "123456",
   password_hash: "$pbkdf2-sha512$160000$kFH06jbBH6dkaRuIgIuHXA$fxhwqSV5YMI.rIPrnWUbU4W5gBR9db5avdhTdaBeyfXdNQjvvFY4BKHZavWOfabIzpTz/64/xCEhG6Bn.bgMLg",
   updated_at: ~N[2021-06-14 01:09:04]
 }}
```

E vemos se foi salvo no banco

```elixir
iex> Repo.all(User)
[debug] QUERY OK source="users" db=12.0ms queue=5.7ms idle=1959.1ms
SELECT u0."id", u0."address", u0."age", u0."cep", u0."cpf", u0."email", u0."name", u0."password_hash", u0."inserted_at", u0."updated_at" FROM "users" AS u0 []
[
  %Rockelivery.User{
    __meta__: #Ecto.Schema.Metadata<:loaded, "users">,
    address: "Rua das bananeiras",
    age: 36,
    cep: "12345678",
    cpf: "12345678900",
    email: "cintia@banana.com",
    id: "c08a9d34-fab8-4a0e-99e1-63f94b26e2f4",
    inserted_at: ~N[2021-06-14 01:09:04],
    name: "Cintia",
    password: nil,
    password_hash: "$pbkdf2-sha512$160000$kFH06jbBH6dkaRuIgIuHXA$fxhwqSV5YMI.rIPrnWUbU4W5gBR9db5avdhTdaBeyfXdNQjvvFY4BKHZavWOfabIzpTz/64/xCEhG6Bn.bgMLg",
    updated_at: ~N[2021-06-14 01:09:04]
  }
]
```

Ou pelo get

```elixir
iex> Repo.get(User, "c08a9d34-fab8-4a0e-99e1-63f94b26e2f4")
[debug] QUERY OK source="users" db=3.6ms queue=6.5ms idle=1404.9ms
SELECT u0."id", u0."address", u0."age", u0."cep", u0."cpf", u0."email", u0."name", u0."password_hash", u0."inserted_at", u0."updated_at" FROM "users" AS u0 WHERE (u0."id" = $1) [<<192, 138, 157, 52, 250, 184, 74, 14, 153, 225, 99, 249, 75, 38, 226, 244>>]
%Rockelivery.User{
  __meta__: #Ecto.Schema.Metadata<:loaded, "users">,
  address: "Rua das bananeiras",
  age: 36,
  cep: "12345678",
  cpf: "12345678900",
  email: "cintia@banana.com",
  id: "c08a9d34-fab8-4a0e-99e1-63f94b26e2f4",
  inserted_at: ~N[2021-06-14 01:09:04],
  name: "Cintia",
  password: nil,
  password_hash: "$pbkdf2-sha512$160000$kFH06jbBH6dkaRuIgIuHXA$fxhwqSV5YMI.rIPrnWUbU4W5gBR9db5avdhTdaBeyfXdNQjvvFY4BKHZavWOfabIzpTz/64/xCEhG6Bn.bgMLg",
  updated_at: ~N[2021-06-14 01:09:04]
}
```

Vamos criar um módulo auxiliar para criação de usuários no banco de dados. Criamos o arquivo `users/create.ex`

```elixir
defmodule Rockelivery.Users.Create do
  alias Rockelivery.{Repo, User}

  def call(params) do
    params
    |> User.changeset()
    |> Repo.insert()
  end
end
```

E vamos ver no `iex`

```elixir
iex> alias Rockelivery.Users.Create
Rockelivery.Users.Create

ex> user_params = %{address: "Rua das bananeiras", age: 36, cep: "12345678", cpf: "12345678902", email: "cintia2@banana.com", name: "Cintia", password: "123456"}
%{
  address: "Rua das bananeiras",
  age: 36,
  cep: "12345678",
  cpf: "12345678902",
  email: "cintia2@banana.com",
  name: "Cintia",
  password: "123456"
}

iex> Create.call(user_params)
[debug] QUERY OK db=4.9ms queue=2.7ms idle=350.0ms
INSERT INTO "users" ("address","age","cep","cpf","email","name","password_hash","inserted_at","updated_at","id") VALUES ($1,$2,$3,$4,$5,$6,$7,$8,$9,$10) ["Rua das bananeiras", 36, "12345678", "12345678902", "cintia2@banana.com", "Cintia", "$pbkdf2-sha512$160000$l8qjZB8q67g9Tv1I/6.S4A$XvIqbdx9GIGLAw3BsO3AA0JyS/wJ4edJxWhjnjxY4rcr56aGZSYjKUQAQxxnl7WwLsjr8WQF/KfUCsIhxc3tkA", ~N[2021-06-14 01:17:39], ~N[2021-06-14 01:17:39], <<108, 108, 192, 170, 6, 9, 72, 119, 149, 24, 159, 31, 23, 96, 229, 115>>]
{:ok,
 %Rockelivery.User{
   __meta__: #Ecto.Schema.Metadata<:loaded, "users">,
   address: "Rua das bananeiras",
   age: 36,
   cep: "12345678",
   cpf: "12345678902",
   email: "cintia2@banana.com",
   id: "6c6cc0aa-0609-4877-9518-9f1f1760e573",
   inserted_at: ~N[2021-06-14 01:17:39],
   name: "Cintia",
   password: "123456",
   password_hash: "$pbkdf2-sha512$160000$l8qjZB8q67g9Tv1I/6.S4A$XvIqbdx9GIGLAw3BsO3AA0JyS/wJ4edJxWhjnjxY4rcr56aGZSYjKUQAQxxnl7WwLsjr8WQF/KfUCsIhxc3tkA",
   updated_at: ~N[2021-06-14 01:17:39]
 }}
```

E vemos pelo `Repo.all(User)`

## Criando a rota de criação de usuários pt 1

Vamos criar a rota que irá criar o usuário pelo http. No arquivo `router.ex` temos o `pipeline` que é uma forma de criar configurações extras que atuam sobre nossa conexão. É isso que o `plug` faz, recebendo, validando e/ou modificando os dados da nossa conexão. O `scope` define um namespace na nossa rota.

```elixir
  scope "/api", RockeliveryWeb do
    pipe_through :api

    get "/", WelcomeController, :index
    resources "/users", UsersController # <-- Adição desta linha
  end
```

E criamos o módulo em `users_controller.ex` vazio somente para carregar o que o `resources` faz. Vemos no terminal

```bash
$ mix phx.routes
Compiling 2 files (.ex)
Generated rockelivery app
       welcome_path  GET     /api                    RockeliveryWeb.WelcomeController :index
         users_path  GET     /api/users              RockeliveryWeb.UsersController :index
         users_path  GET     /api/users/:id/edit     RockeliveryWeb.UsersController :edit
         users_path  GET     /api/users/new          RockeliveryWeb.UsersController :new
         users_path  GET     /api/users/:id          RockeliveryWeb.UsersController :show
         users_path  POST    /api/users              RockeliveryWeb.UsersController :create
         users_path  PATCH   /api/users/:id          RockeliveryWeb.UsersController :update
                     PUT     /api/users/:id          RockeliveryWeb.UsersController :update
         users_path  DELETE  /api/users/:id          RockeliveryWeb.UsersController :delete
live_dashboard_path  GET     /dashboard              Phoenix.LiveView.Plug :home
live_dashboard_path  GET     /dashboard/:page        Phoenix.LiveView.Plug :page
live_dashboard_path  GET     /dashboard/:node/:page  Phoenix.LiveView.Plug :page
          websocket  WS      /live/websocket         Phoenix.LiveView.Socket
           longpoll  GET     /live/longpoll          Phoenix.LiveView.Socket
           longpoll  POST    /live/longpoll          Phoenix.LiveView.Socket
          websocket  WS      /socket/websocket       RockeliveryWeb.UserSocket
```

Então já temos todas as actions. Como não teremos algumas das rotas, podemos passar a opção de excluir em `resource`

```elixir
    resources "/users", UsersController, except: [:new, :edit]
```

No nosso controller, teremos as funções das actions que podem precisar de muitos `alias` renomeados. Então, para evitar isso, podemos criar os arquivos de facade

```elixir
defmodule RockeliveryWeb.UsersController do
  use RockeliveryWeb, :controller
  alias Rockelivery.Users.Create, as: UserCreate
  alias Rockelivery.Orders.Create, as: OrderCreate
  #...
end
```

Na raiz, temos o arquivo `rockelivery.ex` e chamamos o `defdelegate` que delega a chamada

```elixir
defmodule Rockelivery do
  alias Rockelivery.Users.Create, as: UserCreate

  defdelegate create_user(params), to: UserCreate, as: :call
end
```

## Criando a rota de criação de usuários pt 2

Adicionamos o `with` no `UserController` para validar os cenários por pattern matching. E podemos colocar vários casos de sucesso de uma vez só. Se usasse `case`, ficaria um `case` encadeado no outro. Se, não tratar o erro especificamente, devolvemos esse erro para quem chamou. Senão, tratamos no `else`.

```elixir
defmodule RockeliveryWeb.UsersController do
  use RockeliveryWeb, :controller
  alias Rockelivery.User

  def create(conn, params) do
    with {:ok, %User{}} <- Rockelivery.create_user(params),
         {:ok, %User{}} <- Rockelivery.outra_funcao(params),
         {:ok, %User{}} <- Rockelivery.outra_funcao2(params) do
    else
      :error -> alguma_coisa
      nil -> outra_coisa
    end
  end
end
```

Vamos retornar um `201 created` e chamar uma função `render`, que chama uma `view`, que é a definição do json que iremos imprimir para o usuário que fizer a chamada para nossa ação.

```elixir
defmodule RockeliveryWeb.UsersController do
  use RockeliveryWeb, :controller
  alias Rockelivery.User

  def create(conn, params) do
    with {:ok, %User{} = user} <- Rockelivery.create_user(params) do
      conn
      |> put_status(:created)
      |> render("create.json", user: user)
    end
  end
end
```

Passamos para nossa `view` esse `user` que recebemos como parâmetro via keyword argument.

Vamos criar a `view` no arquivo `users_view.ex`. Importante a `view` ter o mesmo nome que o `controller`, para o `controller` saber automaticamente qual módulo da `view` que será renderizado.

O esqueleto de uma `view`, basta criar uma função do tipo `render` do tipo `create.json` que vai receber o `user`. Vamos retornar um map que representa um json de criação do usuário. Escolhemos exibir `id` e `name`.

```elixir
defmodule RockeliveryWeb.UsersView do
  use RockeliveryWeb, :view

  alias Rockelivery.User

  def render("create.json", %{user: %User{id: id, name: name}}) do
    %{
      message: "User created!",
      user: %{
        id: id,
        name: name
      }
    }
  end
end
```

Vamos para o terminal e rodar o servidor

```bash
$ mix phx.server
```

E vamos usar o Insomnia (client http) para fazer chamadas para o servidor.

Vamos enviar uma requisição `POST` para `http://localhost:4000/api/users` e com um body:

```json
{
  "address": "Rua das bananeiras, S/N",
  "age": 36,
  "cep": "12345678",
  "cpf": "12345678900",
  "email": "cintia@banana.com",
  "name": "Cintia",
  "password": "123456"
}
```

Recebemos como retorno algo do tipo

```json
{
  "message": "User created!",
  "user": {
    "id": "2c6a3d71-e442-4ed5-a197-49f35ac70dbf",
    "name": "Cintia"
  }
}
```

Para não ter que ficar escrevendo que vai retornar o `id`, o `name`, tem uma forma mais fácil. No `mix.exs` tem uma lib `jason`.

Adicionamos ao `user.ex` o `@derive` passando uma tupla com o `Jason.Encoder` e `only:` com a lista do que quero mostrar na minha `view`.

```elixir
  @derive {Jason.Encoder, only: [:id, :age, :cpf, :address, :email, :name]}
```

Na `users_view` passamos a struct toda de `user`

```elixir
defmodule RockeliveryWeb.UsersView do
  use RockeliveryWeb, :view

  alias Rockelivery.User

  def render("create.json", %{user: %User{} = user}) do
    %{
      message: "User created!",
      user: user
    }
  end
end
```

## Tratando erros com o fallback controller

No `with` dentro do `create` de user, se não der pattern matching, o erro vai retornar para quem chamou. No phoenix, temos a convenção de tratar erros com fallback controller. Se ninguém tratar determinado retorno, o fallback vai ser o último na linha de pipeline de repassagem de parâmetros para receber esse retorno. Então, é bom centralizar a tratativa de erros nesse fallback controller.

No controller do user, vamos passar um `action_fallback` e passar o nome de um controller, no caso, `FallbackController`

```elixir
defmodule RockeliveryWeb.UsersController do
  use RockeliveryWeb, :controller

  alias Rockelivery.User
  alias RockeliveryWeb.FallbackController

  action_fallback FallbackController

  def create(conn, params) do
    with {:ok, %User{} = user} <- Rockelivery.create_user(params) do
      conn
      |> put_status(:created)
      |> render("create.json", user: user)
    end
  end
end
```

Vamos criar um novo arquivo `fallback_controller.ex` que será responsável por renderizar os erros quando eles existirem.

```elixir
defmodule RockeliveryWeb.FallbackController do
  use RockeliveryWeb, :controller

  alias RockeliveryWeb.ErrorView

  def call(conn, {:error, result}) do
    conn
    |> put_status(:bad_request)
    |> put_view(ErrorView)
    |> render("400.json", result: result)
  end
end
```

Um controller de Fallback, não definimos nenhuma action. Sempre vai executar uma função chamada `call` que recebe a conexão e os parâmetros. Se receber um `{:error, result}`, pegar a `conn` e colocar um status, chamar uma `view` e renderizar uma view `400.json` e enviar o `result`.

Ao testar no insomnia, temos o retorno do erro

```json
{
  "errors": {
    "detail": "Bad Request"
  }
}
```

A `ErrorView` tem como default um `template_not_found`. Agora temos que estender para renderizar os erros do changeset.

```elixir
  def render("400.json", %{result: result}) do
    %{message: "Deu ruim"}
  end
```

Vamos no insominia de novo

```json
{
  "message": "Deu ruim"
}
```

O que queremos exigir são os erros do Changeset. Se por pattern matching recebermos um `Changeset`, iremos traduzir a mensagem de erro numa função `translate_errors`. O Ecto já tem uma função `traverse_errors` que traduz o erro do Changeset que podemos importar para usá-la (preciso falar sua aridade).

```elixir
defmodule RockeliveryWeb.ErrorView do
  use RockeliveryWeb, :view

  import Ecto.Changeset, only: [traverse_errors: 2]

  alias Ecto.Changeset

  # If you want to customize a particular status code
  # for a certain format, you may uncomment below.
  # def render("500.json", _assigns) do
  #   %{errors: %{detail: "Internal Server Error"}}
  # end

  # By default, Phoenix returns the status message from
  # the template name. For example, "404.json" becomes
  # "Not Found".
  def template_not_found(template, _assigns) do
    %{errors: %{detail: Phoenix.Controller.status_message_from_template(template)}}
  end

  def render("400.json", %{result: %Changeset{} = changeset}) do
    %{message: translate_errors(changeset)}
  end

  defp translate_errors(changeset) do
    traverse_errors(changeset, fn {msg, opts} ->
      Enum.reduce(opts, msg, fn {key, value}, acc ->
        String.replace(acc, "%{#{key}}", to_string(value))
      end)
    end)
  end
end
```

E testando no insomnia uma requisição de criação de user com um cpf já utilizado anteriormente

```json
{
  "message": {
    "cpf": ["has already been taken"]
  }
}
```

Testando com uma idade menor

```json
{
  "message": {
    "age": ["must be greater than or equal to 18"]
  }
}
```

E com cep e cpf

```json
{
  "message": {
    "cep": ["should be 8 character(s)"],
    "cpf": ["should be 11 character(s)"]
  }
}
```

Todas validações de memória são mostradas separadamente, mas as `unique_constraints` temos que ir no banco validar se já existem.

## Possibilitando múltiplos status de erro

Vamos flexibilizar o fallback controller para escolher o status http conforme tratativa de erro.

Em `create.ex` ao invés de só devolve um `:ok`, vamos adicionar um `handle_insert` que no caso de sucesso só retorna o result. E para caso de erro, enviaremos um mapa contendo o status (nesse caso será um `:bad_request`) e o result. Então, conseguiremos alterar o status.

```elixir
defmodule Rockelivery.Users.Create do
  alias Rockelivery.{Repo, User}

  def call(params) do
    params
    |> User.changeset()
    |> Repo.insert()
    |> handle_insert()
  end

  defp handle_insert({:ok, %User{}} = result), do: result

  defp handle_insert({:error, result}) do
    {:error, %{status: :bad_request, result: result}}
  end
end
```

Em `fallback_controller.ex`, vamos receber esse mapa contendo `status` e `result`. E também mudamos o render para `error.json`

```elixir
  def call(conn, {:error, %{status: status, result: result}}) do
    conn
    |> put_status(status)
    |> put_view(ErrorView)
    |> render("error.json", result: result)
  end
```

E finalmente, renomeamos em `error_view.ex` para `error.json`

```elixir
  def render("error.json", %{result: %Changeset{} = changeset}) do
    %{message: translate_errors(changeset)}
  end
```
