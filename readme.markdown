特别注意
===
项目是.net4.7.1以上的版本。我需要的是.net4.0的版本，因此对版本.net的版本进行降级。这个项目用到.net4.7.1的RuntimeInformation的API:
```
if (RuntimeInformation.IsOSPlatform(OSPlatform.Windows))
{
    socket.SetSocketOption(SocketOptionLevel.IPv6, SocketOptionName.IPv6Only, false);
}
```

我的项目只在window上运行，因此直接去除判断。

Fleck
===

[![Build status](https://ci.appveyor.com/api/projects/status/k0s8hq5y4emak5j3/branch/master?svg=true)](https://ci.appveyor.com/project/statianzo/fleck/branch/master) [![NuGet](https://img.shields.io/nuget/v/Fleck.svg)](https://www.nuget.org/packages/Fleck/)

Fleck is a WebSocket server implementation in C#. Branched from the
[Nugget][nugget] project, Fleck requires no inheritance, container, or
additional references.

Example
---

The following is an example that will echo to a client.

```c#

var server = new WebSocketServer("ws://0.0.0.0:8181");
server.Start(socket =>
{
  socket.OnOpen = () => Console.WriteLine("Open!");
  socket.OnClose = () => Console.WriteLine("Close!");
  socket.OnMessage = message => socket.Send(message);
});
        
```

Supported WebSocket Versions
---

Fleck supports several WebSocket versions of modern web browsers

- Hixie-Draft-76/Hybi-00 (Safari 5, Chrome < 14, Firefox 4 (when enabled))
- Hybi-07 (Firefox 6)
- Hybi-10 (Chrome 14-16, Firefox 7)
- Hybi-13 (Chrome 17+)

Secure WebSockets (wss://)
---

Enabling secure connections requires two things: using the scheme `wss` instead
of `ws`, and pointing Fleck to an x509 certificate containing a public and
private key

```cs
var server = new WebSocketServer("wss://0.0.0.0:8431");
server.Certificate = new X509Certificate2("MyCert.pfx");
server.Start(socket =>
{
  //...use as normal
});
```

SubProtocol Negotiation
---

To enable negotiation of subprotocols, specify the supported protocols on
the `WebSocketServer.SupportedSubProtocols` property. The negotiated
subprotocol will be available on the socket's `ConnectionInfo.NegotiatedSubProtocol`.

If no supported subprotocols are found on the client request (the
Sec-WebSocket-Protocol header), the connection will be closed.

```cs
var server = new WebSocketServer("ws://0.0.0.0:8181");
server.SupportedSubProtocols = new []{ "superchat", "chat" };
server.Start(socket =>
{
  //socket.ConnectionInfo.NegotiatedSubProtocol is populated
});
```

Custom Logging
---

Fleck can log into Log4Net or any other third party logging system. Just override the `FleckLog.LogAction` property with the desired behavior.

```cs
ILog logger = LogManager.GetLogger(typeof(FleckLog));

FleckLog.LogAction = (level, message, ex) => {
  switch(level) {
    case LogLevel.Debug:
      logger.Debug(message, ex);
      break;
    case LogLevel.Error:
      logger.Error(message, ex);
      break;
    case LogLevel.Warn:
      logger.Warn(message, ex);
      break;
    default:
      logger.Info(message, ex);
      break;
  }
};

```

Disable Nagle's Algorithm
---

Set `NoDelay` to `true` on the `WebSocketConnection.ListenerSocket`

```cs
var server = new WebSocketServer("ws://0.0.0.0:8181");
server.ListenerSocket.NoDelay = true;
server.Start(socket =>
{
  //Child connections will not use Nagle's Algorithm
});
```

Auto Restart After Listen Error
---

Set `RestartAfterListenError` to `true` on the `WebSocketConnection`

```cs
var server = new WebSocketServer("ws://0.0.0.0:8181");
server.RestartAfterListenError = true;
server.Start(socket =>
{
  //...use as normal
});
```

[nugget]: http://nugget.codeplex.com/ 
