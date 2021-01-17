+++
authors = ["Josep Jesus Bigorra Algaba"]
date = 2020-11-13T23:00:00Z
excerpt = "Wondering what to do with the dreaded too many open files error in your Go application?"
hero = "https://res.cloudinary.com/dehs6irlh/image/upload/v1605355422/jjba-site/blog/go-file-descriptors/file-descriptor_azndem.jpg"
timeToRead = 5
title = "Go File Descriptors"

+++
Ever heard of _dangling pointers_? Have you ever left your database connections without closing them properly?

One must pay special attention to this specially when dealing with lower level languages like Go, C, and C++.

Resource leaks are really a do or die in many applications, something very often and very easily overlooked. I personally have had applications catastrophically crash due to this. Read this to understand useful patterns and practices to avoid these problems in Go..

## A true story!

Last week I saw one of my beloved Go applications crash catastrophically with such an error:

```sh
server.go:3095: http: Accept error: accept tcp [::]:7002: accept4: too many open files; retrying in 5ms 
```
   

This error kept repeating for a good few minutes, until I luckily spotted the issue, and temporarily fixed it by restarting the application (Go binary as system service).

The application in question deals with a lot of incoming and outgoing HTTP requests, acting as a Facade between several services.

Due to the nature of this application, many requests of all kinds are sent, and sometimes I do not care about the response body of a performed request, only about the returned status code, or not even that.

This lead to a situation where I performed several requests ignoring the response body with the `_` variable.

So please follow my advice, always, ALWAYS, close the response body of a request.

**BAD**

```go
_, err := client.Do(request)
if err != nil {
  log.Println(err.Error())
  return nil
} 
```
    

**GOOD**

```go
resp, err := client.Do(request)
if err != nil {
  log.Println(err.Error())
  return nil
}

defer resp.Body.Close()
```

        
If you do not close the response body, or for that matter defer its closing, the resources allocated for that will never be released, and thus your application will keep it indefinitely reserved, in Unix systems, in a file descriptor.

Keep in mind, that the default limit per application is of 1024 file descriptors. this can be manipulated, but in my opinion really should not be done for web services, unless there is an actual necessity. Keep it in the back of your head, if you need to increase this, you most certainly have a leak in your program.

The bad news is that not only HTTP requests will cause this issue. Database connections, Redis connections, Buffered readers, Actual File I/O operations...

A solid example, is indeed the database connection:

**BAD**

```go
db, err := sql.Open(databaseType, connection)
if err != nil {
  log.Println(err.Error())
  panic(err.Error())
}

// If you do not defer close to the connection
// you will dangerously forget about doing so.
```

**GOOD**

```go
db, err := sql.Open(databaseType, connection)
if err != nil {
  log.Println(err.Error())
  panic(err.Error())
}

defer db.Close()
// Use db here, no risk of forgetting to close the db connection  
```  

Anything that you have opened and forgot to close, will hog the system and bring the file descriptor count upwards, and will come back to haunt you. For that reason, you should check your system services file descriptor count sometimes. This can easily be achieved with a couple commands. Log in as root user in your Unix server and do a `ps aux`.

You might be wondering, Go is a Garbage Collected (GC) language, why doesn't it simply destroy these unused connections/handles/response bodies?

This simply is not the case since for the GC these handles are still "referenced" and thus are not eligible for garbage collection. They are basically dangling around, and you can avoid this bad situation with a simple `defer` statement.

Imagining your application is called `my-application` and is located at `/internalApplications/my-application/app`, you should filter the `ps aux`output to find:

```sh
root     531  0.0  0.1 1323436 8000 ?        Ssl  Nov12   0:35 /internalApplications/my-application/app     
```

What we are intereseted in here, is the 2nd column and 11th column, which contain respectively the PID of the process and the name of the process.

To get a file descriptor count per PID, do `ls /proc/{pid}/fd | wc -l`, in this case you would need to do, `ls /proc/531/fd | wc -l`

You can take this script I made, that finds all services that are located in the `/internalApplications` folder and modify it, to suit your needs:

```sh
#!/bin/bash
pid=( $(ps aux | grep /internalApplications/ | grep -v "grep" | awk '{print $2}') )
processName=( $(ps aux | grep /internalApplications/ | grep -v "grep" | awk '{print $11}') )

for index in ${!pid[*]};  do
    printf "%s --> %s --> %s\n" "${pid[$index]}" "$( ls /proc/${pid[$index]}/fd | wc -l )"  "${processName[$index]}"
done 
```   

Other good tips to avoid file descriptors being hogged are more related to HTTP requests. If your application is exposed to other HTTP services, ensure you follow the below tips.

Always add a request close, unless you plan to have `keep-alive` connections. This can easily be done:

```go
req, err := http.NewRequestWithContext(
  context.Background(),
  "POST",
  "www.example.com",
  bytes.NewBuffer([]byte(`{"test": "example"}`)),
)
if err != nil {
  log.Println(err.Error())
  return nil, err
}

req.Header.Add("Content-Type", "application/json")
req.Header.Add("Charset", "utf-8")
// This will add a Connection: close header to the request
req.Close = true
// alternatively
req.Header.Add("Connection", "close")  
```  

Set a default timeout for HTTP requests, to avoid hanging requests, or malicious attempts:

```go
// Do this at the beggining of your application, in your `main.go`
http.DefaultClient.Timeout = 30 * time.Second   
```   

If you use `http.Server` directly then make sure to set some sensible timeouts:

```go
srv := &http.Server{
  ReadTimeout:  10 * time.Second,
  WriteTimeout: 30 * time.Second,
  IdleTimeout:  100 * time.Second,
}

// optional disable keep alives
srv.SetKeepAlivesEnabled(false) 
```   

If you use `http.Client` directly then also make sure to set a sensible timeout:

```go
client := &http.Client{Timeout: 30 * time.Second}
```

In conclusion, you should always take special care with closing resources properly so as to not have stale sockets, and file descriptors in your application.

Always cleanup after yourself, and if possible, do a `defer whatever.Close()` as close to the declarations as possible, so as to not forget to do it further down the code. This also avoids resources not being closed due to early returns.

With great power, such as what Go gives developers, comes great responsability.

I will keep expanding this article as I find more useful tips for this, stay tuned!