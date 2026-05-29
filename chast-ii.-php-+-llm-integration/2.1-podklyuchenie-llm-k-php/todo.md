---
hidden: true
---

# todo





## 2.1.10 Rate limiting

LLM API ограничивают:

* RPM (requests per minute)
* TPM (tokens per minute)

Стоимость примерно:

Cost \propto input + output tokens

Cost \propto InputTokens + OutputTokens

Чем больше context window – тем выше стоимость.

***

## 2.1.11 Timeout handling

Без таймаутов система может зависнуть.

```php
new Client([
    'timeout' => 30,
    'connect_timeout' => 5
]);
```

Типы таймаутов:

* connect;
* request;
* streaming.

***

## 2.1.12 Failover между моделями

В production почти всегда несколько моделей:

```
GPT → Claude → Ollama
```

PHP:

```php
foreach ($providers as $provider) {
    try {
        return $provider->generate($prompt);
    } catch (Throwable $e) {}
}
```

Это даёт resilience системы.

***

## 2.1.13 Universal AI client architecture

Со временем система превращается в AI gateway:

```
App → AI Gateway → Routing → Providers
```

\[IMAGE\_PLACEHOLDER: AI gateway architecture]

Функции:

* routing;
* caching;
* observability;
* failover;
* policy enforcement.

***

## 2.1.14 Observability для AI-систем

AI нельзя мониторить как обычный API.

Нужно отслеживать:

* latency;
* tokens;
* cost;
* hallucinations;
* retries;
* provider failures.

Проблема: качество ответа субъективно.

***

## 2.1.15 Кэширование и оптимизация стоимости

Один из самых сильных способов оптимизации – caching.

Варианты:

* prompt hashing;
* semantic caching;
* embedding-based lookup.

Цель:

reduce \\; cost = reduce(tokens + calls)

***

## Итог главы

Мы прошли путь от простого HTTP запроса к полноценной архитектуре AI-системы.

Главный вывод:

LLM-интеграция – это не API вызов, а distributed probabilistic system с инфраструктурными требованиями уровня backend platform.

***

Если хотите, дальше я могу:

* превратить это в стиль “GitBook-ready”;
* или добавить ещё больше PHP (Laravel-версию);
* или перейти к главе 2.2 (архитектура AI gateway) и связать всё в единую систему.
