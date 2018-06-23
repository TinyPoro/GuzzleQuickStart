# GuzzleQuickStart

[Source](http://docs.guzzlephp.org/en/stable/quickstart.html "Permalink to Quickstart — Guzzle Documentation")

# Bắt đầu nhanh — Tài liệu Guzzle 

Trang này sẽ giới thiệu nhanh về Guzzle cùng 1 số ví dụ giới thiệu. Nếu bạn chưa cài Guzzle, hãy tới trang [Cài đặt][1].

## Tạo 1 Request

Bạn có thể gửi các request với Guzzle bằng cách sử dụng 1 đối tượng  `GuzzleHttpClientInterface`.

### Tạo 1 Client
    
    
    use GuzzleHttpClient;
    
    $client = new Client([
        // Base URI is used with relative requests
        'base_uri' => 'http://httpbin.org',
        // You can set any number of default request options.
        'timeout'  => 2.0,
    ]);
    

Các client là bất biến trong Guzzle 6, có nghĩa là bạn sẽ không thể thay đổi các mặc định được client sử dụng sau khi nó được tạo.

Việc khởi tạo client cho phép mảng liên kết các tùy chọn:

`base_uri`
: 

(string|UriInterface) URI gốc của client được nối với các URI tương đối. Có thể là 1 string hoặc là 1 thực thể UriInterface. Khi client đưa ra 1 Uri tương đối, client sẽ nối URI gốc với URI tương đối sử dụng các luật được mô tả trong [RFC 3986, section 2][2].
    
    
    // Create a client with a base URI
    $client = new GuzzleHttpClient(['base_uri' => 'https://foo.com/api/']);
    // Send a request to https://foo.com/api/test
    $response = $client->request('GET', 'test');
    // Send a request to https://foo.com/root
    $response = $client->request('GET', '/root');
    
Không cảm thấy là đã từng đọc RFC 3986? Vậy thì đây là 1 vài ví dụ nhanh về cách mà 1 `base_uri` sẽ được xử lý với URI khác.

| base_uri              | URI              | Result                   |  
| --------------------- | ---------------- | ------------------------ |  
| `http://foo.com`      | `/bar`           | `http://foo.com/bar`     |  
| `http://foo.com/foo`  | `/bar`           | `http://foo.com/bar`     |  
| `http://foo.com/foo`  | `bar`            | `http://foo.com/bar`     |  
| `http://foo.com/foo/` | `bar`            | `http://foo.com/foo/bar` |  
| `http://foo.com`      | `http://baz.com` | `http://baz.com`         |  
| `http://foo.com/?bar` | `bar`            | `http://foo.com/bar`     |  

`handler`
: (callable) 1 hàm chuyển các request HTTP trên đường dẫn. Hàm được gọi với 1 `Psr7HttpMessageRequestInterface`  và mảng các tùy chọn chuyển dời, và phải trả về 1 `GuzzleHttpPromisePromiseInterface` thỏa mãn 1 `Psr7HttpMessageResponseInterface` khi thành công. `handler` là 1 khởi tạo mà tùy chọn không thể được ghi đè trong các tùy chọn per/request

`...`
: (mixed) Tất cả các tùy chọn khác được truyền vào hàm khởi tạo được sử dụng như các tùy chọn request mặc định với mỗi request được tạo bởi client.
.

### Gửi các Requests

Các Magic method trên client khiến việc gửi các request đồng bộ rất dễ dàng:
    
    
    $response = $client->get('http://httpbin.org/get');
    $response = $client->delete('http://httpbin.org/delete');
    $response = $client->head('http://httpbin.org/get');
    $response = $client->options('http://httpbin.org/get');
    $response = $client->patch('http://httpbin.org/patch');
    $response = $client->post('http://httpbin.org/post');
    $response = $client->put('http://httpbin.org/put');
    

Bạn có thể tạo 1 request và sau đó dùng client để gửi request khi bạn sẵn sàng:
    
    
    use GuzzleHttpPsr7Request;
    
    $request = new Request('PUT', 'http://httpbin.org/put');
    $response = $client->send($request, ['timeout' => 2]);
    
`
Các đối tượng client cung cấp 1 giải pháp linh động trong cách vận chuyển các request bao gồm các tùy chọn request mặc định, các tập middleware xử lý mặc định được sử dụng bởi mỗi request, và 1 URI gốc cho phép bạn gửi các request với các URI tương đối.

You can find out more about client middleware in the [_Handlers and Middleware_][3] page of the documentation.

### Async Requests

You can send asynchronous requests using the magic methods provided by a client:
    
    
    $promise = $client->getAsync('http://httpbin.org/get');
    $promise = $client->deleteAsync('http://httpbin.org/delete');
    $promise = $client->headAsync('http://httpbin.org/get');
    $promise = $client->optionsAsync('http://httpbin.org/get');
    $promise = $client->patchAsync('http://httpbin.org/patch');
    $promise = $client->postAsync('http://httpbin.org/post');
    $promise = $client->putAsync('http://httpbin.org/put');
    

You can also use the sendAsync() and requestAsync() methods of a client:
    
    
    use GuzzleHttpPsr7Request;
    
    // Create a PSR-7 request object to send
    $headers = ['X-Foo' => 'Bar'];
    $body = 'Hello!';
    $request = new Request('HEAD', 'http://httpbin.org/head', $headers, $body);
    
    // Or, if you don't need to pass in a request instance:
    $promise = $client->requestAsync('GET', 'http://httpbin.org/get');
    

The promise returned by these methods implements the [Promises/A+ spec][4], provided by the [Guzzle promises library][5]. This means that you can chain `then()` calls off of the promise. These then calls are either fulfilled with a successful `PsrHttpMessageResponseInterface` or rejected with an exception.
    
    
    use PsrHttpMessageResponseInterface;
    use GuzzleHttpExceptionRequestException;
    
    $promise = $client->requestAsync('GET', 'http://httpbin.org/get');
    $promise->then(
        function (ResponseInterface $res) {
            echo $res->getStatusCode() . "n";
        },
        function (RequestException $e) {
            echo $e->getMessage() . "n";
            echo $e->getRequest()->getMethod();
        }
    );
    

### Concurrent requests

You can send multiple requests concurrently using promises and asynchronous requests.
    
    
    use GuzzleHttpClient;
    use GuzzleHttpPromise;
    
    $client = new Client(['base_uri' => 'http://httpbin.org/']);
    
    // Initiate each request but do not block
    $promises = [
        'image' => $client->getAsync('/image'),
        'png'   => $client->getAsync('/image/png'),
        'jpeg'  => $client->getAsync('/image/jpeg'),
        'webp'  => $client->getAsync('/image/webp')
    ];
    
    // Wait on all of the requests to complete. Throws a ConnectException
    // if any of the requests fail
    $results = Promiseunwrap($promises);
    
    // Wait for the requests to complete, even if some of them fail
    $results = Promisesettle($promises)->wait();
    
    // You can access each result using the key provided to the unwrap
    // function.
    echo $results['image']['value']->getHeader('Content-Length')[0]
    echo $results['png']['value']->getHeader('Content-Length')[0]
    

You can use the `GuzzleHttpPool` object when you have an indeterminate amount of requests you wish to send.
    
    
    use GuzzleHttpPool;
    use GuzzleHttpClient;
    use GuzzleHttpPsr7Request;
    
    $client = new Client();
    
    $requests = function ($total) {
        $uri = 'http://127.0.0.1:8126/guzzle-server/perf';
        for ($i = 0; $i < $total; $i++) {
            yield new Request('GET', $uri);
        }
    };
    
    $pool = new Pool($client, $requests(100), [
        'concurrency' => 5,
        'fulfilled' => function ($response, $index) {
            // this is delivered each successful response
        },
        'rejected' => function ($reason, $index) {
            // this is delivered each failed request
        },
    ]);
    
    // Initiate the transfers and create a promise
    $promise = $pool->promise();
    
    // Force the pool of requests to complete.
    $promise->wait();
    

Or using a closure that will return a promise once the pool calls the closure.
    
    
    $client = new Client();
    
    $requests = function ($total) use ($client) {
        $uri = 'http://127.0.0.1:8126/guzzle-server/perf';
        for ($i = 0; $i < $total; $i++) {
            yield function() use ($client, $uri) {
                return $client->getAsync($uri);
            };
        }
    };
    
    $pool = new Pool($client, $requests(100));
    

## Using Responses

In the previous examples, we retrieved a `$response` variable or we were delivered a response from a promise. The response object implements a PSR-7 response, `PsrHttpMessageResponseInterface`, and contains lots of helpful information.

You can get the status code and reason phrase of the response:
    
    
    $code = $response->getStatusCode(); // 200
    $reason = $response->getReasonPhrase(); // OK
    

You can retrieve headers from the response:
    
    
    // Check if a header exists.
    if ($response->hasHeader('Content-Length')) {
        echo "It exists";
    }
    
    // Get a header from the response.
    echo $response->getHeader('Content-Length');
    
    // Get all of the response headers.
    foreach ($response->getHeaders() as $name => $values) {
        echo $name . ': ' . implode(', ', $values) . "rn";
    }
    

The body of a response can be retrieved using the `getBody` method. The body can be used as a string, cast to a string, or used as a stream like object.
    
    
    $body = $response->getBody();
    // Implicitly cast the body to a string and echo it
    echo $body;
    // Explicitly cast the body to a string
    $stringBody = (string) $body;
    // Read 10 bytes from the body
    $tenBytes = $body->read(10);
    // Read the remaining contents of the body as a string
    $remainingBytes = $body->getContents();
    

## Query String Parameters

You can provide query string parameters with a request in several ways.

You can set query string parameters in the request's URI:
    
    
    $response = $client->request('GET', 'http://httpbin.org?foo=bar');
    

You can specify the query string parameters using the `query` request option as an array.
    
    
    $client->request('GET', 'http://httpbin.org', [
        'query' => ['foo' => 'bar']
    ]);
    

Providing the option as an array will use PHP's `http_build_query` function to format the query string.

And finally, you can provide the `query` request option as a string.
    
    
    $client->request('GET', 'http://httpbin.org', ['query' => 'foo=bar']);
    

## Uploading Data

Guzzle provides several methods for uploading data.

You can send requests that contain a stream of data by passing a string, resource returned from `fopen`, or an instance of a `PsrHttpMessageStreamInterface` to the `body` request option.
    
    
    // Provide the body as a string.
    $r = $client->request('POST', 'http://httpbin.org/post', [
        'body' => 'raw data'
    ]);
    
    // Provide an fopen resource.
    $body = fopen('/path/to/file', 'r');
    $r = $client->request('POST', 'http://httpbin.org/post', ['body' => $body]);
    
    // Use the stream_for() function to create a PSR-7 stream.
    $body = GuzzleHttpPsr7stream_for('hello!');
    $r = $client->request('POST', 'http://httpbin.org/post', ['body' => $body]);
    

An easy way to upload JSON data and set the appropriate header is using the `json` request option:
    
    
    $r = $client->request('PUT', 'http://httpbin.org/put', [
        'json' => ['foo' => 'bar']
    ]);
    

### POST/Form Requests

In addition to specifying the raw data of a request using the `body` request option, Guzzle provides helpful abstractions over sending POST data.

#### Sending form fields

Sending `application/x-www-form-urlencoded` POST requests requires that you specify the POST fields as an array in the `form_params` request options.
    
    
    $response = $client->request('POST', 'http://httpbin.org/post', [
        'form_params' => [
            'field_name' => 'abc',
            'other_field' => '123',
            'nested_field' => [
                'nested' => 'hello'
            ]
        ]
    ]);
    

#### Sending form files

You can send files along with a form (`multipart/form-data` POST requests), using the `multipart` request option. `multipart` accepts an array of associative arrays, where each associative array contains the following keys:

* name: (required, string) key mapping to the form field name.
* contents: (required, mixed) Provide a string to send the contents of the file as a string, provide an fopen resource to stream the contents from a PHP stream, or provide a `PsrHttpMessageStreamInterface` to stream the contents from a PSR-7 stream.
    
    
    $response = $client->request('POST', 'http://httpbin.org/post', [
        'multipart' => [
            [
                'name'     => 'field_name',
                'contents' => 'abc'
            ],
            [
                'name'     => 'file_name',
                'contents' => fopen('/path/to/file', 'r')
            ],
            [
                'name'     => 'other_file',
                'contents' => 'hello',
                'filename' => 'filename.txt',
                'headers'  => [
                    'X-Foo' => 'this is an extra header to include'
                ]
            ]
        ]
    ]);
    

## Cookies

Guzzle can maintain a cookie session for you if instructed using the `cookies` request option. When sending a request, the `cookies` option must be set to an instance of `GuzzleHttpCookieCookieJarInterface`.
    
    
    // Use a specific cookie jar
    $jar = new GuzzleHttpCookieCookieJar;
    $r = $client->request('GET', 'http://httpbin.org/cookies', [
        'cookies' => $jar
    ]);
    

You can set `cookies` to `true` in a client constructor if you would like to use a shared cookie jar for all requests.
    
    
    // Use a shared client cookie jar
    $client = new GuzzleHttpClient(['cookies' => true]);
    $r = $client->request('GET', 'http://httpbin.org/cookies');
    

## Redirects

Guzzle will automatically follow redirects unless you tell it not to. You can customize the redirect behavior using the `allow_redirects` request option.

* Set to `true` to enable normal redirects with a maximum number of 5 redirects. This is the default setting.
* Set to `false` to disable redirects.
* Pass an associative array containing the 'max' key to specify the maximum number of redirects and optionally provide a 'strict' key value to specify whether or not to use strict RFC compliant redirects (meaning redirect POST requests with POST requests vs. doing what most browsers do which is redirect POST requests with GET requests).
    
    
    $response = $client->request('GET', 'http://github.com');
    echo $response->getStatusCode();
    // 200
    

The following example shows that redirects can be disabled.
    
    
    $response = $client->request('GET', 'http://github.com', [
        'allow_redirects' => false
    ]);
    echo $response->getStatusCode();
    // 301
    

## Exceptions

Guzzle throws exceptions for errors that occur during a transfer.

* In the event of a networking error (connection timeout, DNS errors, etc.), a `GuzzleHttpExceptionRequestException` is thrown. This exception extends from `GuzzleHttpExceptionTransferException`. Catching this exception will catch any exception that can be thrown while transferring requests.
    
        use GuzzleHttpPsr7;
    use GuzzleHttpExceptionRequestException;
    
    try {
        $client->request('GET', 'https://github.com/_abc_123_404');
    } catch (RequestException $e) {
        echo Psr7str($e->getRequest());
        if ($e->hasResponse()) {
            echo Psr7str($e->getResponse());
        }
    }
    

* A `GuzzleHttpExceptionConnectException` exception is thrown in the event of a networking error. This exception extends from `GuzzleHttpExceptionRequestException`.
* A `GuzzleHttpExceptionClientException` is thrown for 400 level errors if the `http_errors` request option is set to true. This exception extends from `GuzzleHttpExceptionBadResponseException` and `GuzzleHttpExceptionBadResponseException` extends from `GuzzleHttpExceptionRequestException`.
    
        use GuzzleHttpExceptionClientException;
    
    try {
        $client->request('GET', 'https://github.com/_abc_123_404');
    } catch (ClientException $e) {
        echo Psr7str($e->getRequest());
        echo Psr7str($e->getResponse());
    }
    

* A `GuzzleHttpExceptionServerException` is thrown for 500 level errors if the `http_errors` request option is set to true. This exception extends from `GuzzleHttpExceptionBadResponseException`.
* A `GuzzleHttpExceptionTooManyRedirectsException` is thrown when too many redirects are followed. This exception extends from `GuzzleHttpExceptionRequestException`.

All of the above exceptions extend from `GuzzleHttpExceptionTransferException`.

## Environment Variables

Guzzle exposes a few environment variables that can be used to customize the behavior of the library.

`GUZZLE_CURL_SELECT_TIMEOUT`
: Controls the duration in seconds that a curl_multi_* handler will use when selecting on curl handles using `curl_multi_select()`. Some systems have issues with PHP's implementation of `curl_multi_select()` where calling this function always results in waiting for the maximum duration of the timeout.

`HTTP_PROXY`
: 

Defines the proxy to use when sending requests using the "http" protocol.

Note: because the HTTP_PROXY variable may contain arbitrary user input on some (CGI) environments, the variable is only used on the CLI SAPI. See  for more information.

`HTTPS_PROXY`
: Defines the proxy to use when sending requests using the "https" protocol.

[1]: http://docs.guzzlephp.org/overview.html#installation
[2]: http://tools.ietf.org/html/rfc3986#section-5.2
[3]: http://docs.guzzlephp.org/handlers-and-middleware.html
[4]: https://promisesaplus.com/
[5]: https://github.com/guzzle/promises
