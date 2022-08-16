---
title: Introducing SynSharp
tags: synsharp synapse threat-intel
---

[Synapse](https://vertex.link/synapse) is a collaborative cyber threat
intelligence platform provided by Vertex. The platform allows the analyst to
share a common database to perform their analysis and reporting.

In my opinion, it is the best platform available for cyber threat intelligence
analysis. The two main aspects that motivates my opinion are: the data and
analytic model. Synapse comes with a pre-defined data model that provides a
very large ontology of elements useful in the analysis. These type comes with
normalization and helpers so that you have less work to do and can focus on the
analysis itself. The analytical model takes the form of a tag hierarchy that
clearly separates facts (modelled with the data model) from assessments
(modelled with the analytical model). It is especially important when you work
with many analysts, including junior ones.

While the core is open source, the user interface, called Optic, and
many of the helpers commands, called power-ups, are only available in their
commercial offer.

Because I wanted to integrate Synapse within other projects written in C#, I
developped a client to interact with a Synapse sever. The client is dubbed
SynSharp and is available on [GitHub](https://github.com/ancailliau/SynSharp)
under an Apache Licence 2.0.

To connect to a server, you can simply use the client while providing the
connection url and the credentials.

```csharp
SynapseClient = new SynapseClient("https://localhost:8901", logger);
await SynapseClient.LoginAsync("root", "secret");
```

The logger is a simple `ILogger<SynapseClient>` that can be built with a
`NullLogger` or with
   
```csharp
using var loggerFactory = LoggerFactory.Create(builder =>
{
	builder.AddConsole();
});
var logger = loggerFactory.CreateLogger<SynapseClient>();
```

Once the client is ready, you can start querying the server. For example, to
add the IP address `8.8.8.8` into the system, you use the bracket syntax for
storm, that is:
   
```csharp
var response = await SynapseClient
            .StormAsync<InetIpV4>("[ inet:ipv4=8.8.8.8 ]")
            .ToListAsync();
```

The variable `response` now contains a Synapse node representing the IP address:

```
response.First().Equals(IPAddress.Parse("8.8.8.8"))
```

You can find many other examples in the
[repository](https://github.com/ancailliau/SynSharp/tree/master/Synsharp.Tests).
