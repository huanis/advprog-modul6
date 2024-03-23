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