# Технология ORM: сравнительный анализ подходов Active Record (Django ORM) и Data Mapper (SQLAlchemy)

# ОГЛАВЛЕНИЕ

- [ВВЕДЕНИЕ](#введение)
- [1. Технология ORM: основные понятия и принципы](#1-технология-orm-основные-понятия-и-принципы)
  - [1.1. Определение и назначение ORM](#11-определение-и-назначение-orm)
  - [1.2. Преимущества и недостатки использования ORM](#12-преимущества-и-недостатки-использования-orm)
  - [1.3. Типичные проблемы при работе с ORM](#13-типичные-проблемы-при-работе-с-orm)
- [2. Паттерн Active Record и Django ORM](#2-паттерн-active-record-и-django-orm)
  - [2.1. Описание паттерна Active Record](#21-описание-паттерна-active-record)
  - [2.2. Архитектура Django ORM](#22-архитектура-django-orm)
  - [2.3. Работа со связями в Django ORM](#23-работа-со-связями-в-django-orm)
- [3. Паттерн Data Mapper и SQLAlchemy](#3-паттерн-data-mapper-и-sqlalchemy)
  - [3.1. Описание паттерна Data Mapper](#31-описание-паттерна-data-mapper)
  - [3.2. Архитектура SQLAlchemy](#32-архитектура-sqlalchemy)
  - [3.3. Unit of Work и Identity Map](#33-unit-of-work-и-identity-map)
- [4. Сравнительный анализ подходов](#4-сравнительный-анализ-подходов)
  - [4.1. Критерии сравнения](#41-критерии-сравнения)
  - [4.2. Оптимизация запросов](#42-оптимизация-запросов)
  - [4.3. Рекомендации по выбору подхода](#43-рекомендации-по-выбору-подхода)
- [ЗАКЛЮЧЕНИЕ](#заключение)
- [СПИСОК ЛИТЕРАТУРЫ](#список-литературы)

---

## ВВЕДЕНИЕ

Современная разработка ПО неразрывно связана с использованием баз данных. Разработчики сталкиваются с проблемой «object-relational impedance mismatch» — несоответствием между ООП и реляционной моделью данных.

Согласно исследованию JetBrains State of Developer Ecosystem 2023, более 70% Python-разработчиков используют ORM в своих проектах.

Схематически работа ORM:
```
Python Object ↔ ORM Layer ↔ SQL Query ↔ Database Table
```

Цель реферата — сравнительный анализ паттернов Active Record (Django ORM) и Data Mapper (SQLAlchemy).

---

## 1. Технология ORM: основные понятия и принципы

### 1.1. Определение и назначение ORM

ORM (Object-Relational Mapping) — технология, связывающая БД с концепциями ООП, создавая «виртуальную объектную базу данных» [Fowler, 2002].

Ключевые функции ORM:

- Отображение классов на таблицы (mapping)
- Преобразование объектов в записи (serialization/deserialization)
- Генерация SQL-запросов
- Управление связями (1:1, 1:N, M:N)
- Отслеживание изменений (dirty checking)

### 1.2. Преимущества и недостатки использования ORM

**Преимущества:**

| Аспект | Описание |
|--------|----------|
| Продуктивность | Сокращение кода на 30-50% |
| Независимость от СУБД | Миграция между PostgreSQL, MySQL, SQLite |
| Безопасность | Защита от SQL-инъекций |
| Сопровождаемость | Наглядная объектная модель |

**Недостатки:**

- Потеря производительности (overhead)
- Ограниченность для сложных SQL-конструкций
- «Leaky abstraction» — знание SQL всё равно необходимо

### 1.3. Типичные проблемы при работе с ORM

**Проблема N+1 запросов** — 1 запрос для списка + N запросов для связанных объектов.

**Lazy loading** — неожиданные запросы при обращении к связанным объектам.

**Управление транзакциями** — deadlock'и и утечки соединений при неправильных границах транзакций.

---

## 2. Паттерн Active Record и Django ORM

### 2.1. Описание паттерна Active Record

Active Record [Fowler, 2002] — объект инкапсулирует данные и методы персистенции.

Характеристики:

- Класс = таблица, экземпляр = строка
- Методы save(), delete(), find() в модели
- Простой API, минимум boilerplate
- Нарушение SRP (Single Responsibility Principle)

### 2.2. Архитектура Django ORM

Компоненты:

- **Model** — базовый класс, методы сохранения/удаления
- **Field** — типы данных (CharField, ForeignKey, etc.)
- **Manager** — интерфейс запросов (`Model.objects`)
- **QuerySet** — ленивый набор объектов

Пример:
```python
from django.db import models

class Article(models.Model):
    title = models.CharField(max_length=200)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    author = models.ForeignKey('Author', on_delete=models.CASCADE)
```

### 2.3. Работа со связями в Django ORM

- **ForeignKey** — один-ко-многим
- **OneToOneField** — один-к-одному
- **ManyToManyField** — многие-ко-многим (через промежуточную таблицу)

Оптимизация: `select_related()` (JOIN), `prefetch_related()` (отдельные запросы).

---

## 3. Паттерн Data Mapper и SQLAlchemy

### 3.1. Описание паттерна Data Mapper

Data Mapper [Fowler, 2002] — разделение объектов предметной области и слоя персистенции.

Характеристики:

- Соответствие принципу SRP из SOLID
- Модели = POPO (Plain Old Python Objects)
- Маппинг определяется отдельно
- Улучшенная тестируемость

### 3.2. Архитектура SQLAlchemy

Два компонента:

- **SQLAlchemy Core** — низкоуровневый SQL-конструктор
- **SQLAlchemy ORM** — высокоуровневый маппер

Пример (декларативный стиль):
```python
from sqlalchemy import Column, Integer, String, Text, ForeignKey
from sqlalchemy.orm import declarative_base, relationship

Base = declarative_base()

class Article(Base):
    __tablename__ = 'articles'
    
    id = Column(Integer, primary_key=True)
    title = Column(String(200))
    content = Column(Text)
    author_id = Column(Integer, ForeignKey('authors.id'))
    
    author = relationship('Author', back_populates='articles')
```

### 3.3. Unit of Work и Identity Map

**Unit of Work** (Session) — отслеживает изменения (new, dirty, deleted), при commit() генерирует минимальный набор SQL.

**Identity Map** — один объект Python на одну запись в рамках сессии, предотвращает проблемы согласованности.

---

## 4. Сравнительный анализ подходов

### 4.1. Критерии сравнения

| Критерий | Django ORM | SQLAlchemy |
|----------|------------|------------|
| Паттерн | Active Record | Data Mapper |
| Простота | Высокая | Средняя |
| Гибкость | Ограниченная | Высокая |
| Производительность | Хорошая | Отличная (с тюнингом) |
| Тестируемость | Средняя | Высокая |
| Экосистема | Django | Flask, FastAPI, Pyramid |

### 4.2. Оптимизация запросов

**Django ORM:**
- `select_related()` — JOIN для FK, O2O
- `prefetch_related()` — отдельные запросы для M2M
- `only()`, `defer()` — выбор полей
- `annotate()`, `aggregate()` — агрегация

**SQLAlchemy:**
- `joinedload()` — JOIN
- `subqueryload()` — подзапрос
- `selectinload()` — IN-запрос
- `raiseload()` — запрет lazy loading
- `load_only()` — выбор полей

### 4.3. Рекомендации по выбору подхода

**Django ORM рекомендуется для:**

- Проектов на Django
- Быстрого прототипирования и MVP
- CRUD-приложений с простой предметной областью
- Команд с небольшим опытом ORM

**SQLAlchemy рекомендуется для:**

- Сложной предметной области (DDD)
- Legacy баз данных
- Микросервисов на Flask/FastAPI
- Максимальной гибкости и производительности

---

## ЗАКЛЮЧЕНИЕ

Проведён сравнительный анализ паттернов Active Record и Data Mapper:

- **Active Record (Django ORM)** — простота, скорость разработки, идеален для CRUD
- **Data Mapper (SQLAlchemy)** — гибкость, разделение ответственности, чистая архитектура, Unit of Work, Identity Map

Выбор между подходами — вопрос соответствия инструмента задаче, а не «лучше/хуже».

---

## СПИСОК ЛИТЕРАТУРЫ

- **Fowler, M.** Patterns of Enterprise Application Architecture. — Addison-Wesley, 2002. — 560 p.

- **Django Software Foundation.** Django Documentation [Электронный ресурс]. — URL: https://docs.djangoproject.com/ (дата обращения: 15.12.2025).

- **SQLAlchemy.** SQLAlchemy Documentation [Электронный ресурс]. — URL: https://docs.sqlalchemy.org/ (дата обращения: 15.12.2025).

- **Copeland, R.** Essential SQLAlchemy. — O'Reilly Media, 2015. — 232 p.

- **Percival, H.** Architecture Patterns with Python / H. Percival, B. Gregory. — O'Reilly, 2020. — 304 p.

- **Martin, R.** Clean Architecture. — Prentice Hall, 2017. — 432 p.

- **Evans, E.** Domain-Driven Design. — Addison-Wesley, 2003. — 560 p.

- **JetBrains.** State of Developer Ecosystem 2023 [Электронный ресурс]. — URL: https://www.jetbrains.com/lp/devecosystem-2023/ (дата обращения: 15.12.2025).

- **Real Python.** Django ORM vs SQLAlchemy [Электронный ресурс]. — URL: https://realpython.com/tutorials/databases/ (дата обращения: 15.12.2025).

- **Python Software Foundation.** PEP 249 [Электронный ресурс]. — URL: https://peps.python.org/pep-0249/ (дата обращения: 15.12.2025).

- **PostgreSQL.** PostgreSQL Documentation [Электронный ресурс]. — URL: https://www.postgresql.org/docs/ (дата обращения: 15.12.2025).

- **Alembic.** Alembic Documentation [Электронный ресурс]. — URL: https://alembic.sqlalchemy.org/ (дата обращения: 15.12.2025).
