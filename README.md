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