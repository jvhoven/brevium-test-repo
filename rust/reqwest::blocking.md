A blocking Client API.

The blocking `Client` will block the current thread to execute, instead
of returning futures that need to be executed on a runtime.

This requires the optional `blocking` feature to be enabled.

For a single request, you can use the [`get`](../../reqwest/blocking/fn.get.html) shortcut method.

```
letbody=reqwest::blocking::get("https://www.rust-lang.org")?
    .text()?;

println!("body = {:?}", body);
```

Additionally, the blocking [`Response`](../../reqwest/blocking/struct.Response.html) struct implements Rust's`Read` trait, so many useful standard library and third party crates will
have convenience methods that take a `Response` anywhere `T: Read` is
acceptable

**NOTE**: If you plan to perform multiple requests, it is best to create a[`Client`](../../reqwest/blocking/struct.Client.html) and reuse it, taking advantage of keep\-alive connection
pooling.

There are several ways you can set the body of a request. The basic one is
by using the `body()` method of a [`RequestBuilder`](../../reqwest/blocking/struct.RequestBuilder.html). This lets you set the
exact raw bytes of what the body should be. It accepts various types,
including `String`, `Vec<u8>`, and `File`. If you wish to pass a custom
Reader, you can use the `reqwest::blocking::Body::new()` constructor.

```
letclient=reqwest::blocking::Client::new();
letres=client.post("http://httpbin.org/post")
    .body("the exact body that is sent")
    .send()?;
```

Most features available to the asynchronous `Client` are also available,
on the blocking `Client`, see those docs for more.

|--------------|----------------------------------------------------------------------|
|     Body     |                        The body of aRequest.                         |
|    Client    |                    AClientto make Requests with.                     |
|ClientBuilder |AClientBuildercan be used to create aClientwith  custom configuration.|
|   Request    |        A request which can be executed withClient::execute().        |
|RequestBuilder|          A builder to construct the properties of aRequest.          |
|   Response   |                  A Response to a submittedRequest.                   |

|---|----------------------------------------------------|
|get|Shortcut method to quickly make ablockingGETrequest.|
