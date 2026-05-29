---
hidden: true
---

# todo

Оптимальная структура:

*   2.2 Structured AI-output



    * 2.2.3 Typed AI responses и DTO
    * 2.2.4 Validation layers
    * 2.2.5 Recovery strategies
    * 2.2.6 Deterministic pipelines
    * 2.2.7 Confidence scoring
    * 2.2.8 Schema evolution
    * 2.2.9 Production patterns для structured AI systems
    * AI output contracts ???????????





***

### 2.2.3 Typed AI responses и DTO

Когда JSON становится стабильным интерфейсом, следующий шаг – типизация.

В PHP это обычно реализуется через DTO.

***

#### Пример DTO

```php
final class SentimentAnalysisResult
{
    public function __construct(
        public readonly string $sentiment,
        public readonly float $urgency,
        public readonly string $summary,
    ) {}
}
```

Теперь AI-output превращается в обычный typed object.

***

#### Mapping JSON -> DTO

```php
$data = json_decode($response, true);

$result = new SentimentAnalysisResult(
    sentiment: $data['sentiment'],
    urgency: $data['urgency'],
    summary: $data['summary'],
);
```

Такой подход резко повышает надежность AI-системы.

Вместо хаотичного текста приложение начинает работать с:

* типами,
* контрактами,
* инвариантами,
* validation rules.

***

#### DTO как boundary layer

DTO особенно важны в AI-системах, потому что LLM нельзя полностью доверять.

DTO создают boundary между:

* probabilistic layer,
* deterministic business logic.

Архитектурно это выглядит так:

```
LLM -> Validation -> DTO -> Domain Logic
```

***

#### Почему associative arrays недостаточно

Многие PHP-разработчики оставляют AI-output в массиве:

```php
$data['urgency']
```

Но production AI быстро становится слишком сложным.

Появляются:

* versioning,
* retries,
* fallback-модели,
* observability,
* confidence scoring,
* routing,
* orchestration.

DTO помогают удерживать систему под контролем.

***

### 2.2.4 Validation layers

Даже если модель поддерживает structured output, validation всё равно обязательна.

LLM может:

* забыть поле,
* вернуть null,
* перепутать тип,
* выдать invalid enum,
* вернуть текст вместо числа,
* сгенерировать broken JSON.

Поэтому production AI строится в несколько validation layers.

***

#### Layer 1 – JSON parsing

```php
$data = json_decode($response, true);

if (json_last_error() !== JSON_ERROR_NONE) {
    throw new RuntimeException('Invalid JSON');
}
```

***

#### Layer 2 – Schema validation

```php
if (!isset($data['sentiment'])) {
    throw new RuntimeException('Missing sentiment');
}
```

Обычно используются:

* opis/json-schema,
* symfony/validator,
* custom validators.

***

#### Layer 3 – Semantic validation

Даже корректный JSON может быть логически неправильным.

Например:

```json
{
  "urgency": 7.5
}
```

JSON валиден.

Но бизнес-логика сломана.

Поэтому появляются semantic validators:

```php
if ($data['urgency'] < 0 || $data['urgency'] > 1) {
    throw new DomainException('Invalid urgency');
}
```

***

#### Layer 4 – Business validation

Иногда output формально корректен, но нарушает бизнес-правила.

Например:

```json
{
  "department": "finance"
}
```

Хотя support-система поддерживает только:

* sales,
* support,
* billing.

***

#### Defense in depth

Production AI почти всегда использует layered validation.

Это аналог defense in depth в security engineering.

***

### 2.2.5 Recovery strategies

Даже хорошие модели периодически генерируют invalid output.

Поэтому production AI обязан уметь восстанавливаться.

***

#### Retry strategy

Самая простая стратегия:

```php
for ($i = 0; $i < 3; $i++) {

    $response = $llm->generate($prompt);

    if ($validator->isValid($response)) {
        return $response;
    }
}
```

***

#### Self-healing prompt

Иногда системе проще отправить модели её собственную ошибку:

```
Ваш JSON невалиден.
Поле urgency должно быть числом от 0 до 1.
Исправьте ответ.
```

Surprisingly, это работает очень хорошо.

***

#### Fallback model

Если primary model не справилась:

```
GPT-5 -> Claude -> Smaller local model
```

Это особенно важно в distributed AI-platforms.

***

#### Partial recovery

Иногда можно восстановить только часть ответа.

Например:

```json
{
  "sentiment": "negative",
  "summary": "..."
}
```

Если urgency отсутствует, система может:

* вычислить fallback,
* запросить повторно только missing field,
* использовать default value.

***

#### Human escalation

Некоторые ошибки нельзя исправить автоматически.

Тогда pipeline делает escalation:

```
AI confidence too low -> Human review
```

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
