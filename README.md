# Fetch PHP

[![Latest Version on Packagist](https://img.shields.io/packagist/v/fetchphp/fetch.svg)](https://packagist.org/packages/fetchphp/fetch)
[![Tests](https://github.com/CodeWithSushil/fetch-php/actions/workflows/tests.yml/badge.svg?label=tests&branch=master)](https://github.com/CodeWithSushil/fetch-php/actions/workflows/tests.yml)
[![PHP Version](https://img.shields.io/packagist/php-v/fetchphp/fetch.svg)](https://packagist.org/packages/fetchphp/fetch)
[![License](https://img.shields.io/packagist/l/fetchphp/fetch.svg)](https://packagist.org/packages/fetchphp/fetch)
[![Total Downloads](https://img.shields.io/packagist/dt/fetchphp/fetch.svg)](https://packagist.org/packages/fetchphp/fetch)

**Fetch PHP** is a modern HTTP client library for PHP that brings JavaScript's `fetch` API experience to PHP. Built on top of Guzzle, Fetch PHP allows you to write HTTP code with a clean, intuitive JavaScript-like syntax while still maintaining PHP's familiar patterns.

With support for both synchronous and asynchronous requests, a fluent chainable API, and powerful retry mechanics, Fetch PHP streamlines HTTP operations in your PHP applications.

Full documentation can be found [here](https://fetch-php.thavarshan.com/)

---

## Key Features

- **JavaScript-like Syntax**: Write HTTP requests just like you would in JavaScript with the `fetch()` function and `async`/`await` patterns
- **Promise-based API**: Use familiar `.then()`, `.catch()`, and `.finally()` methods for async operations
- **Fluent Interface**: Build requests with a clean, chainable API
- **Built on Guzzle**: Benefit from Guzzle's robust functionality with a more elegant API
- **Retry Mechanics**: Automatically retry failed requests with exponential backoff
- **PHP-style Helper Functions**: Includes traditional PHP function helpers (`get()`, `post()`, etc.) for those who prefer that style

## Why Choose Fetch PHP?

### Beyond Guzzle

While Guzzle is a powerful HTTP client, Fetch PHP enhances the experience by providing:

- **JavaScript-like API**: Enjoy the familiar `fetch()` API and `async`/`await` patterns from JavaScript
- **Global client management**: Configure once, use everywhere with the global client
- **Simplified requests**: Make common HTTP requests with less code
- **Enhanced error handling**: Reliable retry mechanics and clear error information
- **Type-safe enums**: Use enums for HTTP methods, content types, and status codes

| Feature | Fetch PHP | Guzzle |
|---------|-----------|--------|
| API Style | JavaScript-like fetch + async/await + PHP-style helpers | PHP-style only |
| Client Management | Global client + instance options | Instance-based only |
| Request Syntax | Clean, minimal | More verbose |
| Types | Modern PHP 8.1+ enums | String constants |
| Helper Functions | Multiple styles available | Limited |

## Installation

```bash
composer require fetchphp/fetch
```

> **Requirements**: PHP 8.1 or higher
**Original and official**
```bash
composer require jerome/fetch-php
```

## Basic Usage

### JavaScript-style API

```php
// Simple GET request
$response = fetch('https://api.example.com/users');
$users = $response->json();

// POST request with JSON body
$response = fetch('https://api.example.com/users', [
    'method' => 'POST',
    'json' => ['name' => 'John Doe', 'email' => 'john@example.com'],
]);
```

### PHP-style Helpers

```php
// GET request with query parameters
$response = get('https://api.example.com/users', ['page' => 1, 'limit' => 10]);

// POST request with JSON data
$response = post('https://api.example.com/users', [
    'name' => 'John Doe',
    'email' => 'john@example.com'
]);
```

### Fluent API

```php
// Chain methods to build your request
$response = fetch_client()
    ->baseUri('https://api.example.com')
    ->withHeaders(['Accept' => 'application/json'])
    ->withToken('your-auth-token')
    ->withQueryParameters(['page' => 1, 'limit' => 10])
    ->get('/users');
```

## Async/Await Pattern

### Using Async/Await

```php
// Import async/await functions
use function async;
use function await;

// Wrap your fetch call in an async function
$promise = async(function() {
    return fetch('https://api.example.com/users');
});

// Await the result
$response = await($promise);
$users = $response->json();

echo "Fetched " . count($users) . " users";
```

### Multiple Concurrent Requests with Async/Await

```php
use function async;
use function await;
use function all;

// Execute an async function
await(async(function() {
    // Create multiple requests
    $results = await(all([
        'users' => async(fn() => fetch('https://api.example.com/users')),
        'posts' => async(fn() => fetch('https://api.example.com/posts')),
        'comments' => async(fn() => fetch('https://api.example.com/comments'))
    ]));

    // Process the results
    $users = $results['users']->json();
    $posts = $results['posts']->json();
    $comments = $results['comments']->json();

    echo "Fetched " . count($users) . " users, " .
         count($posts) . " posts, and " .
         count($comments) . " comments";
}));
```

### Sequential Requests with Async/Await

```php
use function async;
use function await;

await(async(function() {
    // First request: get auth token
    $authResponse = await(async(fn() =>
        fetch('https://api.example.com/auth/login', [
            'method' => 'POST',
            'json' => [
                'username' => 'user',
                'password' => 'pass'
            ]
        ])
    ));

    $token = $authResponse->json()['token'];

    // Second request: use token to get user data
    $userResponse = await(async(fn() =>
        fetch('https://api.example.com/me', [
            'token' => $token
        ])
    ));

    return $userResponse->json();
}));
```

### Error Handling with Async/Await

```php
use function async;
use function await;

try {
    $data = await(async(function() {
        $response = await(async(fn() =>
            fetch('https://api.example.com/users/999')
        ));

        if ($response->isNotFound()) {
            throw new \Exception("User not found");
        }

        return $response->json();
    }));

    // Process the data

} catch (\Exception $e) {
    echo "Error: " . $e->getMessage();
}
```

## Traditional Promise-based Pattern

```php
// Set up an async request
// Get the handler for async operations
$handler = fetch_client()->getHandler();
$handler->async();

// Make the async request
$promise = $handler->get('https://api.example.com/users');

// Handle the result with callbacks
$promise->then(
    function ($response) {
        // Process successful response
        $users = $response->json();
        foreach ($users as $user) {
            echo $user['name'] . PHP_EOL;
        }
    },
    function ($exception) {
        // Handle errors
        echo "Error: " . $exception->getMessage();
    }
);
```

## Advanced Async Usage

### Concurrent Requests with Promise Utilities

```php
use function race;

// Create promises for redundant endpoints
$promises = [
    async(fn() => fetch('https://api1.example.com/data')),
    async(fn() => fetch('https://api2.example.com/data')),
    async(fn() => fetch('https://api3.example.com/data'))
];

// Get the result from whichever completes first
$response = await(race($promises));
$data = $response->json();
echo "Got data from the fastest source";
```

### Controlled Concurrency with Map

```php
use function map;

// List of user IDs to fetch
$userIds = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// Process at most 3 requests at a time
$responses = await(map($userIds, function($id) {
    return async(function() use ($id) {
        return fetch("https://api.example.com/users/{$id}");
    });
}, 3));

// Process the responses
foreach ($responses as $index => $response) {
    $user = $response->json();
    echo "Processed user {$user['name']}\n";
}
```

### Batch Processing

```php
use function batch;

// Array of items to process
$items = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// Process in batches of 3 with max 2 concurrent batches
$results = await(batch(
    $items,
    function($batch) {
        // Process a batch
        return async(function() use ($batch) {
            $batchResults = [];
            foreach ($batch as $id) {
                $response = await(async(fn() =>
                    fetch("https://api.example.com/users/{$id}")
                ));
                $batchResults[] = $response->json();
            }
            return $batchResults;
        });
    },
    3, // batch size
    2  // concurrency
));
```

### With Retries

```php
use function retry;

// Retry a flaky request up to 3 times with exponential backoff
$data = await(retry(
    function() {
        return async(function() {
            return fetch('https://api.example.com/unstable-endpoint');
        });
    },
    3, // max attempts
    function($attempt) {
        // Exponential backoff strategy
        return min(pow(2, $attempt) * 100, 1000);
    }
));
```

## Advanced Configuration

### Authentication

```php
// Basic auth
$response = fetch('https://api.example.com/secure', [
    'auth' => ['username', 'password']
]);

// Bearer token
$response = fetch_client()
    ->withToken('your-oauth-token')
    ->get('https://api.example.com/secure');
```

### Proxies

```php
$response = fetch('https://api.example.com', [
    'proxy' => 'http://proxy.example.com:8080'
]);

// Or with fluent API
$response = fetch_client()
    ->withProxy('http://proxy.example.com:8080')
    ->get('https://api.example.com');
```

### Global Client Configuration

```php
// Configure once at application bootstrap
fetch_client([
    'base_uri' => 'https://api.example.com',
    'headers' => [
        'User-Agent' => 'MyApp/1.0',
        'Accept' => 'application/json',
    ],
    'timeout' => 10,
]);

// Use the configured client throughout your application
function getUserData($userId) {
    return fetch_client()->get("/users/{$userId}")->json();
}

function createUser($userData) {
    return fetch_client()->post('/users', $userData)->json();
}
```

## Working with Responses

```php
$response = fetch('https://api.example.com/users/1');

// Check if request was successful
if ($response->isSuccess()) {
    // HTTP status code
    echo $response->getStatusCode(); // 200

    // Response body as JSON
    $user = $response->json();

    // Response body as string
    $body = $response->getBody()->getContents();

    // Get a specific header
    $contentType = $response->getHeaderLine('Content-Type');

    // Check status code categories
    if ($response->getStatus()->isSuccess()) {
        echo "Request succeeded";
    }
}
```

## Working with Type-Safe Enums

```php
use Fetch\Enum\Method;
use Fetch\Enum\ContentType;
use Fetch\Enum\Status;

// Use enums for HTTP methods
$client = fetch_client();
$response = $client->request(Method::POST, '/users', $userData);

// Check HTTP status with enums
if ($response->getStatus() === Status::OK) {
    // Process successful response
}

// Content type handling
$response = $client->withBody($data, ContentType::JSON)->post('/users');
```

## Error Handling

```php
// Synchronous error handling
try {
    $response = fetch('https://api.example.com/nonexistent');

    if (!$response->isSuccess()) {
        echo "Request failed with status: " . $response->getStatusCode();
    }
} catch (\Throwable $e) {
    echo "Exception: " . $e->getMessage();
}

// Asynchronous error handling
$handler = fetch_client()->getHandler();
$handler->async();

$promise = $handler->get('https://api.example.com/nonexistent')
    ->then(function ($response) {
        if ($response->isSuccess()) {
            return $response->json();
        }
        throw new \Exception("Request failed with status: " . $response->getStatusCode());
    })
    ->catch(function (\Throwable $e) {
        echo "Error: " . $e->getMessage();
    });
```

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details.

## Contributing

Contributions are welcome! We're currently looking for help with:

- Expanding test coverage
- Improving documentation
- Adding support for additional HTTP features

To contribute:

1. Fork the Project
2. Create your Feature Branch (`git checkout -b feature/amazing-feature`)
3. Commit your Changes (`git commit -m 'Add some amazing-feature'`)
4. Push to the Branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## Acknowledgments

- Thanks to **Guzzle HTTP** for providing the underlying HTTP client
- Thanks to all contributors who have helped improve this package
- Special thanks to the PHP community for their support and feedback
