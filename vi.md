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

(string|UriInterface) URI gốc của client được nối th các URI tương đối. Có thể là 1 string hoặc là 1 thực thể UriInterface. Khi client đưa ra 1 Uri tương đối, client sẽ nối URI gốc với URI tương đối sử dụng các luật được mô tả trong [RFC 3986, section 2][2].
    
    
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

Bạn có thể tìm hiểu thêm về client middleware ở trang [_Handlers and Middleware_][3] trong tài liệu.
 

### Các Request bất đồng bộ

You can send asynchronous requests using the magic methods provided by a client:
Bạn có thể gửi các request bất đồng bộ sử dụng các magic method
    
    $promise = $client->getAsync('http://httpbin.org/get');
    $promise = $client->deleteAsync('http://httpbin.org/delete');
    $promise = $client->headAsync('http://httpbin.org/get');
    $promise = $client->optionsAsync('http://httpbin.org/get');
    $promise = $client->patchAsync('http://httpbin.org/patch');
    $promise = $client->postAsync('http://httpbin.org/post');
    $promise = $client->putAsync('http://httpbin.org/put');
    

Bạn cũng có thể sử dụng các hàm sendAsync() và requestAsync() của 1 client:
    
    
    use GuzzleHttpPsr7Request;
    
    // Create a PSR-7 request object to send
    $headers = ['X-Foo' => 'Bar'];
    $body = 'Hello!';
    $request = new Request('HEAD', 'http://httpbin.org/head', $headers, $body);
    
    // Or, if you don't need to pass in a request instance:
    $promise = $client->requestAsync('GET', 'http://httpbin.org/get');
    

Các promise trả về của những hàm này implements [Promises/A+ spec][4], được cung cấp bởi [Guzzle promises library][5]. Điều này có nghĩa là bạn có thể gọi hàm`then()` nối sau promise. Những lời gọi sau này cũng thỏa mãn 1 `PsrHttpMessageResponseInterface` thành công hoặc bị từ chối với 1 ngoại lệ.
 
    
    
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
    

### Các requests đồng thời

Bạn có thể gửi nhiều request đồng thời sử dụng các promise và các request bất đồng bộ.
    
    
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
    

Bạn có thể sử dụng đối tượng `GuzzleHttpPool`  khi bạn không xác định được lượng request cần phải gửi.
    
    
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
    

Hoặc sử dụng 1 closure trả về 1 promise khi pool gọi closure.
    
    
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
    

## Sử dụng các Responses

Trong ví dụ bên trên, chúng ta đã lấy được 1 biến `$response` hoặc bạn sẽ lấy được 1 response từ 1 promise. Đối tưởng response sẽ implements 1 PSR-7 response, `PsrHttpMessageResponseInterface`, và chứa rất nhiều thông tin có ích. 

Chúng ta có thể lấy được mã trạng thái và reason phrase của response:
    
    
    $code = $response->getStatusCode(); // 200
    $reason = $response->getReasonPhrase(); // OK
    

Bạn cũng có thể lấy được các header từ response:
    
    
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
    

Nội dung body của 1 response có thể lấy được bằng cách sử dụng hàm `getBody`. Nội dung body có thể được sử dụng như 1 chuỗi, ép thành 1 chuỗi, hayay sử dụng như 1 luồng như đối tượng.
    
    
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

Bạn có thể tạo ra các tham số trong chuỗi query trong 1 request bằng 1 vài cách.

Bạn cóó thể đặt các tham số chuỗi query trong URI của request:

    
    
    $response = $client->request('GET', 'http://httpbin.org?foo=bar');
    

Bạn cũng có thể đặc tả các tham số chuỗi query sử dụng tùy chọn request `query` dưới dạng mảng.

    
    
    $client->request('GET', 'http://httpbin.org', [
        'query' => ['foo' => 'bar']
    ]);
    

Tạo tùy chọn dạng mảng sẽ sử dụng hàm  `http_build_query` của PHP để định dạng chuỗi query. 

Và cuối cùng, bạn đã có thể tạo được tùy chọn request `query` như 1 chuỗi rùi đó.
    
    
    $client->request('GET', 'http://httpbin.org', ['query' => 'foo=bar']);
    

## Tải dữ liệu lên

Guzzle cung cấp 1 vài phương thức cho việc tải dữ liệu lên. 

Bạn có thể gửi các request chứa luồng dữ liệu bằng cách truyền 1 chuỗi, resource trả về từ hàm `fopen`, hay 1 thực thể của `PsrHttpMessageStreamInterface` cho tùy chọn request `body`. 
    
    
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
    

1 cách đơn giản để tải dữ liệu Json lên và đặt header riêng là sử dụng tùy chọn request `json`:
    
    
    $r = $client->request('PUT', 'http://httpbin.org/put', [
        'json' => ['foo' => 'bar']
    ]);
    

### Các request POST/Form

Để đặc tả thêm các dữ liệu thô của 1 request sử dụng các tùy chọn request `body`, Guzzle cũng cấp các trừu tượng hữu dụng trong quá trình gửi dữ liệu POST.

#### Gửi các trường  trong form

Gửi các request POST `application/x-www-form-urlencoded` yêu cầu bạn phải đặc tả trường POST dưới dạng 1 mảng trong các tùy chọn request `form_params` 
    
    
    $response = $client->request('POST', 'http://httpbin.org/post', [
        'form_params' => [
            'field_name' => 'abc',
            'other_field' => '123',
            'nested_field' => [
                'nested' => 'hello'
            ]
        ]
    ]);
    

#### Gửi các file trong form 

Bạn có thể gửi các file cùng với 1 form  (các yêu cầu `multipart/form-data` POST), sử dụng tùy chọn request `miultipart`. `multipart` chấp nhận các mảng liên kết, mỗi mảng liên kết sẽ chứa các khóa sau"

* name: (required, string) khóa nối với trường tên của form
* contents: (required, mixed) chứa 1 chuỗi để gửi nội dụng của file dưới dạng 1 chuỗi, chứa 1 resource fopen để chuyển nội dụng của 1 luồng PHP, hoặc chứa 1  `PsrHttpMessageStreamInterface` để chuyển nội dung từ 1 luồng PSR-7.
    
    
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

Guzzle có thể giữ 1 phiên cookie nếu được yêu cầu sử dụng tùy chọn request `cookies`. Khi gửi 1 request, tùy chọn `cookies` phải được gắn với 1 thực thể
 `GuzzleHttpCookieCookieJarInterface`.
    
    
    // Use a specific cookie jar
    $jar = new GuzzleHttpCookieCookieJar;
    $r = $client->request('GET', 'http://httpbin.org/cookies', [
        'cookies' => $jar
    ]);
    

Bạn có thể đặt `cookies` về `true` trong khởi tạo client nếu bạn muốn sử dụng cookie jar chung cho tất cả các request.
    
    
    // Use a shared client cookie jar
    $client = new GuzzleHttpClient(['cookies' => true]);
    $r = $client->request('GET', 'http://httpbin.org/cookies');
    

## Điều hướng

Guzzle sẽ tự động chuyển điều hướng trừ khi bạn bảo nó đừng làm vậy. Bạn có thể tùy chỉnh hành đồng điều hướng sử dụng tùy chọn request `allow_redirects`.

* Đặt thành `true` để bật điều hướng bình thường với tối đa 5 lần điều hướng. Đây là cài đặt mặc định.
* Đặt thành `false` để tắt chức năng điều hướng.
* Truyền 1 mảng liên kết chứa khóa 'max' để đặc tả số lần điều hướng tối đa và có thể thêm khóa 'strict' để đặc tả có sử dụng các chuẩn điều hướng RFC nghiêm ngặt hay không( có nghĩa là điều hướng các request POST với các request POST và làm những gì hầu hết các trình duyệt làm với việc điều hướng các request POST với các request GET).
    
    
    $response = $client->request('GET', 'http://github.com');
    echo $response->getStatusCode();
    // 200
    

The following example shows that redirects can be disabled.
    
    
    $response = $client->request('GET', 'http://github.com', [
        'allow_redirects' => false
    ]);
    echo $response->getStatusCode();
    // 301
    

## Các ngoại lệ

Guzzle sẽ ném các ngoại lệ cho những lỗi xảy ra trong quá trình truyền thông tin.

* Trong sự kiện lỗi mạng ( hết thời gian kết nối, các lỗi DNS, vân vân), 1
 `GuzzleHttpExceptionRequestException` sẽ được ném ra. Ngoại lệ này extend từ  `GuzzleHttpExceptionTransferException`. Việc bắt ngoại lệ này sẽ bắt bất cứ ngoại lệ nào có thể được ném ra trong quá trình truyền thông tin.

    
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
    

* 1 ngoại lệ `GuzzleHttpExceptionConnectException` được ném ra trong trường hợp lỗi mạng. Ngoại lệ này extends từ  `GuzzleHttpExceptionRequestException`.
* 1 `GuzzleHttpExceptionClientException` được ném ra mức lỗi 400 nếu tùy chọn request `http_errors` được đặt thành true. Ngoại lệ này extends từ  `GuzzleHttpExceptionBadResponseException` và `GuzzleHttpExceptionBadResponseException` extends từ `GuzzleHttpExceptionRequestException`.
    
        use GuzzleHttpExceptionClientException;
    
    try {
        $client->request('GET', 'https://github.com/_abc_123_404');
    } catch (ClientException $e) {
        echo Psr7str($e->getRequest());
        echo Psr7str($e->getResponse());
    }
    

* 1 `GuzzleHttpExceptionServerException` được ném ra cho lỗi mức 500 nếu tùy chọn request `http_errors` được đặt thành true. Ngoại lệ này extends từ `GuzzleHttpExceptionBadResponseException`.
* 1 `GuzzleHttpExceptionTooManyRedirectsException` sẽ được ném ra khi có quá nhiều lần điều hướng xảy ra. Ngoại lệ này extends từ  `GuzzleHttpExceptionRequestException`.

Tất cả những ngoại lệ trên đều extend từ  `GuzzleHttpExceptionTransferException`.

## Các biến môi trường

Guzzle cung cấp 1 vài biến môi trường có thể sử dụng để tuỳ chỉnh hành động của thư viện.

`GUZZLE_CURL_SELECT_TIMEOUT`
: điều chỉnh thời gian tính bằng giây mà 1 xử lý curl_multi_* sẽ sử dụng khi chọn xử lý curl bằng  `curl_multi_select()`. 1 vài hệ thống có vấn đề với các triển khai của PHP của  `curl_multi_select()` khi mà việc gọi hàm này luôn luôn dẫn tới 1 kết quả bị quá thời gian thực thi tối đa.

`HTTP_PROXY`
: 

Định nghĩa proxy được sử dụng khi gửi các request sử dụng giao thức "http".

Ghi chú: vì biến HTTP_PROXY có thể chứa đầu vào tùy ý từ người dùng trên 1 vài môi trường (CGI), nên nó chỉ được sử dụng trên CLI SAPI. Xem thêm thông tin sau đây.

`HTTPS_PROXY`
: Định nghĩa proxy được sử dụng khi gửi các request sử dụng giao thức "https".

[1]: http://docs.guzzlephp.org/overview.html#installation
[2]: http://tools.ietf.org/html/rfc3986#section-5.2
[3]: http://docs.guzzlephp.org/handlers-and-middleware.html
[4]: https://promisesaplus.com/
[5]: https://github.com/guzzle/promises
