## Quickstart

```php
// Create new Notifier instance.
$notifier = new Airbrake\Notifier(array(
  'projectId' => 12345, // FIX ME
  'projectKey' => 'abcdefg', // FIX ME
));

// Set global notifier instance.
Airbrake\Instance::set($notifier);

// Somewhere in the app...
try {
  throw new Exception('hello from phpbrake');
} catch(Exception $e) {
  Airbrake\Instance::notify($e);
}
```

## API

Notifier API constists of 4 methods:
- `buildNotice` - builds [Airbrake notice](https://airbrake.io/docs/#create-notice-v3).
- `sendNotice` - sends notice to Airbrake.
- `notify` - shortcut for `buildNotice` and `sendNotice`.
- `addFilter` - adds filter that can modify and/or filter notices.

## Adding custom data to the notice

```php
$notifier->addFilter(function ($notice) {
  $notice['context']['environment'] = 'production';
  return $notice;
});
```

## Filtering sensitive data from the notice

```php
$notifier->addFilter(function ($notice) {
  if (isset($notice['params']['password'])) {
    $notice['params']['password'] = 'FILTERED';
  }
  return $notice;
});
```

## Ignoring specific exceptions

```php
$notifier->addFilter(function ($notice) {
  if ($notice['errors'][0]['type'] == 'MyExceptionClass') {
    return false;
  }
  return $notice;
});
```

## Error handler

Notifier can handle PHP errors, uncatched exceptions and shutdown. You can register appropriate handlers using following code:

```php
$handler = new Airbrake\ErrorHandler($notifier);
$handler->register();
```

Under the hood `$handler->register` does following:

```
set_error_handler(array($this, 'onError'), error_reporting());
set_exception_handler(array($this, 'onException'));
register_shutdown_function(array($this, 'onShutdown'));
```