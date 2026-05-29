---
hidden: true
---

# todo 2



## 2.1.10 Rate limiting

### Стоимость вычислений

Cost \propto InputTokens + OutputTokens

***

### Laravel middleware

```php
RateLimiter::for('ai', function () {
    return Limit::perMinute(60);
});
```

***

## 2.1.11 Timeout handling

### PHP

```php
$client = new Client([
    'timeout' => 30,
    'connect_timeout' => 5,
]);
```

***

### Laravel

```php
Http::timeout(30)
    ->connectTimeout(5)
    ->post(...);
```

***

## 2.1.12 Failover между моделями

```php
foreach ($providers as $provider) {
    try {
        return $provider->generate($prompt);
    } catch (Throwable $e) {}
}
```

***

### Laravel version

```php
app(AIService::class)->ask($prompt);
```

А внутри сервиса может быть routing между провайдерами.

***

## 2.1.13 Universal AI client architecture

```
App → AI Gateway → Routing → Providers
```

\[IMAGE\_PLACEHOLDER: AI gateway architecture]

***

### Laravel архитектура

* AIService (facade)
* ProviderManager
* OpenAIProvider
* AnthropicProvider
* OllamaProvider
* Router
* Cache layer

***

## 2.1.14 Observability для AI-систем

Важно отслеживать:

* latency
* tokens
* cost
* failures
* retries
* hallucinations

***

### Laravel logging

```php
Log::info('AI request', [
    'tokens' => $tokens,
    'latency' => $latency,
]);
```

***

## 2.1.15 Кэширование и оптимизация стоимости

### Подходы

* prompt hashing
* semantic caching
* embedding reuse

***

### Laravel cache example

```php
$key = md5($prompt);

return Cache::remember($key, 3600, function () use ($prompt) {
    return $this->provider->generate($prompt);
});
```

***

## Итог

Эта глава показывает эволюцию:

от простого HTTP запроса → к полноценной AI-инфраструктуре уровня backend platform

***

Если хочешь дальше, следующий логичный шаг:

* ￼ 2.2 Архитектура AI-клиента (очень глубокая глава с диаграммами, retry policies, circuit breakers, queues и т.д.)
