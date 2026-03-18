> [!info] TL;DR
> Rails Logger — встроенная система логирования Rails для отладки и мониторинга приложения. Она пишет сообщения в лог-файлы по окружениям, поддерживает уровни важности (`debug`/`info`/`warn`/`error`/`fatal`/`unknown`), позволяет настраивать формат логов, использовать теги для контекста и включать ротацию файлов. Чем подробнее логирование, тем выше его стоимость по производительности, поэтому важно выбирать подходящий log level и не делать избыточные вызовы логгера.

## Refs

- [Debugging Rails Applications](https://guides.rubyonrails.org/debugging_rails_applications.html)
- [Mastering Rails Logger - Tips for Effective Debugging](https://signoz.io/guides/rails-logger/#4-formatting-issues-in-logs)
- [Supercharge your Rails logs with tags](https://thoughtbot.com/blog/supercharge-your-rails-logs-with-tags)
- [Active Support Tagged Logging](https://api.rubyonrails.org/classes/ActiveSupport/TaggedLogging.html)

## API

- `debug` — используется как уровень/метод для детализированных сообщений. Источник: [Debugging Rails Applications](https://guides.rubyonrails.org/debugging_rails_applications.html)
- `to_yaml`
- `simple_format`
- `verbose_query_logs`
- `ActiveSupport::Logger`
- `ActiveRecord::QueryLogs`
- `ActiveSupport::TaggedLogging` — тегирование сообщений логов. Источник: [Active Support Tagged Logging](https://api.rubyonrails.org/classes/ActiveSupport/TaggedLogging.html)

## Topics

### Logger

#### Definition

Rails Logger — это встроенная в Ruby on Rails система логирования, предназначенная для отслеживания и понимания поведения приложения. Она фиксирует системные события, ошибки, предупреждения и пользовательские сообщения.

#### How it Works

Rails Logger записывает сообщения в лог-файлы приложения, которые обычно лежат в директории `log/` и разделяются по окружениям, например: `development.log`, `test.log`, `production.log`.

#### Usage

Основные сценарии использования:

- отладка;
- мониторинг;
- запись значимых событий и данных в ходе работы приложения.

#### Examples

```ruby
class OrdersController < ApplicationController
  def create
    @order = Order.new(order_params)

    Rails.logger.info("Creating order for user: #{current_user.id}")

    if @order.save
      Rails.logger.info("Order #{@order.id} created successfully")
    else
      Rails.logger.warn("Order creation failed: #{@order.errors.full_messages}")
    end
  end
end
```

#### When to Use

Когда нужно фиксировать ключевые этапы бизнес-процесса, отслеживать ошибки и наблюдать за поведением приложения.

#### Summary

Rails Logger дает базовый механизм наблюдаемости в Rails и помогает связывать поведение приложения с конкретными событиями во время выполнения.

### Rails.logger.level

#### Definition

`Rails.logger.level` определяет уровень важности сообщений, которые будут записываться в лог.

#### How it Works

В исходных данных перечислены уровни важности:

- `debug: 0` — детальная информация для диагностики;
- `info: 1` — общая информация о работе системы;
- `warn: 2` — потенциально вредные ситуации;
- `error: 3` — ошибки, после которых приложение еще может продолжить работу;
- `fatal: 4` — критические ошибки, которые, вероятно, приводят к остановке;
- `unknown: 5` — сообщения, не попадающие в остальные категории.

#### Usage

Уровни помогают управлять плотностью логов и отделять диагностические сообщения от действительно критичных событий.

#### Examples

```ruby
Rails.logger.info("Order created")
Rails.logger.warn("Order creation failed")
```

#### When to Use

- `debug` — при диагностике проблем;
- `info` — для штатных рабочих событий;
- `warn` — для подозрительных состояний;
- `error` и `fatal` — для ошибок и сбоев.

#### Summary

Разница между log level заключается в серьезности события и объеме логирования, который вы хотите видеть в конкретной среде.

### verbose logs

#### Definition

> [!warning]
> Требуется проверка информации

#### How it Works

Из входных данных следует только то, что `:debug` создает больший performance penalty, чем `:fatal`, потому что вычисляется и записывается значительно больше строк в лог.

#### Usage

Подробное логирование имеет смысл использовать там, где важнее диагностика, чем минимизация накладных расходов.

#### Examples

Прямой пример для `verbose logs` во входных данных не приведен. Из доступной информации есть только сравнение: `:debug` более затратен, чем `:fatal`.

#### When to Use

Когда нужно получить больше диагностической информации о поведении приложения.

#### Summary

Чем подробнее логирование, тем выше нагрузка на вычисление строк и запись в лог.

### Tagged Logging

#### Definition

Tagged Logging позволяет добавлять префиксы-теги к сообщениям логов, чтобы упростить фильтрацию и выделение нужного контекста.

#### How it Works

`Rails.logger.tagged("tag")` оборачивает блок и добавляет тег ко всем сообщениям внутри него. Если передать значение, которое `.blank?`, тег не будет добавлен. Вызовы можно вкладывать, и тогда теги будут накапливаться.

#### Usage

- оборачивать entrypoint сложной операции;
- размечать логи конкретной job;
- добавлять теги только при определенных условиях.

#### Examples

```ruby
Rails.logger.tagged("tag") do
  Rails.logger.info("message")
end
```

```ruby
class MyJob < ApplicationJob
  def perform
    Rails.logger.tagged("MyJob") do
      # ...
    end
  end
end
```

```ruby
Rails.logger.tagged("first") do
  Rails.logger.tagged("second") do
    Rails.logger.info("wow!")
  end
end
```

#### When to Use

Когда в приложении много логов и нужен дополнительный контекст для конкретного запроса, job или подозрительного сценария.

#### Summary

Tagged Logging делает логи более адресными и упрощает их фильтрацию при отладке.

#### Best Practices

- добавлять теги на границах важных операций;
- вкладывать теги для нескольких уровней контекста;
- использовать условные теги, если контекст нужен не всегда.

### Performance

#### Definition

Performance в контексте Rails Logger связан с накладными расходами на формирование и запись логов.

#### How it Works

Использование уровня `:debug` создает больший performance penalty, чем `:fatal`, потому что приходится вычислять и записывать больше строк. Дополнительный риск — слишком большое количество вызовов Logger в коде.

#### Usage

- выбирать подходящий log level;
- использовать conditional logging;
- следить за плотностью логов в горячих участках кода.

#### Examples

- `:debug` более затратен, чем `:fatal`;
- одна из рекомендованных стратегий: логировать только при выполнении определенных условий.

#### When to Use

При настройке логирования в production и в участках приложения, где избыточные логи могут влиять на производительность.

#### Summary

Эффективное логирование требует баланса между объемом диагностической информации и стоимостью записи логов.

#### Best Practices

- избегать избыточных вызовов логгера;
- использовать log level по назначению;
- помнить о влиянии многословного логирования на производительность.

### Customizing Log Format

#### Definition

Формат логов в Rails можно кастомизировать, чтобы включать дополнительные данные или приводить вывод к формату, ожидаемому инструментами мониторинга.

#### How it Works

Для этого можно определить собственный formatter и назначить его через `Rails.logger.formatter`. Метод `call(severity, time, progname, msg)` вызывается внутри Logger при обработке сообщения.

#### Usage

Настройка обычно выполняется в initializer, например в `config/initializers/logger.rb`.

#### Examples

```ruby
class CustomLoggerFormatter < Logger::Formatter
  def call(severity, time, progname, msg)
    "#{time.to_s(:db)} #{severity} #{msg}\n"
  end
end

Rails.logger = Logger.new(STDOUT)
Rails.logger.formatter = CustomLoggerFormatter.new
```

#### When to Use

Когда стандартного формата логов недостаточно или нужен формат, совместимый с системой мониторинга.

#### Summary

Кастомный formatter позволяет централизованно управлять видом сообщений Rails Logger.

### Log file rotation

#### Definition

Ротация логов помогает контролировать размер файлов, периодически переименовывая и архивируя старые логи.

#### How it Works

Rails поддерживает ротацию из коробки. Во входных данных приведен пример настройки через `Logger.new('log/production.log', 10, 100.megabytes)`.

#### Usage

Настраивается в initializer для управления количеством файлов и их максимальным размером.

#### Examples

```ruby
config.logger = Logger.new('log/production.log', 10, 100.megabytes)
```

#### When to Use

Прежде всего в production, чтобы лог-файлы не занимали слишком много дискового пространства.

#### Summary

Ротация логов помогает удерживать размер логов под контролем без ручной очистки.

## Research Questions

### В чем разница между лог левелами

Краткий ответ: log level определяет серьезность сообщения и ожидаемую плотность логирования. `debug` используется для максимально подробной диагностики, `info` — для штатной информации о работе системы, `warn` — для потенциально проблемных ситуаций, `error` — для ошибок, после которых приложение еще может продолжать работу, `fatal` — для критических ошибок, а `unknown` — для сообщений вне стандартных категорий. Более подробные уровни логирования, такие как `:debug`, также дают больший performance penalty, чем более строгие уровни вроде `:fatal`, потому что обрабатывается и записывается больше данных.

Источник: [Debugging Rails Applications](https://guides.rubyonrails.org/debugging_rails_applications.html), [Mastering Rails Logger - Tips for Effective Debugging](https://signoz.io/guides/rails-logger/#4-formatting-issues-in-logs)
