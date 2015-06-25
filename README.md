[![Build status](https://ci.appveyor.com/api/projects/status/h2rb5mjk50u2n8hy?svg=true)](https://ci.appveyor.com/project/alexvictoor/mock4net)

# Mock4Net
Mock4Net allows to get an HTTP server in a glance. No official release yet but a  prerelease version is available on nuget.org:
```
PM> Install-Package Mock4Net.Core -Pre
```
A fluent API allows to specify the behavior of the server and hence easily stub and mock webservices and REST ressources.
```
server = FluentMockServer.Start();
server
  .Given(
    Requests.WithUrl("/*")
  )
  .RespondWith(
    Responses
      .WithStatusCode(200)
      .WithBody(@"{ msg: ""Hello world!""}")
  );
```

Based on class HttpListener from the .net framework, it is very lightweight and have no external dependencies. 

# Mock4Net API in a nutshell

## Start a server
First thing first, to start a server it is as easy as calling a static method, and your done!
```
server = FluentMockServer.Start();
```
You can pass as an argument a port number but if you do not an available port will be chosen for you. 
An available port will be chosen for you. To know on which port your server is listening, just use the property *Port*.

## Configure routes and behaviors
By default the server returns a dummy message with a 404 status code for all requests. To define a route, you need to specify request for this route and the response you want the server to return. This can be done in a fluent way using classes *Requests* and *Responses* as shown in the example below:
```
server
  .Given(
    Requests.WithUrl("/api").UsingGet()
  )
  .RespondWith(
    Responses
      .WithStatusCode(200)
      .WithBody(@"{ result: ""Some data""}")
  ); 
```

The above example is pretty simple. You can be much more selective routing requests according to url patterns, HTTP verbs, request headers and also body contents. Regarding responses, you can specify response headers, status code and body content.
Below a more exhaustive example:
```
server
  .Given(
    Requests
      .WithUrl("/api")
      .UsingPost()
      .WithHeader("X-version", "42")
      .WithBody("   { msg: \"Hello Body, I will be stripped anyway!!\" }   ");
  )
  .RespondWith(
    Responses
      .WithStatusCode(200)
      .WithHeader("content-type", "application/json")
      .WithBody(@"{ result: ""Some data""}")
  ); 
```

## Verify interactions
The server keeps a log of the received requests. You can use this log to verify the interactions that have been done with the server during a test.  
To get all the request received by the server, you just need to read property *RequestLogs*:
```
var allRequests = server.RequestLogs;
```
If you need to be more specific on the requests that have been send to the server, you can use the very same fluent API that allows to define routes:
```
var customerReadRequests 
  = server.SearchLogsFor(
    Requests
      .WithUrl("/api/customer*")
      .UsingGet()
  ); 
```

# Mock4Net with your favourite test framework

Obviously you can use your favourite test framework and use Mock4Net within your tests. In order to avoid flaky tests you should:
  - let Mock4Net choose dynamicaly ports. It might seem common sens, avoid hard coded ports in your tests!
  - clean up the request log or shutdown the server at the end of each test

Below a simple example using Nunit and NFluent test assertion library:
```
[SetUp]
public void StartMockServer()
{
    _server = FluentMockServer.Start();
}

[Test]
public async void Should_respond_to_request()
{
    // given
    _sut = new SomeComponentDoingHttpCalls();

    _server
        .Given(
            Requests
                .WithUrl("/foo")
                .UsingGet())
        .RespondWith(
            Responses
                .WithStatusCode(200)
                .WithBody(@"{ msg: ""Hello world!""}")
            );

    // when
    var response 
        = _sut.DoSomething();
    
    // then
    Check.That(response).IsEqualTo(EXPECTED_RESULT);
    // and optionnaly
    Check.That(_server.SearchLogsFor(Requests.WithUrl("/error*")).IsEmpty();
}

...

[TearDown]
public void ShutdownServer()
{
    _server.Stop();
}
```


# Mock4Net as a standalone process

This is quite straight forward to launch a mock server within a console application. Below a simple "main" method that takes as a parameter from the commandline a port number and then start a mock server that will respond "Hello World" on every request:
```
        static void Main(string[] args)
        {
            var port = Int16.Parse(args[0]);
            var server = FluentMockServer.Start(port);
            Console.Out.WriteLine("Server started on port " + port);
            server
                .Given(
                    Requests.WithUrl("/*")
                ).RespondWith(
                    Responses
                        .WithStatusCode(200)
                        .WithBody("Hello World!")
                );
            Console.Out.WriteLine("Press any key to stop the server!");
            Console.In.Read();
        }
```

# SSL

TBD

# Simulating Faults and delay

TBD

# Advanced usage

TBD
