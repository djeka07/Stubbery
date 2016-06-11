# Stubbery

Simple library for creating and running Api stubs in .NET.

## Introduction

In many situations it comes handy if we're able to start a simple service that responds on certain routes with preconfigured static responses.

This is particularly important in integration testing, when we might want to replace some of our dependencies with a stub that can reliably provide the expected responses.

**Stubbery** is a library with which we can simply configure and start a web server that responds on particular routes with the configured results.  
It supports .NET Core and the full .NET Framework up from .NET 4.5.1.

## How to use

The central class of the library is `ApiStub`. In order to start a new server we have to create an instance of `ApiStub`, set up some routes using the methods `Get`, `Post`, `Put` and `Delete`, and start the server by calling `Start`.

The server listens on `localhost` on a randomly picked free port. The full address is returned by the `Address` property.

After usage the server should be stopped to free up the TCP port. This can be done by calling `Dispose` (or use the stub in a `using` block).

### Basic usage

```
using (var sut = new ApiStub())
{
    sut.Get(
	"/testget",
	(req, args) => "testresponse");

    sut.Start();

    var result = await httpClient.GetAsync(new UriBuilder(new Uri(sut.Address)) { Path = "/testget" }.Uri);

    // resultString will contain "testresponse"
    var resultString = await result.Content.ReadAsStringAsync();
}
```

### Accessing the request parameters

Parameters of the request can be accessed through the second argument of the lambda setting up the response. The following code sample shows how the route and the query string parameters can be accessed.

```
using (var sut = new ApiStub())
{
    sut.Get(
	"/testget/{arg1}",
	(req, args) => $"testresponse arg1: {args.Route.arg1} queryArg1: {args.Query.queryArg1}");

    sut.Start();

    var result = await httpClient.GetAsync(
	new UriBuilder(new Uri(sut.Address)) { Path = "/testget/orange", Query = "?queryArg1=melon"}.Uri);

    // resultString will contain "testresponse arg1: orange queryArg1: melon"
    var resultString = await result.Content.ReadAsStringAsync();
}
```

The following example shows how the HTTP body can be accessed.

```
using (var sut = new ApiStub())
{
    sut.Post(
	"/testpost",
	(req, args) => $"testresponse body: {args.Body.ReadAsString()}");

    sut.Start();

    var result = await httpClient.PostAsync(
	new UriBuilder(new Uri(sut.Address)) { Path = "/testpost" }.Uri,
	new StringContent("orange"));

    // resultString will contain "testresponse body: orange"
    var resultString = await result.Content.ReadAsStringAsync();
}
```

### Other verbs

If we want to use a different HTTP verb, the `Setup` method can be used.

```
using (var sut = new ApiStub())
{
    sut.Setup(
	HttpMethod.Options,
	"/testoptions",
	(req, args) => "testresponse");

    sut.Start();

    var result = await httpClient.SendAsync(
	new HttpRequestMessage
	{
	    RequestUri = new UriBuilder(new Uri(sut.Address)) { Path = "/testoptions" }.Uri,
	    Method = HttpMethod.Options
	});

    // resultString will contain "testresponse"
    var resultString = await result.Content.ReadAsStringAsync();
}
```