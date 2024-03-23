# advprog-modul6
## Reflections
### COMMIT 1: handle-connection, check response
The `handle_connection` function wraps the input stream with a buffer reader. It then reads the stream per line (separated by newlines) through the reader. Then, the value from the line is extracted. Only non-empty lines will be collected. They are referred by the `http_request` variable. Then, the function will print what is referred by `http_request` variable.
```
Request: [
    "GET / HTTP/1.1",
    "Host: 127.0.0.1:7878",
    "Connection: keep-alive",
    "sec-ch-ua: \"Chromium\";v=\"122\", \"Not(A:Brand\";v=\"24\", \"Google Chrome\";v=\"122\"",
    "sec-ch-ua-mobile: ?0",
    "sec-ch-ua-platform: \"Windows\"",
    "Upgrade-Insecure-Requests: 1",
    "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Safari/537.36",
    "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7",
    "Sec-Fetch-Site: cross-site",
    "Sec-Fetch-Mode: navigate",
    "Sec-Fetch-Dest: document",
    "Accept-Encoding: gzip, deflate, br, zstd",
    "Accept-Language: en-US,en;q=0.9",
]
```
### COMMIT 2: return html response
At first, the `handle_connection` function does the same thing as before (commit 1). However, instead of just printing the `http_request`, it constructs and sends an http response. It first defines the http status line for a successful response, which is "HTTP/1.1 200 OK" in the variable `status_line`. Then, it reads the content of the file named `hello.html` in the root directory as a string to the variable named `contents`. After that, the length of said content is calculated, stored into the variable named `length`. The response is then constructed with the format `"{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"`. Then, the function sends the response to the client through the TCP stream.
![Commit 2 screen capture](/assets/images/commit2.png)

### COMMIT 3: validating request and selectively responding
Before, when you go to any URL, as long as the host is the same (in this case, the host: "127.0.0.1:7878"), the server will return `hello.html` as the content's response. However, we now differentiate between get requests through the url "{host}/" and other urls (example: "{host}/good" or "{host}/bad"). We can differentiate them by looking at the first line of the client's request. If the client uses "{host}/" url, the first line will be `"GET / HTTP/1.1"`. Otherwise, if the client uses, for example, "{host}/bad" url, the first line will be `"GET /bad HTTP/1.1"` ("{host}/good" -> `"GET /good HTTP/1.1"`). As you can see, the address used is defined there. For requests other than `"GET / HTTP/1.1"`, the server will return the contents of `404.html`.
```rust
// Before Refactoring
fn handle_connection(mut stream: TcpStream) {
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
}
```
We did refactoring to the above code as all lines are duplicated lines in the if and else blocks. The only differences between the two blocks are the status_line and the html file used as the response' content. We extract the duplicated lines from the if-else block. Inside the if-else blocks, they only define the status_line and html file name. The reason we needed to refactor this is in consideration that if duplicated code contains a bug, the same bug appears multiple times. Therefore, duplicated code is takes more time to maintain as we need to repeat the same fix multiple times.
![Commit 3 screen capture](/assets/images/commit3.png)

### COMMIT 4:  simulation of slow request
Another condition is added to the code from the last commit. This time, when the request line is `"GET /sleep HTTP/1.1"`, then the thread will sleep for 10 second before proceeding to make the response and send it to the client. Note that the program only uses 1 thread. This meant that if there are two requests at the same time, one of them will have to wait for the first one in queue to be finished. This is why when you send a request to `"GET /sleep HTTP/1.1"` before `"GET / HTTP/1.1"`, the one with `"GET / HTTP/1.1"` takes a long time to be processed since it has to wait for `"GET /sleep HTTP/1.1"` to finish up.

### COMMIT 5: multithreaded server using threadpool
Considering the problem from last commit, it's wise to consider multithreading. In this case, a process can have multiple threads, thus if there are more than 2 threads and there are 2 requests, each thread could take a request. If the first request takes too long to be processed, the second one could be processed by another thread. In this code, we use a thread pool. Basically, we define a fixed number of threads, ready to take on tasks. In this case, we define 4 threads in a pool. When there are 5 requests sent simultaneously, 4 will be taken by the threads while 1 needs to wait for one of the threads to finish their job. We implemented a worker as data structure that stands as a bridge between the `ThreadPool` and the threads. The worker will run the job they take in the worker's thread. The channel will be the queue of jobs waiting to be taken by a worker.
![Commit 5 screen capture](/assets/images/commit5.png)