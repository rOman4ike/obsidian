> [!info] TL;DR  
> Rails Logger - встроенная система логирования в Ruby on Rails для debugging и monitoring. Она пишет сообщения в log-файлы по environment, поддерживает severity levels (`debug`, `info`, `warn`, `error`, `fatal`, `unknown`), позволяет настраивать format и log rotation, а также добавлять context через `TaggedLogging`. Из входных данных следует, что verbose logging и частые вызовы logger могут ухудшать performance, поэтому логировать стоит ключевые события, избегая sensitive data.

## Refs

1. [Debugging Rails Applications](https://guides.rubyonrails.org/debugging_rails_applications.html)
2. [Mastering Rails Logger - Tips for Effective Debugging](https://signoz.io/guides/rails-logger/#4-formatting-issues-in-logs)
3. [Supercharge your Rails logs with tags](https://thoughtbot.com/blog/supercharge-your-rails-logs-with-tags)
4. [Active Support Tagged Logging](https://api.rubyonrails.org/classes/ActiveSupport/TaggedLogging.html)

## API

- `Rails.logger` - основной logger Rails для записи runtime events, errors, warnings и custom messages. Источники: [Debugging Rails Applications](https://guides.rubyonrails.org/debugging_rails_applications.html), [Mastering Rails Logger - Tips for Effective Debugging](https://signoz.io/guides/rails-logger/#4-formatting-issues-in-logs)
- `Rails.logger.level` - severity level logger; во входных данных указано соответствие: `debug: 0`, `info: 1`, `warn: 2`, `error: 3`, `fatal: 4`, `unknown: 5`. Источник: входные данные
- `Logger::Formatter#call(severity, time, progname, msg)` - hook для custom log format. Источник: [Mastering Rails Logger - Tips for Effective Debugging](https://signoz.io/guides/rails-logger/#4-formatting-issues-in-logs)
- `Logger.new(STDOUT)` / `Logger.new('log/production.log', 10, 100.megabytes)` - создание logger с выводом в `STDOUT` и настройкой log rotation. Источник: [Mastering Rails Logger - Tips for Effective Debugging](https://signoz.io/guides/rails-logger/#4-formatting-issues-in-logs)
- `Rails.logger.tagged(...)` / `ActiveSupport::TaggedLogging` - добавление tags к log messages; поддерживаются nested tags, а `.blank?` tag не добавляется. Источники: [Supercharge your Rails logs with tags](https://thoughtbot.com/blog/supercharge-your-rails-logs-with-tags), [Active Support Tagged Logging](https://api.rubyonrails.org/classes/ActiveSupport/TaggedLogging.html)
- `debug` gem - debugger, который в Rails 7 включается в новые приложения CRuby и готов к использованию в `development` и `test`. Источник: [Debugging Rails Applications](https://guides.rubyonrails.org/debugging_rails_applications.html)

## Topics

### Definition

Rails Logger - default logging system в Ruby on Rails, который помогает отслеживать behaviour приложения. Он фиксирует system events, errors, warnings и custom log messages и используется для debugging и monitoring.

### How it Works

Rails Logger пишет сообщения в log files внутри Rails application, обычно в `log/` с разделением по environment, например `development.log`, `test.log` и `production.log`.

Severity levels определяют, какие сообщения будут записаны:

- `debug` - подробная информация для диагностики проблем
- `info` - общая информация о работе системы
- `warn` - потенциально вредные ситуации
- `error` - ошибки, после которых приложение может продолжить работу
- `fatal` - критические ошибки, которые могут привести к остановке приложения
- `unknown` - сообщения, не попадающие в другие категории

Формат сообщений можно менять через custom formatter. Во входных данных указано, что метод `call` у `CustomLoggerFormatter` вызывается внутри `Logger` каждый раз, когда обрабатывается log message.

Для добавления context можно использовать `Rails.logger.tagged("tag")`, чтобы префиксовать сообщения tags. Tags можно вкладывать друг в друга.

### Usage

Rails Logger используется для записи ключевых событий в workflow приложения, для conditional logging и для context-rich messages, упрощающих troubleshooting.

Типовые сценарии из входных данных:

- логирование создания и ошибок при сохранении `Order` в controller
- логирование `after_create`, `after_update`, `after_destroy` в model
- логирование validation errors
- выделение логов конкретной job через `Rails.logger.tagged("MyJob")`
- настройка log rotation в production

### Examples

Custom log format:

```ruby
class CustomLoggerFormatter < Logger::Formatter
  def call(severity, time, progname, msg)
    "#{time.to_s(:db)} #{severity} #{msg}\n"
  end
end

Rails.logger = Logger.new(STDOUT)
Rails.logger.formatter = CustomLoggerFormatter.new
```

Tagged logging:

```ruby
Rails.logger.tagged("tag") do
  Rails.logger.info("message")
end
```

Logging in controller:

```ruby
class OrdersController < ApplicationController
  def create
    @order = Order.new(order_params)

    Rails.logger.info("Creating order for user: #{current_user.id}")

    if @order.save
      Rails.logger.info("Order #{@order.id} created successfully")
      redirect_to @order, notice: 'Order was successfully created.'
    else
      Rails.logger.warn("Order creation failed: #{@order.errors.full_messages}")
      render :new
    end
  end
end
```

### When to Use

Rails Logger стоит использовать, когда нужно:

- диагностировать проблемы и понимать runtime behaviour приложения
- мониторить важные события и этапы workflow
- логировать model events, validations и callbacks
- изолировать logs конкретной job или участка execution через tags
- контролировать размер log files в production через rotation

### Best Practices

- Не логировать sensitive data, включая passwords и credit card numbers
- Использовать structured logging, например JSON-formatted logs, если нужен machine-readable output
- Добавлять logging в background jobs, например в `Sidekiq` или `ActiveJob`
- Выбирать подходящий log level, чтобы контролировать плотность информации
- Помнить о performance impact: `:debug` создает больший performance penalty, чем `:fatal`, а слишком частые вызовы logger могут быть проблемой
- Делать messages достаточно контекстными для troubleshooting

### Summary

Rails Logger из входных данных представлен как базовый инструмент Rails для debugging и monitoring. Его ключевые возможности - severity levels, custom formatting, tagged logging и log rotation. Практическая ценность достигается за счет логирования ключевых событий, добавления context и контроля объема логов с учетом performance и security.

## Research Questions

### В чем разница между лог левелами

Лог уровни различаются степенью важности сообщений и тем, сколько информации попадает в logs: `debug` предназначен для максимально подробной диагностики, `info` для обычных operational messages, `warn` для потенциальных проблем, `error` для ошибок, после которых приложение еще может работать, `fatal` для критических сбоев, а `unknown` для сообщений вне стандартных категорий. Из входных данных также следует, что более verbose уровень вроде `debug` дает больший performance penalty, чем `fatal`, потому что в лог попадает больше вычисляемых и записываемых строк. Источники: [Mastering Rails Logger - Tips for Effective Debugging](https://signoz.io/guides/rails-logger/#4-formatting-issues-in-logs), [Debugging Rails Applications](https://guides.rubyonrails.org/debugging_rails_applications.html)
