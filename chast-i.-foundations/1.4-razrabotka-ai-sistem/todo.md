---
hidden: true
---

# todo





31. Planning и execution
32. Reflection loops
33. Statefulness в AI systems
34. Durable execution
35. Human-in-the-loop systems
36. Когда агент НЕ нужен
37. Anti-patterns AI systems





***

## 1.4.4 Planning и execution

Planning – одна из самых переоценённых и одновременно самых важных тем в AI systems.

Когда говорят “агент умеет планировать”, обычно имеют в виду:

– decomposition задач\
– step generation\
– dependency analysis\
– dynamic execution

***

### Простейший planner

Пусть пользователь говорит:

“Проанализируй конкурентов моего SaaS.”

Planner может разбить задачу:

1. Найти конкурентов
2. Собрать pricing
3. Извлечь features
4. Сравнить
5. Сгенерировать summary

***

### Tree-of-thoughts

Некоторые системы строят несколько вариантов reasoning paths.

Идея напоминает поиск в дереве.

\[PLACEHOLDER: tree of thoughts]

#### Prompt для картинки

```
Tree of Thoughts AI reasoning diagram:
Root problem branching into multiple reasoning paths with evaluation scores and best path selected.
Minimalistic technical visualization.
```

***

### Execution engine

Planner сам по себе бесполезен.

Нужен execution engine.

Он отвечает за:

– retries\
– state\
– timeout\
– scheduling\
– cancellation\
– dependency graph

***

### Почему planning сложно

Каждый новый шаг увеличивает вероятность ошибки.

Если вероятность успешного шага:

p=0.95

то вероятность успеха 20 последовательных шагов:

P=0.95^{20}

что уже примерно равно 0.36.

Это фундаментальная проблема autonomous systems.

Чем длиннее reasoning chain – тем выше вероятность failure accumulation.

***

## 1.4.5 Reflection loops

Reflection loop – это механизм, при котором модель анализирует собственный результат.

Простейший пример:

1. Generate answer
2. Critique answer
3. Improve answer
4. Re-evaluate

***

### Self-critique

Модель может выступать:

– исполнителем\
– ревьюером\
– validator’ом

***

### Почему reflection работает

LLM плохо генерируют идеальный результат с первого раза.

Но они surprisingly хорошо умеют:

– находить ошибки\
– улучшать структуру\
– исправлять reasoning

***

### Пример reflection prompt

```
Review your previous answer.
Find logical errors.
Suggest improvements.
Return corrected version only.
```

***

### Опасность reflection loops

Без ограничений агент может уйти в бесконечный цикл.

Например:

– “answer could be improved”\
– “still not optimal”\
– “retrying”

Поэтому production systems почти всегда ограничивают:

– max iterations\
– token budget\
– wall-clock timeout

***

## 1.4.6 Statefulness в AI systems

LLM stateless.

Модель не “помнит” прошлое между запросами.

Вся память существует вне модели.

Это один из важнейших принципов AI engineering.

***

### Где хранится state

Обычно state хранится:

– в Redis\
– PostgreSQL\
– vector DB\
– object storage\
– event logs

***

### Типы state

#### Ephemeral state

Краткоживущий state:

– текущий conversation context\
– temporary execution data

***

#### Durable state

Долговременный state:

– user memory\
– task history\
– workflows\
– checkpoints

***

### Event sourcing и AI

Многие AI systems постепенно приходят к event-driven architecture.

Например:

```
TASK_CREATED
TOOL_CALLED
TOOL_FAILED
RETRY_STARTED
RESULT_GENERATED
```

Это помогает:

– восстанавливать execution\
– дебажить reasoning\
– строить observability

***

## 1.4.7 Durable execution

Это одна из самых недооценённых тем в AI engineering.

LLM operations могут занимать:

– минуты\
– часы\
– иногда дни

Особенно если система:

– запускает research\
– работает с людьми\
– ждёт approvals\
– вызывает внешние APIs

***

### Почему обычный request/response не подходит

Классический HTTP lifecycle слишком короткий.

AI systems требуют:

– pause/resume\
– retries\
– checkpoints\
– recovery after crash

***

### Durable execution engine

Популярные подходы:

– Temporal\
– LangGraph\
– event sourcing\
– workflow engines

***

### Пример durable flow

1. Agent started
2. Tool failed
3. State persisted
4. Process restarted
5. Workflow resumed from checkpoint

***

### PHP и durable execution

PHP традиционно request-oriented.

Но современные AI systems на PHP обычно используют:

– queues\
– Kafka\
– Redis streams\
– RoadRunner\
– Swoole\
– workflow engines

***

### Пример очереди задач

```php
<?php

class AIJob
{
    public function handle(): void
    {
        $result = $this->callLLM();

        if (!$result) {
            $this->retry();
            return;
        }

        $this->save($result);
    }

    private function retry(): void
    {
        echo "Retry scheduled";
    }
}
```

***

## 1.4.8 Human-in-the-loop systems

Полностью autonomous AI systems пока крайне ненадёжны.

Поэтому многие production architectures включают человека.

***

### Human approval

Например:

AI предлагает действие:

– отправить email\
– изменить contract\
– выполнить payment

Но финальное подтверждение делает человек.

***

### HITL architecture

\[PLACEHOLDER: HITL diagram]

#### Prompt для картинки

```
Human-in-the-loop AI system:
User request -> AI analysis -> Human approval -> Execution -> Final result.
Modern enterprise workflow diagram.
```

***

### Почему HITL важен

Он резко снижает:

– hallucinations\
– legal risks\
– security risks\
– catastrophic failures

***

## 1.4.9 Когда агент НЕ нужен

Это чрезвычайно важный раздел.

Потому что сейчас индустрия пытается превратить любую задачу в agentic system.

Чаще всего это ошибка.

***

### Агент НЕ нужен, если:

#### Workflow заранее известен

Например:

– OCR → classification → save

Тут не нужен autonomous planner.

***

#### Цена ошибки высока

Например:

– банковские операции\
– медицина\
– legal systems

***

#### Нужна deterministic execution

Если система должна быть полностью воспроизводимой, agentic behavior опасен.

***

#### Задача слишком простая

Очень часто обычный CRUD + LLM prompt работает лучше, чем сложный агент.

***

## 1.4.10 Anti-patterns AI systems

AI engineering быстро развивается, поэтому anti-patterns появляются постоянно.

***

### “LLM решит всё”

Одна из самых частых ошибок.

LLM не заменяет:

– архитектуру\
– validation\
– business logic\
– observability\
– security

***

### Infinite agent loops

Классическая проблема:

– recursive retries\
– endless reflection\
– uncontrolled planning

***

### No state persistence

Если execution state нигде не хранится, система становится хрупкой.

Crash = потеря workflow.

***

### Over-agentization

Очень многие задачи не требуют агента.

Иногда лучший AI system выглядит так:

```
User Input
↓
Prompt Template
↓
LLM
↓
Structured Output
↓
Save Result
```

И всё.

***

### Prompt spaghetti

Когда orchestration существует только внутри prompt’ов, система быстро становится неуправляемой.

Например:

```
If tool A failed do B unless C happened before D...
```

Это уже не engineering.

Это хаос.

***

## Заключение

Современная AI-разработка гораздо ближе к distributed systems engineering, чем кажется на первый взгляд.

Большая часть сложности AI systems находится не внутри модели, а вокруг неё:

– orchestration\
– workflows\
– state management\
– retries\
– observability\
– durable execution\
– human approvals\
– tool integrations

Именно поэтому AI engineering сегодня всё сильнее сближается с классической бэкенд-разработкой.

LLM – это только новый вычислительный primitive.

Настоящие AI-системы строятся инфраструктурой.
