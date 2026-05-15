---
hidden: true
---

# todo

## 1.3.4 Probability вместо “мышления”

Самое важное, что должен понять инженер:

LLM не “думает” как человек.

Модель вычисляет вероятности.

На каждом шаге она отвечает на вопрос:

какой токен наиболее вероятен следующим?

***

### Пример

Если prompt:

```
The capital of France is
```

модель может вычислить:

| Token  | Probability |
| ------ | ----------- |
| Paris  | 0.91        |
| Lyon   | 0.03        |
| London | 0.01        |

После этого sampling algorithm выбирает токен.

***

### Chain-of-thought

Когда модель “рассуждает”, происходит не магия мышления.

Она просто генерирует последовательность токенов, которая статистически похожа на reasoning.

И это очень важное различие.

***

### Emergent behavior

Интересно, что при масштабировании моделей появляются способности, которые явно не программировались:

* reasoning
* planning
* code generation
* tool usage
* multilingual understanding

Это называется emergent behavior.

Даже исследователи до конца не понимают, почему это возникает.

***

## 1.3.5 Temperature, top-p и sampling

Если всегда выбирать самый вероятный токен, ответы станут скучными и повторяющимися.

Поэтому используются sampling strategies.

***

### Temperature

Temperature управляет “случайностью” модели.

Низкая temperature:

```
0.1
```

– более deterministic output.

<br>

Высокая:

```
1.5
```

– больше randomness.

***

### Пример распределения

P\_i=\frac{e^{z\_i/T\}}{\sum\_j e^{z\_j/T\}}

где:

* T – temperature
* z – logits модели

***

### Почему это важно для engineering

Для production systems высокая randomness часто опасна.

Например:

* SQL generation
* financial systems
* agents with tools
* infrastructure automation

Там нужен более deterministic behavior.

***

### Top-p sampling

Top-p ограничивает выбор токенов только наиболее вероятными вариантами.

Например:

```
top-p = 0.9
```

означает:

выбираем только токены, суммарная вероятность которых достигает 90%.

***

### Production guideline

Типичная конфигурация:

| Use Case         | Temperature |
| ---------------- | ----------- |
| Code generation  | 0.1–0.3     |
| SQL              | 0.0–0.2     |
| Creative writing | 0.8–1.2     |
| AI agents        | 0.2–0.5     |

***

## 1.3.6 Embeddings

Embeddings – это числовое представление смысла.

Модель превращает текст в вектор:

```
"cat" → [0.12, -0.91, 0.44, ...]
```

Похожие тексты имеют похожие embeddings.

***

### Почему embeddings изменили backend engineering

Именно embeddings сделали возможными:

* semantic search
* RAG systems
* vector databases
* recommendation engines
* memory systems
* retrieval pipelines

***

### Cosine similarity

Сходство обычно измеряется через cosine similarity:

\cos(\theta)=\frac{A\cdot B}{\\|A\\|\\|B\\|}

Если cosine similarity близок к 1:

* embeddings похожи.

***

### Пример на PHP

```php
<?php

function cosineSimilarity(array $a, array $b): float
{
    $dot = 0;
    $normA = 0;
    $normB = 0;

    for ($i = 0; $i < count($a); $i++) {
        $dot += $a[$i] * $b[$i];
        $normA += $a[$i] ** 2;
        $normB += $b[$i] ** 2;
    }

    return $dot / (sqrt($normA) * sqrt($normB));
}

$v1 = [0.1, 0.3, 0.7];
$v2 = [0.2, 0.4, 0.6];

echo cosineSimilarity($v1, $v2);
```

***

### Визуализация embeddings space

\[IMAGE\_PLACEHOLDER: embeddings\_space]

#### Prompt

```
3D embeddings space visualization.
Words like "cat", "dog", "animal" clustered together.
Technical AI vector database style.
Dark modern educational design.
```

***

## 1.3.7 Hallucinations и uncertainty

Hallucination – это ситуация, когда модель генерирует уверенно звучащий, но ложный ответ.

Ключевая проблема:

модель оптимизирована на правдоподобность, а не на истинность.

Это фундаментальное ограничение autoregressive generation.

***

### Почему hallucinations неизбежны

Модель не имеет встроенного truth engine.

Она лишь продолжает наиболее вероятную последовательность токенов.

Если данных недостаточно:

* модель начинает “достраивать”
* interpolation превращается в fabrication

***

### Особенно опасны hallucinations в:

* медицине
* юриспруденции
* финансах
* infrastructure automation
* AI agents с tool execution

***

### Почему AI-системы обязаны проверять модель

Production AI architecture обычно строится так:

```
LLM
 ↓
Validation
 ↓
Constraints
 ↓
Business Rules
 ↓
Execution
```

Нельзя напрямую доверять output модели.

***

## 1.3.8 Почему LLM ошибаются

Причин много.

***

### 1. Ограниченность training data

Модель знает только то, что было в обучении.

***

### 2. Context fragmentation

Большой контекст ухудшает attention.

***

### 3. Нет настоящего world model

LLM не “понимает” мир как человек.

Она статистически моделирует язык.

***

### 4. Probabilistic generation

Даже правильный ответ может проиграть sampling process.

***

### 5. Отсутствие persistent memory

Модель не “помнит” между запросами, если память явно не реализована системой.

***

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
