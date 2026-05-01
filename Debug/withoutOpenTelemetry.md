# PHP Logging with Graylog (Simple Explanation)

---

## 🧠 1. Big Picture

👉 How logs flow without OpenTelemetry:

PHP App → Log file → Log Agent → Graylog → UI (search & alerts)

---

## 🐘 2. PHP Logging (Basic)

### 🔹 Simple logging

```php
<?php
error_log("User logged in");
error_log("Payment failed");


This writes logs to a file (like /var/log/...)
```

### 🔹 Better: JSON logging

```php
<?php
$log = [
    "level" => "info",
    "message" => "User logged in",
    "user_id" => 123
];

error_log(json_encode($log));
```

### 🔹 Using a library (Recommended)

```php
<?php
use Monolog\Logger;
use Monolog\Handler\StreamHandler;

$log = new Logger('app');
$log->pushHandler(new StreamHandler('app.log', Logger::INFO));

$log->info('User logged in', ["user_id" => 123]);
```

## 3. What is a Log Agent?

### A log agent is a small program that:

- reads log files
- processes them
- sends them to Graylog

### Common Log Agents

- Filebeat
- Fluentd
- Logstash

### What it does

app.log → Agent → Graylog

## 4. Sending logs to Graylog

🔹 Option 1: Using Log Agent (Best way)

Flow:

PHP → app.log → Filebeat → Graylog

🔹Option 2: Direct (No agent)

👉 Send logs directly using Monolog + GELF

```php
<?php
use Monolog\Logger;
use Monolog\Handler\GelfHandler;
use Gelf\Publisher;
use Gelf\Transport\UdpTransport;

$transport = new UdpTransport("127.0.0.1", 12201);
$publisher = new Publisher($transport);
$handler = new GelfHandler($publisher);

$logger = new Logger('app');
$logger->pushHandler($handler);

$logger->info("User logged in");
```

### Agent vs Direct

| Feature     | Log Agent | Direct       |
| ----------- | --------- | ------------ |
| Reliability | High      | Medium       |
| Data loss   | No        | Possible     |
| Performance | Good      | Can slow app |
| Setup       | More      | Easy         |
