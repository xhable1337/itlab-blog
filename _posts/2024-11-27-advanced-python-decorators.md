---
layout: post
title:  "Продвинутые декораторы в Python: концепции и примеры"
author: evgeny
categories: [ руководство, Python ]
image: assets/images/python-decorators.jpg
featured: false
hidden: false
toc: true
beforetoc: "В этой статье мы рассмотрим продвинутые концепции декораторов в Python. Мы обсудим, как они работают, и предоставим примеры их использования в различных сценариях."
---

Декораторы в Python - это мощный инструмент, который позволяет изменять поведение функций или классов без изменения их кода. Они широко используются в различных библиотеках и фреймворках для добавления функциональности, такой как логирование, кэширование и контроль доступа. В этой статье мы рассмотрим продвинутые концепции декораторов, их внутреннее устройство и примеры использования в реальных проектах.

# Основы декораторов

Декораторы - это функции, которые принимают другую функцию в качестве аргумента и возвращают новую функцию с измененным поведением. Они определяются с помощью символа `@` перед именем функции. Например, следующий декоратор добавляет простое логирование к функции:

```python
def log_decorator(func):
    def wrapper(*args, **kwargs):
        print(f"Вызов функции {func.__name__} с аргументами {args} и {kwargs}")
        result = func(*args, **kwargs)
        print(f"Функция {func.__name__} вернула {result}")
        return result
    return wrapper

@log_decorator
def add(a, b):
    return a + b

add(2, 3)
```

В этом примере декоратор `log_decorator` оборачивает функцию `add`, добавляя логирование до и после вызова функции. Это позволяет легко добавлять дополнительную функциональность к функциям без изменения их кода.

## Декораторы с аргументами

Иногда требуется передавать аргументы в декоратор. Для этого декоратор должен быть функцией, которая возвращает другую функцию-декоратор. Рассмотрим пример декоратора, который принимает аргумент и использует его для изменения поведения функции:

```python
def repeat_decorator(times):
    def decorator(func):
        def wrapper(*args, **kwargs):
            for _ in range(times):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat_decorator(3)
def greet(name):
    print(f"Привет, {name}!")

greet("Евгений")
```

В этом примере декоратор `repeat_decorator` принимает аргумент `times`, который определяет количество повторений вызова функции. Декоратор `decorator` оборачивает функцию `greet`, вызывая ее указанное количество раз.

## Декораторы классов

Декораторы могут использоваться не только для функций, но и для классов. Декораторы классов позволяют изменять поведение методов класса или добавлять новые методы. Рассмотрим пример декоратора, который добавляет метод к классу:

```python
def add_method_decorator(method):
    def decorator(cls):
        setattr(cls, method.__name__, method)
        return cls
    return decorator

def new_method(self):
    print("Это новый метод!")

@add_method_decorator(new_method)
class MyClass:
    def existing_method(self):
        print("Это существующий метод")

obj = MyClass()
obj.existing_method()
obj.new_method()
```

В этом примере декоратор `add_method_decorator` принимает метод `new_method` и добавляет его к классу `MyClass`. Это позволяет динамически добавлять методы к классам без изменения их исходного кода.

## Вложенные декораторы

Декораторы могут быть вложенными, что позволяет комбинировать несколько декораторов для одной функции или класса. Рассмотрим пример функции, обернутой двумя декораторами:

```python
def uppercase_decorator(func):
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        return result.upper()
    return wrapper

def exclamation_decorator(func):
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        return result + "!"
    return wrapper

@uppercase_decorator
@exclamation_decorator
def greet(name):
    return f"Привет, {name}"

print(greet("Евгений"))
```

В этом примере функция `greet` обернута двумя декораторами: `uppercase_decorator` и `exclamation_decorator`. Декораторы применяются в порядке, обратном их определению, что позволяет комбинировать их эффекты.

## Практическое применение декораторов

### Логирование

Логирование - это одна из самых распространенных задач, для которой используются декораторы. Рассмотрим пример декоратора, который добавляет логирование к функции:

```python
import logging

def log_decorator(func):
    def wrapper(*args, **kwargs):
        logging.info(f"Вызов функции {func.__name__} с аргументами {args} и {kwargs}")
        result = func(*args, **kwargs)
        logging.info(f"Функция {func.__name__} вернула {result}")
        return result
    return wrapper

@log_decorator
def add(a, b):
    return a + b

logging.basicConfig(level=logging.INFO)
add(2, 3)
```

В этом примере декоратор `log_decorator` использует модуль `logging` для записи информации о вызове функции и ее результате. Это позволяет легко добавлять логирование к функциям без изменения их кода.

### Кэширование

Кэширование - это еще одна распространенная задача, для которой используются декораторы. Рассмотрим пример декоратора, который кэширует результаты функции:

```python
def cache_decorator(func):
    cache = {}
    def wrapper(*args):
        if args in cache:
            return cache[args]
        result = func(*args)
        cache[args] = result
        return result
    return wrapper

@cache_decorator
def fibonacci(n):
    if n in (0, 1):
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

print(fibonacci(10))
```

В этом примере декоратор `cache_decorator` кэширует результаты функции `fibonacci`, что позволяет избежать повторных вычислений и улучшить производительность.

### Контроль доступа

Контроль доступа - это еще одна задача, для которой могут использоваться декораторы. Рассмотрим пример декоратора, который проверяет права доступа перед вызовом функции:

```python
def permission_required(permission):
    def decorator(func):
        def wrapper(user, *args, **kwargs):
            if not user.has_permission(permission):
                raise PermissionError("Доступ запрещен")
            return func(user, *args, **kwargs)
        return wrapper
    return decorator

class User:
    def __init__(self, permissions):
        self.permissions = permissions

    def has_permission(self, permission):
        return permission in self.permissions

@permission_required("admin")
def delete_user(user, user_id):
    print(f"Пользователь {user_id} удален")

admin_user = User(permissions=["admin"])
regular_user = User(permissions=[])

delete_user(admin_user, 1)
try:
    delete_user(regular_user, 1)
except PermissionError as e:
    print(e)
```

В этом примере декоратор `permission_required` проверяет, имеет ли пользователь необходимые права доступа перед вызовом функции `delete_user`. Это позволяет легко добавлять контроль доступа к функциям без изменения их кода.

## Заключение

Декораторы в Python - это мощный инструмент, который позволяет изменять поведение функций и классов без изменения их исходного кода. В этой статье мы рассмотрели продвинутые концепции декораторов, их внутреннее устройство и примеры использования в реальных проектах. Декораторы могут использоваться для различных задач, таких как логирование, кэширование и контроль доступа, что делает их незаменимым инструментом для разработчиков. Понимание и умение использовать декораторы позволяет писать более гибкий и расширяемый код, что является важным навыком для любого Python-разработчика.