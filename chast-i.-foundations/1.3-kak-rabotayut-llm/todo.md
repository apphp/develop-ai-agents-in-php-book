---
hidden: true
---

# todo



## 1.3.9 Ограничения современных моделей

Несмотря на огромный прогресс, современные LLM всё ещё имеют серьёзные ограничения.

***

### Они плохо работают с:

* длинными reasoning chains
* сложной математикой
* точными вычислениями
* persistent planning
* long-term consistency
* guaranteed correctness

***

### Они очень дороги

Inference giant models требует:

* GPU clusters
* massive memory bandwidth
* distributed inference
* aggressive optimization

***

### Они нестабильны

Один и тот же prompt может давать разные ответы.

Для enterprise systems это огромная проблема.

***

## 1.3.10 Почему AI-системы должны быть deterministic поверх probabilistic models

Это одна из главных идей современной AI engineering.

***

### Важнейший принцип

LLM не должна быть системой исполнения.

Она должна быть системой генерации гипотез.

Execution layer обязан быть deterministic.

***

### Правильная архитектура

```
User Request
 ↓
LLM reasoning
 ↓
Structured output
 ↓
Validation
 ↓
Policy checks
 ↓
Deterministic execution
 ↓
Result
```

***

### Пример

Плохой подход:

```
"LLM, выполни SQL."
```

Правильный подход:

```
LLM → SQL draft
↓
SQL parser
↓
validation
↓
permissions
↓
execution
```

***

### Почему это критично для AI-agents

AI-agent без deterministic constraints:

* может удалить данные
* вызвать дорогой API
* зациклиться
* генерировать unsafe actions

Поэтому современные agent architectures используют:

* validators
* typed outputs
* schemas
* guardrails
* policy engines
* workflow constraints

***

### Пример structured output на PHP

```php
<?php

$response = json_decode($llmOutput, true);

if (!isset($response['action'])) {
    throw new Exception('Invalid schema');
}

$allowedActions = ['search', 'summarize'];

if (!in_array($response['action'], $allowedActions)) {
    throw new Exception('Forbidden action');
}

executeAction($response);
```

***

## Итоги

Современные LLM – это не “искусственный мозг” и не магия.

Это огромные probabilistic systems, основанные на:

* token prediction
* transformer attention
* embeddings
* statistical language modeling

Но именно комбинация масштаба, transformers и massive training data привела к появлению систем, которые радикально меняют software engineering.

Для backend-разработчика важно понимать главное:

production AI – это не только модель.

Это архитектура вокруг модели:

* context management
* retrieval
* memory
* validation
* orchestration
* deterministic execution
* observability
* cost control

Именно здесь сегодня рождается новая инженерная дисциплина – AI systems engineering.
