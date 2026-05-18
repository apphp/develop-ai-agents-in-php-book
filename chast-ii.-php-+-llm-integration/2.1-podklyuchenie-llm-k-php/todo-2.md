---
hidden: true
---

# todo 2

## 2.1.5 OpenRouter и unified gateways

OpenRouter даёт единый API для множества моделей.

### Архитектура

```
PHP → OpenRouter → multiple LLMs
```

### Плюсы

* единый API
* десятки моделей
* простой failover
* гибкость выбора

***

### Laravel пример

```php
$response = Http::withToken(env('OPENROUTER_API_KEY'))
    ->post('https://openrouter.ai/api/v1/chat/completions', [
        'model' => 'openai/gpt-4.1-mini',
        'messages' => [
            ['role' => 'user', 'content' => 'Explain transformers']
        ]
    ]);
```

***

## 2.1.6 Unified AI abstraction layer

Без abstraction layer код быстро деградирует:

```php
if ($provider === 'openai') {}
if ($provider === 'anthropic') {}
if ($provider === 'ollama') {}
```

***

### Интерфейс

```php
interface AIProvider {
    public function generate(string $prompt): AIResponse;
}
```

***

### Laravel service

```php
class AIService
{
    public function __construct(
        private AIProvider $provider
    ) {}

    public function ask(string $prompt): string
    {
        return $this->provider->generate($prompt)->text;
    }
}
```

***

## 2.1.7 Structured responses

LLM по умолчанию возвращает текст, а не структуру.

### Проблема

```
"Sure! Here is JSON:"
{ broken json
```

***

### Решение: DTO

```php
class AnalysisResult {
    public function __construct(
        public string $summary,
        public float $score
    ) {}
}
```

***

### Laravel validation layer

```php
$data = $response->json();

abort_unless(isset($data['summary']), 500);

$result = new AnalysisResult(
    $data['summary'],
    $data['score']
);
```

***

## 2.1.8 Streaming responses

LLM может возвращать данные постепенно.

### Математика latency

T = T\_{prompt} + N \cdot T\_{token}

T = T\_{prompt} + N \cdot T\_{token}

***

### Laravel streaming (упрощённо)

```php
$response = Http::withToken(env('OPENAI_API_KEY'))
    ->withOptions([
        'stream' => true,
    ])
    ->post('https://api.openai.com/v1/chat/completions', [
        'model' => 'gpt-4.1-mini',
        'stream' => true,
        'messages' => [
            ['role' => 'user', 'content' => 'Explain transformers']
        ]
    ]);
```

***

## 2.1.9 Retries и exponential backoff

### Формула

delay = base \times 2^n

***

### PHP реализация

```php
function retry(callable $fn, int $max = 5)
{
    for ($i = 0; $i < $max; $i++) {
        try {
            return $fn();
        } catch (Throwable $e) {
            sleep(pow(2, $i));
        }
    }

    throw new Exception('Max retries exceeded');
}
```

***

### Laravel retry helper

```php
use Illuminate\Support\Facades\Http;

$response = retry(5, function () {
    return Http::get('https://api.openai.com/v1/models');
});
```

***

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
