# Gear-to-Gear (g2g) Communication

**Note:** This page is being updated for OSS release. Please be patient.

- Antikythera defines an interface for g2g communication.
  The interface closely resembles the HTTP protocol: it has method, path, body, headers, etc.
  This introduces a few nice things:
    - Gear developers can focus on standard HTTP terminology. They can be written/tested in the same way.
    - The same interface enhances code reuse: e.g. code for web request automatically implements gear request handling.
- Antikythera routes a g2g request to its target gear's controller action by matching its method and path, as is the case for web request
  (see also [routing](https://hexdocs.pm/antikythera/routing.html)).
- You can send request to another gear by using `TargetGear.G2g` module.
  For usage of `G2g` modules, refer to [test code in testgear](https://github.com/access-company/testgear/tree/master/test/g2g_test.exs).

## Avoiding unnecessary body decoding/encoding

Suppose you are implementing a gear controller action that acts as a simple proxy to another gear,
i.e., returns a g2g response returned by another gear's action as it is.
You would implement the action as follows (assuming JSON encoded body):

```elixir
# Bad: don't do this
def action_xyz(conn) do
  g2g_res = AnotherGear.G2g.send(conn)
  json(conn, 200, g2g_res.body)
end
```

This implementation involves unnecessary decoding/encoding of `g2g_res.body` because

- `G2g.send/{1,2}` automatically decodes the response body, and
- `Antikythera.Conn.json/3` encodes the body.

Instead you should bypass the decoding and re-encoding using `G2g.send_without_decoding/{1,2}`.

```elixir
# Good
def some_action(conn) do
  g2g_res = AnotherGear.G2g.send_without_decoding(conn)
  conn
  |> Conn.put_status(200)
  |> Conn.put_resp_body(g2g_res.body)
  |> Conn.put_resp_headers(g2g_res.headers)
end
```
