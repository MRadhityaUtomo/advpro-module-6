# MODULE 6
## REFLECTION

1.  [Commit] Add reflection notes in Readme.md, put the title clearly
such as Commit 1 Reflection notes. commit your works, put
commit message “(1) Handle-connection, check response”, and
then push it to your repository. 

### Commit 1 Reflection Notes 
Describing the method by breaking it down:
- `fn handle_connection(mut stream: TcpStream)`: This function takes a mutable reference to a TcpStream as its parameter. The mut keyword enable the stream to be modified within the function.
- `let buf_reader = BufReader::new(&mut stream);`: It creates a BufReader instance, which buffers input from the provided TcpStream. The BufReader is used for efficient reading from the stream.
- Based on the documentation provided for Rust, BufReaderA BufReader<R> performs large, infrequent reads on the underlying Read and maintains an in-memory buffer of the results BufReader<R> can improve the speed of programs that make small and repeated read calls to the same file or network socket.
- `let http_request: Vec<_> = buf_reader.lines()...`: This line reads lines from the buffered input using the lines() method provided by BufReader. It maps each line to a String, collecting them into a Vec<String> named http_request.
- `.take_while(|line| !line.is_empty())`: This part of the code uses take_while to stop reading lines once an empty line is encountered. It stops processing the request once an empty line is reached, assuming it marks the end of the HTTP request header.
- `.collect();`: This collects the lines into a Vec<String> named http_request.
- `println!("Request: {:#?}", http_request);`: Finally, it prints out the HTTP request header for debugging purposes, showing the content of the http_request vector.  

2. [Commit] Complete your reflection on the new handle_connection
in the readme.md, put the title clearly such as Commit 2
Reflection notes. commit with message “(2) Returning HTML”,
push it to your git repository server.

### Commit 2 Reflection Notes 

![Commit 2 screen capture](/assets/images/commit2.png)

After modifications are made to the `handle_connection` method, it can now respond requests from the TCP stream.

Based on the Rust documentation:
- `fs` is used to bring the standard library's filesystem module into scope.
- `format!` is used to add the file’s contents as the body of the success response. To ensure a valid HTTP response, we add the `Content-Length` header which is set to the size of our response body, in this case the size of `hello.html`.

The process includes sending a line response that validates a success with the code `200 OK`. Once it detects an OK it will read the html file, in this case `hello.html` and as stated above, it's size is counted and put in header response.

3. [Commit] Add additional reflection notes, put the title clearly such
as Commit 3 Reflection notes. Commit your work with message
“(3) Validating request and selectively responding”. Push your
commit. 

### Commit 3 Reflection Notes 

Currently, the web will return `hello.html` regardless of the request sent.

We can simply divide the desired request by using `if` and `else` blocks. Basically, if the request is visibile, and has a response with either a render of an html file or some other way do this, else do this.

Based on the Rust Documentation: 

```rust
let buf_reader = BufReader::new(&mut stream);
let request_line = buf_reader.lines().next().unwrap().unwrap();

if request_line == "GET / HTTP/1.1" {
    let status_line = "HTTP/1.1 200 OK";
    let contents = fs::read_to_string("hello.html").unwrap();
    let length = contents.len();

    let response = format!(
        "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
    );

    stream.write_all(response.as_bytes()).unwrap();
} else {
    let status_line = "HTTP/1.1 404 NOT FOUND";
    let contents = fs::read_to_string("404.html").unwrap();
    let length = contents.len();

    let response = format!(
        "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
    );

    stream.write_all(response.as_bytes()).unwrap();
}
```

`request_line` being the request sent and checked by the if-else blocks, and status line being the response given to that request.
for this module, I made a `404.html` as in "404 page not found" to be a response to unfound requests.

`404.html` :
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>Hello!</title>
  </head>
  <body>
    <h1>Oops!</h1>
    <p>Sorry, your request was not found.</p>
  </body>
</html>
```
with the end result by accessing `http://127.0.0.1:7878/bad` being:
![Commit 3 screen capture](/assets/images/commit3_notfound.jpg)

Next we should refactor the code to make it more readable and clean.
- Conjoin the 2 if-else blocks and seperate them from their own similar code. Basically only having the `status_line` for the response
- Extract methods after the first point to clean up the format of the code by removing unnecessary lines.
- Overall improve maintanability and achieving the ability for modification of each if-else statements easily

resulting code: 
```rust
fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let request_line = buf_reader.lines().next().unwrap().unwrap();

    let (status_line, filename) = if request_line == "GET / HTTP/1.1" {
        ("HTTP/1.1 200 OK", "hello.html")
    } else {
        ("HTTP/1.1 404 NOT FOUND", "404.html")
    };

    let contents = fs::read_to_string(filename).unwrap();
    let length = contents.len();

    let response =
        format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");

    stream.write_all(response.as_bytes()).unwrap();
}
```

4. [Commit] Add additional reflection notes, put the title clearly such
as Commit 4 Reflection notes. Commit your work with message
“(4) Simulation of slow request. “

### Commit 4 Reflection Notes 

Currently the server is running on single thread, meaning with enough abundant of users accessing the web, it will suffer negatives and slow down.

We can simulate a slowing down event by integrating sleep to the thread.
- By changing the `if` block in `handle_connection` to `match` to "match" the value of `request_line` with the desired endpoints (`/`,`/sleep`,and none).
- The new `/sleep` endpoint has the `thread::sleep(Duration::from_secs(5));`. This is how a single-threaded server handles a multi-threaded process by delaying the process by 5 seconds.

With this, by opening 2 browsers are opened and repectively access `127.0.0.1/sleep` and `127.0.0.1`, the server will delay other responses thus blocking other a access to the servers with a single-thread.


5. [Commit] Add additional reflection notes, put the title clearly such
as Commit 5 Reflection notes. Commit your work with message
“(5) Multithreaded server using Threadpool “

### Commit 5 Reflection Notes 

We can make a multithreading system by creating a ThreadPool.
A ThreadPool is as per what it says, a pool of threads with each thread ready to handle a request.

Next we need a Worker that accepts and runs more specific jobs. We can connect the ThreadPool and Worker by making a new code that makes the ThreadPool able to send a signal through sender to receiver that has been cloned and assigned to each Worker. This enables when the ThreadPool accepts a request, the signal will be sent and assign the request to a valid Worker which will then process the request. Worker then locks the receiver to process the data until it's finished, and only then will the lock be unlocked and enables other Workers to accept other jobs.