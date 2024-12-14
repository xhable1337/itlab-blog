---
layout: post
title: "Использование FastAPI для создания Web API"
author: evgeny
categories: [ руководство, Python, API ]
image: assets/images/fastapi.jpg
featured: true
hidden: false
toc: true
beforetoc: "В этой статье мы рассмотрим, как использовать FastAPI для создания Web API. Мы обсудим основные концепции, интеграцию с SQLAlchemy и Pydantic, а также контейнеризацию с помощью Docker."
---

# Введение

FastAPI - это современный, быстрый (высокопроизводительный) веб-фреймворк для создания API на Python 3.6+ на основе стандартов OpenAPI и JSON Schema. Он был разработан для обеспечения высокой производительности и простоты использования, что делает его отличным выбором для создания Web API.

## Основные концепции FastAPI

FastAPI использует асинхронные функции Python для обеспечения высокой производительности. Он также поддерживает автоматическую генерацию документации API с использованием OpenAPI и JSON Schema. Вот основные концепции, которые следует знать при работе с FastAPI:

### Асинхронность

FastAPI поддерживает асинхронные функции, что позволяет обрабатывать множество запросов одновременно без блокировки. Это особенно полезно для приложений, которые требуют высокой производительности и масштабируемости.

### Типизация

FastAPI использует аннотации типов Python для автоматической валидации данных и генерации документации API. Это позволяет разработчикам писать более безопасный и читаемый код.

### Pydantic

FastAPI использует Pydantic для валидации данных. Pydantic позволяет определять схемы данных с использованием аннотаций типов Python и автоматически проверяет входные данные на соответствие этим схемам.

### OpenAPI и JSON Schema

FastAPI автоматически генерирует документацию API на основе стандартов OpenAPI и JSON Schema. Это упрощает интеграцию с другими системами и инструментами, такими как Swagger UI и ReDoc.

## Пример создания простого API с FastAPI

Давайте рассмотрим пример создания простого API с использованием FastAPI:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def read_root():
    return {"message": "Hello, World!"}

@app.get("/items/{item_id}")
async def read_item(item_id: int, q: str = None):
    return {"item_id": item_id, "q": q}
```

Этот код создает простое API с двумя маршрутами: один для корневого URL и один для получения элемента по его идентификатору.

## Интеграция с SQLAlchemy

SQLAlchemy - это популярная библиотека ORM (Object-Relational Mapping) для Python, которая позволяет взаимодействовать с базами данных с использованием объектно-ориентированного подхода. FastAPI легко интегрируется с SQLAlchemy для работы с базами данных.

### Установка зависимостей

Для начала установим необходимые зависимости:

```sh
pip install sqlalchemy databases asyncpg
```

### Настройка базы данных

Создадим файл `database.py` для настройки подключения к базе данных:

```python
from sqlalchemy import create_engine, MetaData
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

DATABASE_URL = "postgresql://user:password@localhost/dbname"

engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()
metadata = MetaData()
```

### Определение моделей

Создадим файл `models.py` для определения моделей базы данных:

```python
from sqlalchemy import Column, Integer, String
from .database import Base

class Item(Base):
    __tablename__ = "items"

    id = Column(Integer, primary_key=True, index=True)
    name = Column(String, index=True)
    description = Column(String, index=True)
```

### Создание зависимостей

Создадим файл `dependencies.py` для создания зависимостей базы данных:

```python
from .database import SessionLocal

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

### Создание маршрутов

Создадим файл `main.py` для создания маршрутов API:

```python
from fastapi import FastAPI, Depends, HTTPException
from sqlalchemy.orm import Session
from . import models, schemas, dependencies

app = FastAPI()

@app.post("/items/", response_model=schemas.Item)
def create_item(item: schemas.ItemCreate, db: Session = Depends(dependencies.get_db)):
    db_item = models.Item(name=item.name, description=item.description)
    db.add(db_item)
    db.commit()
    db.refresh(db_item)
    return db_item

@app.get("/items/{item_id}", response_model=schemas.Item)
def read_item(item_id: int, db: Session = Depends(dependencies.get_db)):
    db_item = db.query(models.Item).filter(models.Item.id == item_id).first()
    if db_item is None:
        raise HTTPException(status_code=404, detail="Item not found")
    return db_item
```

## Интеграция с Pydantic

Pydantic используется для валидации данных и создания схем данных. Давайте создадим файл `schemas.py` для определения схем данных:

```python
from pydantic import BaseModel

class ItemBase(BaseModel):
    name: str
    description: str

class ItemCreate(ItemBase):
    pass

class Item(ItemBase):
    id: int

    class Config:
        orm_mode = True
```

## Контейнеризация с помощью Docker

Docker позволяет упаковать приложение и его зависимости в контейнер, который можно легко развернуть на любом сервере. Давайте создадим Dockerfile для нашего приложения:

```dockerfile
# Используем официальный образ Python
FROM python:3.9

# Устанавливаем рабочую директорию
WORKDIR /app

# Копируем файлы приложения
COPY . /app

# Устанавливаем зависимости
RUN pip install --no-cache-dir -r requirements.txt

# Запускаем приложение
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "80"]
```

Создадим файл 

docker-compose.yml

 для настройки Docker Compose:

```yaml
version: "3.7"

services:
  db:
    image: postgres:12
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: dbname
    volumes:
      - postgres_data:/var/lib/postgresql/data

  web:
    build: .
    command: uvicorn main:app --host 0.0.0.0 --port 80
    volumes:
      - .:/app
    ports:
      - "8000:80"
    depends_on:
      - db

volumes:
  postgres_data:
```

Теперь мы можем запустить наше приложение с помощью Docker Compose:

```sh
docker-compose up --build
```

## Сравнение с другими фреймворками и языками

FastAPI отличается высокой производительностью и простотой использования. Давайте сравним его с другими популярными фреймворками и языками:

### Flask

Flask - это легковесный веб-фреймворк для Python, который предоставляет минимальный набор инструментов для создания веб-приложений. В отличие от FastAPI, Flask не поддерживает асинхронные функции и автоматическую генерацию документации API. Однако Flask имеет большую экосистему и множество расширений.

### Django

Django - это мощный веб-фреймворк для Python, который предоставляет множество встроенных функций, таких как ORM, аутентификация и админ-панель. Django более сложен в использовании по сравнению с FastAPI, но он идеально подходит для крупных проектов с множеством функциональных требований.

### Express.js

Express.js - это популярный веб-фреймворк для Node.js, который предоставляет минимальный набор инструментов для создания веб-приложений. Express.js поддерживает асинхронные функции и имеет большую экосистему. Однако FastAPI превосходит Express.js по производительности благодаря использованию асинхронных функций Python.

### Spring Boot

Spring Boot - это мощный веб-фреймворк для Java, который предоставляет множество встроенных функций для создания веб-приложений. Spring Boot имеет высокую производительность и масштабируемость, но он более сложен в использовании по сравнению с FastAPI.

## Заключение

FastAPI - это мощный и простой в использовании веб-фреймворк для создания Web API на Python. Он обеспечивает высокую производительность благодаря поддержке асинхронных функций и автоматической генерации документации API. FastAPI легко интегрируется с SQLAlchemy и Pydantic, что позволяет создавать безопасные и масштабируемые приложения. Контейнеризация с помощью Docker упрощает развертывание и управление приложениями. FastAPI является отличным выбором для создания современных Web API.