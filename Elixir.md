# Elixir

## Mocks

```elixir
defmodule Textmewhen.V1.NumberControllerTest do
  use Textmewhen.ConnCase, async: false
  
  import Mock

  alias Textmewhen.Number

  @valid_attrs %{phone_number: "9519020972"}
  @invalid_attrs %{}

  setup %{conn: conn} do
    app = Repo.insert!(Textmewhen.OauthApplication.changeset(%Textmewhen.OauthApplication{}, %{name: "App", redirect_uri: "a://b", scopes: "all"}))

    conn = conn
    |> put_req_header("accept", "application/json")
    |> put_req_header("x-application-id", app.uid)

    {:ok, conn: conn}
  end

  test "creates and renders resource when data is valid", %{conn: conn} do
    with_mock ExTwilio.Message, [create: fn (_) -> :ok end ] do
      conn = post conn, v1_number_path(conn, :create), number: @valid_attrs
      assert json_response(conn, 201)["data"]["id"]
      assert Repo.get_by(Number, @valid_attrs)
    end
  end

  test "does not create resource and renders errors when data is invalid", %{conn: conn} do
    conn = post conn, v1_number_path(conn, :create), number: @invalid_attrs
    assert json_response(conn, 422)["errors"] != %{}
  end
end

```

Genserver:
https://blog.drewolson.org/understanding-gen-server/
