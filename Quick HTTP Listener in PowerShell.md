# Quick HTTP Listener in PowerShell

This article is helpful if you want to quickly listen for HTTP requests, inspect them, and respond to them in real time. That means you can decide what request data to explore and what response data to send on the fly!

This could be useful when testing webhooks, dependent web services, or anything else HTTP.

## Prerequisites[](https://drakelambert.dev/2021/09/Quick-HTTP-Listener-in-PowerShell.html#prerequisites)

[Install PowerShell.](https://docs.microsoft.com/en-us/powershell/scripting/install/installing-powershell)

As far as I can tell, all PowerShell versions should be supported.

## Steps[](https://drakelambert.dev/2021/09/Quick-HTTP-Listener-in-PowerShell.html#steps)

This solution is made possible by the .NET [`HttpListener`](https://docs.microsoft.com/en-us/dotnet/api/system.net.httplistener).

### 1\. Create and Start the HTTP Listener[](https://drakelambert.dev/2021/09/Quick-HTTP-Listener-in-PowerShell.html#1-create-and-start-the-http-listener)

```
$httpListener = New-Object System.Net.HttpListener
$httpListener.Prefixes.Add('http://localhost:5001/')
$httpListener.Start()
```

Here, I’m listening for requests aimed at `http://localhost:5001/`, but you could listen to any other interface or port on your machine.

Listen on all interfaces by using a `+`, like so: `http://+:5001/`

[`HttpListener`](https://docs.microsoft.com/en-us/dotnet/api/system.net.httplistener) requires that you include a trailing `/` in the prefix.

### 2\. Wait for a Request to Come In[](https://drakelambert.dev/2021/09/Quick-HTTP-Listener-in-PowerShell.html#2-wait-for-a-request-to-come-in)

```
$context = $httpListener.GetContext()
```

In the above, `$context` will contain both the request and response object.

The `HttpListener.GetContext()` method synchronously blocks until a request is received.

### 3\. Trigger the HTTP Request Elsewhere[](https://drakelambert.dev/2021/09/Quick-HTTP-Listener-in-PowerShell.html#3-trigger-the-http-request-elsewhere)

In a separate PowerShell session:

```
Invoke-WebRequest 'http://localhost:5001/big-test'
```

You could kick off a request from anywhere though. It doesn’t have to be from PowerShell or even from your machine.

### 4\. Inspect the Request[](https://drakelambert.dev/2021/09/Quick-HTTP-Listener-in-PowerShell.html#4-inspect-the-request)

Barring network or firewall issues, the command in step 2 should complete once the request is received.

Now, you have all the time in the world to read every property of the request!

```
$context.Request.HttpMethod
$context.Request.Url
$context.Request.Headers.ToString() # pretty printing with .ToString()

# use a StreamReader to read the HTTP body as a string
$requestBodyReader = New-Object System.IO.StreamReader $context.Request.InputStream
$requestBodyReader.ReadToEnd()
```

### 5\. Send a Response[](https://drakelambert.dev/2021/09/Quick-HTTP-Listener-in-PowerShell.html#5-send-a-response)

If the calling service hasn’t timed out on you yet, you can send back a response!

```
$context.Response.StatusCode = 200
$context.Response.ContentType = 'application/json'

$responseJson = '{"big": "test"}'
$responseBytes = [System.Text.Encoding]::UTF8.GetBytes($responseJson)
$context.Response.OutputStream.Write($responseBytes, 0, $responseBytes.Length)

$context.Response.Close() # end the response
```

You can call `.Write(...)` multiple times before calling `.Close()`.

### 6\. Clean Up[](https://drakelambert.dev/2021/09/Quick-HTTP-Listener-in-PowerShell.html#6-clean-up)

Release the open TCP port:

## What Else Can `HttpListener` Do?[](https://drakelambert.dev/2021/09/Quick-HTTP-Listener-in-PowerShell.html#what-else-can-httplistener-do)

- HTTPS
- Authentication: Basic, Digest, Windows, Negotiate, and NTLM

For all this, check out [the docs](https://docs.microsoft.com/en-us/dotnet/api/system.net.httplistener#remarks).

If you find this post useful, and wish to support it, you can below!

[![Buy Me A Coffee](https://cdn.buymeacoffee.com/buttons/v2/default-blue.png)](https://www.buymeacoffee.com/drakelambert)
