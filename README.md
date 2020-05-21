# responsible

In the wise words of the modern poet Ice Cube:

> So come on and chickity check yo self before you wreck yo self

Responsible is a library that makes it simple, fast an easy to check HTTP responses in crystal-lang.
It provides a lightweight API dealing with errors, logging and static, type-safe parsing.
It works directly with `HTTP::Client::Response` objects and can function in tandem with clients that build on top of these.


## What this isn't

Responsible is not a HTTP client.
It *does not* handle request building, execution, retries, or authentication.
It deals purely with response objects once you already have them, regardless of their source.


## Installation

1. Add the dependency to your `shard.yml`:

   ```yaml
   dependencies:
     responsible:
       github: place-labs/responsible
   ```

2. Run `shards install`

3. Load via:

   ```crystal
   require "responsible"`
   ```


## Usage

To become responsible, prefix any response or method that returns a response with `~`.
```crystal
response = ~HTTP::Client.get("https://www.example.com")
```

Error conditions are clear:
```crystal
response.on_server_error do
  # oh noes
end
```

When working with JSON, it also lets you efficiently parse to type-safe objects.
```crystal
response = ~HTTP::Client.get("https://www.example.com") >> NamedTuple(message: String)
response[:message]
# => hello world
```

---

Responsible Responses™ maintain all the existing functionality of a vanilla response object.
```crystal
chuck_norris_response = ~HTTP::Client.get("https://api.chucknorris.io/jokes/random")

chuck_norris_response.body
# =>
{
  "categories": [],
  "created_at": "2020-01-05 13:42:19.897976",
  "icon_url": "https://assets.chucknorris.host/img/avatar/chuck-norris.png",
  "id": "h6EF5PXvQoK5cW60-lUeDg",
  "updated_at": "2020-01-05 13:42:19.897976",
  "url": "https://api.chucknorris.io/jokes/h6EF5PXvQoK5cW60-lUeDg",
  "value": "Chuck Norris has a large, curved talon on each foot, much like the Velociraptor."
}
```

They also let you inject behaviour for dealing with different response scenarios.
```crystal
# Global handlers - this will apply to every reponse
Responsible.on_client_error do |response|
  raise "I'm a teapot" if status_code == 418
end

# Local action - these apply to the target response
chuck_norris_response.on_redirect do |response|
  my_request_method(response.headers["Location"])
end
```

Once you have a response that you are happy is valid, you can parse this out matching type using the `>>` operator.
```crystal
struct ChuckNorrisFact
   include JSON::Serializable
   getter created_at : Time
   getter id :  String
   getter updated_at : Time
   getter value : String
end

fact = chuck_norris_response >> ChuckNorrisFact

fact.value
# => Chuck Norris has a large, curved talon on each foot, much like the Velociraptor.
```

This also works inline for terse, type-safe, error checked response.
```crystal
fact = ~HTTP::Client.get("https://api.chucknorris.io/jokes/random") >> NamedTuple(value: String)
fact[:value]
# => Chuck Norris can binary search unsorted data.
```


## Contributors

- [Kim Burgess](https://github.com/KimBurgess) - creator and maintainer
