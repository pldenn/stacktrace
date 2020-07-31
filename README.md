## Задача №2 - "Stack Trace"

### Легенда

К вам пришло веб-приложение, которое необходимо запустить и проверить его работоспособность.

Smoke-тест этого приложения будет заключаться в следующем: при выполнении запуска по инструкции приложение стартует и доступно по адресу http://localhost:9999 (нужно открыть браузер и "ручками" вбить адрес http://localhost:9999).

### Инструкция

1. Скачайте файл [stracktrace.jar](artifacts/stacktrace.jar)
1. Откройте терминал в каталоге, в который вы скачали файл из п.1
1. Запустите приложение командой `java -jar stacktrace.jar`

### Задача

Выполните запуск приложения по инструкции, убедитесь, что оно падает с красивым Stack Trace'ом.

Важно: приложение специально написано так, чтобы упасть! Поэтому не пробуйте его починить!

Что нужно сделать: нужно оформить баг-репорт (не забудьте скопировать **весь стек-трейс**).

Для этого:
1. Создайте новый репозиторий
1. В репозитории создайте issue (ознкомьтесь с тем, [как оформлять issue в части кода](https://help.github.com/en/github/writing-on-github/basic-writing-and-formatting-syntax#quoting-code)*)
1. Приложите к issue либо ссылку на файл, либо сам файл (`stacktrace.jar`)

Важно: не закидывайте стек-трейс как обычный код в issue! Оформляйте правильно и аккуратно.

Итого: у вас должен быть репозиторий на GitHub, в котором расположено ваше issue.

### "Заворачивание" исключений

Когда вы будете рассматривать Stack Trace, общая структура будет вот такой:

```
java.lang.IllegalStateException: Failed to execute CommandLineRunner
    ...
Caused by: org.springframework.jdbc.CannotGetJdbcConnectionException: Failed to obtain JDBC Connection; nested exception is com.mysql.cj.jdbc.exceptions.CommunicationsException: Communications link failure
    ...
Caused by: ...
```

Что это значит?

Если вы внимательно посмотрите на конструкторы исключений, то увидите, что есть возможность передавать другое исключение в качестве аргумента:

```java
public class RuntimeException extends Exception {
    ...
    public RuntimeException(String message, Throwable cause) {
        super(message, cause);
    }
    public RuntimeException(Throwable cause) {
        super(cause);
    }
    ...
}
```

Зачем это нужно?

Это позволяет организовать "заворачивание исключений", а именно: происходит какое-то исключение, его перехватывают с помощью `catch` и потом выбрасывают собственное:
```java
try {

} catch (CannotGetJdbcConnectionException e) {
    throw new IllegalStateException("CannotGetJdbcConnectionException", e);
}
```

"Переводится" это следующим образом: мы выкидываем своё исключение, потому что возникло другое исключение (в Stack Trace появляется `Caused by`).

Как вы видели из Stack Trace к текущему заданию, таких "вкладываний" может быть очень много, и самым вложенным является то, которое ниже всех по Stack Trace.

Именно так в большинстве фреймворков Checked исключения заворачивают в Unchecked, чтобы нам с вами не приходилось выносить исключения в сигнатуру или писать `try-catch`.