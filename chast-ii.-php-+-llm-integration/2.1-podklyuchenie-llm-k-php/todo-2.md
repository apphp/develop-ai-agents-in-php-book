---
hidden: true
---

# todo 2

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
