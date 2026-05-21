---
hidden: true
---

# todo

31. Human-in-the-loop systems
32. Когда агент НЕ нужен
33. Anti-patterns AI systems







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
