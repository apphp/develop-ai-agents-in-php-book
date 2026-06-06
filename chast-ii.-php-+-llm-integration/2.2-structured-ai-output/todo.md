---
hidden: true
---

# todo

Оптимальная структура:

*   2.2 Structured AI-output





    * 2.2.6 Deterministic pipelines
    * 2.2.7 Confidence scoring
    * 2.2.8 Schema evolution
    * 2.2.9 Production patterns для structured AI systems
    * AI output contracts ???????????





***

### 2.2.6 Deterministic pipelines

Многие разработчики считают AI inherently nondeterministic.

Это не совсем так.

Сама модель вероятностная, но pipeline вокруг неё может быть почти детерминированным.

***

#### Основная идея

Production AI минимизирует randomness:

* temperature → 0,
* fixed schema,
* validation,
* retries,
* constrained decoding,
* typed contracts.

***

#### Deterministic AI pipeline

```
Input
  ↓
Prompt Builder
  ↓
Schema Enforcement
  ↓
LLM
  ↓
Validation
  ↓
DTO Mapping
  ↓
Business Logic
```

***

#### Почему это важно

Без deterministic pipelines AI начинает вести себя хаотично.

Один и тот же email:

* иногда попадает в billing,
* иногда в support,
* иногда получает different urgency score.

Production-системы так работать не могут.

***

#### Temperature и randomness

Температура влияет на случайность генерации.

Упрощенно вероятность выбора токена выглядит так:

P(x\_i)=\frac{e^{z\_i/T\}}{\sum\_j e^{z\_j/T\}}

Где:

* zi – логит токена,
* T – temperature.

При:

```
T → 0
```

модель становится более предсказуемой.

***

### 2.2.7 Confidence scoring

Production AI редко работает по бинарной схеме:

```
valid / invalid
```

Обычно система оценивает confidence.

***

#### Пример

```json
{
  "sentiment": "negative",
  "confidence": 0.91
}
```

***

#### Confidence как routing mechanism

Например:

```
confidence > 0.9 -> auto-action
confidence > 0.7 -> semi-automatic
confidence < 0.7 -> human review
```

***

#### Почему confidence сложен

LLM часто “слишком уверены”.

Модель может уверенно выдать hallucination.

Поэтому confidence scoring обычно комбинируется:

* model confidence,
* heuristics,
* semantic checks,
* retrieval confidence,
* historical metrics.

***

#### Confidence aggregation

Иногда confidence агрегируется из нескольких сигналов:

C\_{final}=\alpha C\_{model}+\beta C\_{retrieval}+\gamma C\_{validation}

Где:

* Cmodel – уверенность модели,
* Cretrieval – качество retrieval,
* Cvalidation – результат validation.

***

### 2.2.8 Schema evolution

Рано или поздно schema меняется.

Например, сначала система возвращала:

```json
{
  "sentiment": "negative"
}
```

Позже появляется:

```json
{
  "sentiment": "negative",
  "language": "en",
  "toxicity": 0.14
}
```

Это приводит к тем же проблемам, что и versioning API.

***

#### Основные правила schema evolution

**Никогда не удалять поля резко**

Плохо:

```json
{
  "mood": "negative"
}
```

если раньше поле называлось:

```json
{
  "sentiment": "negative"
}
```

***

**Добавлять backward compatibility**

Лучше:

```json
{
  "sentiment": "negative",
  "mood": "negative"
}
```

***

**Versioning**

```json
{
  "schema_version": 2
}
```

***

#### AI contracts должны быть стабильными

В больших AI-platforms schema evolution становится отдельной инженерной задачей.

Появляются:

* schema registries,
* compatibility tests,
* contract testing,
* rollout strategies.

***

### 2.2.9 Production patterns для structured AI systems

К этому моменту становится понятно:

production AI – это не “один prompt”.

Это полноценная распределенная система.

***

#### Типичная архитектура

```
Client
  ↓
AI Gateway
  ↓
Prompt Builder
  ↓
LLM Provider
  ↓
Schema Validator
  ↓
DTO Mapper
  ↓
Business Rules
  ↓
Observability Layer
```

***

#### Structured output как фундамент AI engineering

Без structured output невозможно построить:

* AI agents,
* orchestration systems,
* AI workflows,
* multi-agent systems,
* autonomous pipelines,
* reliable automation.

Потому что все эти системы требуют предсказуемого интерфейса между компонентами.

***

#### Связь со сквозным проектом книги

В нашем SaaS AI-проекте поддержки клиентов structured output будет использоваться практически везде:

* sentiment analysis,
* urgency detection,
* ticket routing,
* suggested replies,
* CRM classification,
* escalation pipelines,
* observability,
* analytics.

Каждый AI-модуль будет возвращать не plain text, а строго типизированный контракт.

Именно это превращает LLM из “чат-бота” в инженерный компонент production-платформы.

***

## Итоги

Structured AI-output – один из важнейших переходов в современной AI-разработке.

Пока AI работает как генератор текста, система остается нестабильной.

Но когда появляются:

* schema,
* DTO,
* validation,
* contracts,
* deterministic pipelines,
* recovery strategies,

LLM начинает превращаться в предсказуемый инфраструктурный компонент.

Именно на этом фундаменте строятся production AI-platforms, AI agents и сложные distributed AI systems.

***

### Идеи картинок для главы

#### Картинка 2 – “Validation layers”

Промпт:

```
Layered AI validation architecture: JSON parsing, schema validation, semantic validation, business validation. Clean engineering diagram with layered defense concept.
```

***

#### Картинка 3 – “Deterministic AI pipeline”

Промпт:

```
Production AI pipeline architecture diagram: Input -> Prompt Builder -> LLM -> Schema Validation -> DTO -> Business Logic -> Observability. Modern distributed systems illustration.
```
