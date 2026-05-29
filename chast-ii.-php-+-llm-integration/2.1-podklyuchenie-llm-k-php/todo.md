---
hidden: true
---

# todo

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
