## Оглавление
1. [Что такое HTMX](#что-такое-htmx)
2. [Установка и базовая настройка](#установка-и-базовая-настройка)
3. [Основные HTMX атрибуты](#основные-htmx-атрибуты)
4. [Детектирование HTMX запросов в Django](#детектирование-htmx-запросов-в-django)
5. [Рекомендуемая структура проекта](#рекомендуемая-структура-проекта)
6. [Примеры реализации](#примеры-реализации)
7. [Лучшие практики](#лучшие-практики)

---

## Что такое HTMX

**HTMX** — это легкая библиотека JavaScript, которая позволяет обращаться к AJAX, CSS Transitions, WebSockets и Server-Sent Events непосредственно из HTML атрибутов. Она дает Django разработчикам возможность создавать интерактивные веб-приложения практически без написания JavaScript кода.

### Основные преимущества:
- ✅ Минимум JavaScript кода — вся логика на сервере
- ✅ Встроенная в HTML синтаксис — просто добавляй атрибуты
- ✅ Естественная интеграция с Django шаблонами
- ✅ Динамическое обновление отдельных элементов без перезагрузки страницы
- ✅ Поддержка прогрессивного улучшения (Progressive Enhancement)

---

## Установка и базовая настройка

### 1. Установка необходимых пакетов

```bash
pip install django-htmx
```

### 2. Добавление в INSTALLED_APPS (settings.py)

```python
INSTALLED_APPS = [
    # ... другие приложения
    'django_htmx',
]
```

### 3. Добавление middleware (settings.py)

```python
MIDDLEWARE = [
    # ... другой middleware
    'django_htmx.middleware.HtmxMiddleware',
]
```

### 4. Подключение HTMX в базовом шаблоне (base.html)

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}Мой сайт{% endblock %}</title>
</head>
<body hx-headers='{"X-CSRFToken": "{{ csrf_token }}"}'>
    {% block content %}{% endblock %}
    
    <!-- Подключение HTMX -->
    <script src="https://unpkg.com/htmx.org@1.9.10"></script>
</body>
</html>
```

**Важно:** Атрибут `hx-headers` с CSRF токеном необходимо поместить на `<body>` для автоматической передачи токена во всех POST запросах.

---

## Основные HTMX атрибуты

### Основные методы запроса

| Атрибут     | Описание      | Пример                                              |
| ----------- | ------------- | --------------------------------------------------- |
| `hx-get`    | GET запрос    | `<button hx-get="/api/data/">Загрузить</button>`    |
| `hx-post`   | POST запрос   | `<button hx-post="/api/save/">Сохранить</button>`   |
| `hx-put`    | PUT запрос    | `<button hx-put="/api/update/">Обновить</button>`   |
| `hx-delete` | DELETE запрос | `<button hx-delete="/api/delete/">Удалить</button>` |
| `hx-patch`  | PATCH запрос  | `<button hx-patch="/api/patch/">Изменить</button>`  |

### Управление целью и обновлением

| Атрибут     | Описание                        | Значения                                                                                         |
| ----------- | ------------------------------- | ------------------------------------------------------------------------------------------------ |
| `hx-target` | Элемент, который будет обновлен | CSS селектор (ID, класс, тег)                                                                    |
| `hx-swap`   | Способ вставки ответа           | `innerHTML`, `outerHTML`, `beforebegin`, `afterbegin`, `beforeend`, `afterend`, `delete`, `none` |

### Триггеры и события

| Атрибут | Описание | Пример |
|---------|---------|--------|
| `hx-trigger` | Событие, вызывающее запрос | `click` (по умолчанию), `change`, `keyup`, `load` и т.д. |
| `hx-on` | Обработчик HTMX событий | `hx-on::htmx:afterRequest="..."` |

### Другие полезные атрибуты

| Атрибут | Описание |
|---------|---------|
| `hx-boost` | Преобразует все ссылки и формы в AJAX запросы |
| `hx-confirm` | Показывает подтверждение перед запросом |
| `hx-indicator` | Показывает индикатор загрузки во время запроса |
| `hx-disabled-elt` | Отключает элемент во время запроса |
| `hx-encoding` | Способ кодирования (по умолчанию application/x-www-form-urlencoded, для файлов используй multipart/form-data) |

### Примеры использования

```html
<!-- Простой GET запрос -->
<button hx-get="/api/items/" hx-target="#item-list" hx-swap="innerHTML">
    Загрузить список
</button>

<!-- Форма с POST запросом -->
<form hx-post="/api/save/" hx-target="#response" hx-swap="afterend">
    <input type="text" name="name" placeholder="Имя">
    <button type="submit">Отправить</button>
</form>

<!-- Удаление с подтверждением -->
<button hx-delete="/api/delete/1/" 
        hx-confirm="Вы уверены?" 
        hx-target="closest tr"
        hx-swap="outerHTML swap:1s">
    Удалить
</button>

<!-- С индикатором загрузки -->
<button hx-get="/api/data/" 
        hx-target="#content"
        hx-indicator="#spinner">
    Загрузить
</button>
<div id="spinner" class="htmx-request hidden">
    <span>Загрузка...</span>
</div>

<!-- Автоматическая загрузка при открытии страницы -->
<div hx-get="/api/content/" hx-trigger="load" hx-swap="innerHTML"></div>

<!-- Изменение при вводе -->
<input type="text" 
       hx-get="/api/search/" 
       hx-trigger="keyup changed delay:500ms"
       hx-target="#search-results">
```

---

## Детектирование HTMX запросов в Django

### В представлениях (views.py)

```python
from django.http import HttpResponse
from django.shortcuts import render

def my_view(request):
    # Проверка, пришел ли запрос от HTMX
    if request.headers.get('HX-Request'):
        # Это HTMX запрос - возвращаем только фрагмент шаблона
        return render(request, 'fragments/item.html', context)
    else:
        # Обычный запрос - возвращаем полную страницу
        return render(request, 'full_page.html', context)
```

### Использование django-htmx middleware

```python
def my_view(request):
    # После добавления middleware, доступно request.htmx
    if request.htmx:
        # HTMX запрос
        return render(request, 'fragments/item.html', context)
    else:
        # Обычный запрос
        return render(request, 'full_page.html', context)
```

### Проверка типа запроса

```python
from django.views.decorators.http import require_http_methods

@require_http_methods(["GET", "POST"])
def item_view(request):
    if request.method == 'POST' and request.htmx:
        # Обработка HTMX POST запроса
        # Вернуть обновленный фрагмент шаблона
        return render(request, 'fragments/item_updated.html', context)
    
    # Обычная обработка
    items = Item.objects.all()
    return render(request, 'items/list.html', {'items': items})
```

---

## Рекомендуемая структура проекта

### Вариант 1: Базовая структура (для простых проектов)

```
myproject/
├── manage.py
├── settings.py
├── urls.py
├── apps/
│   ├── items/
│   │   ├── models.py
│   │   ├── views.py
│   │   ├── urls.py
│   │   ├── forms.py
│   │   └── templates/
│   │       ├── items/
│   │       │   ├── list.html
│   │       │   ├── detail.html
│   │       │   └── fragments/
│   │       │       ├── item_row.html
│   │       │       ├── item_form.html
│   │       │       └── item_card.html
│   │       └── base.html
```

### Вариант 2: Продвинутая структура (для больших проектов)

```
myproject/
├── manage.py
├── settings.py
├── urls.py
├── apps/
│   ├── core/
│   │   └── templates/
│   │       └── base.html
│   │
│   ├── products/
│   │   ├── models.py
│   │   ├── views.py
│   │   ├── urls.py
│   │   ├── forms.py
│   │   ├── htmx_views.py          # ← Отдельный файл для HTMX
│   │   └── templates/
│   │       ├── products/
│   │       │   ├── list.html      # Полная страница
│   │       │   └── detail.html
│   │       │
│   │       └── products/
│   │           └── partials/      # ← HTMX фрагменты
│   │               ├── product_list.html
│   │               ├── product_item.html
│   │               ├── product_form.html
│   │               ├── product_filter.html
│   │               └── product_card.html
│   │
│   ├── orders/
│   │   ├── models.py
│   │   ├── views.py
│   │   ├── urls.py
│   │   ├── htmx_views.py
│   │   └── templates/
│   │       ├── orders/
│   │       │   └── list.html
│   │       └── orders/
│   │           └── partials/
│   │               ├── order_form.html
│   │               └── order_row.html
│   │
│   └── utils/
│       ├── responses.py           # ← Helper функции
│       └── decorators.py
```

### Вариант 3: С использованием django-template-partials (рекомендуется)

```
myproject/
├── apps/
│   └── items/
│       ├── models.py
│       ├── views.py
│       ├── urls.py
│       └── templates/
│           └── items/
│               ├── list.html      # В одном файле:
│               │                  # {% block main_content %}
│               │                  # {% partialdef item_list %}
│               │                  # {% partialdef item_form %}
│               │                  # {% partialdef confirmation %}
│               └── detail.html
```

---

## Примеры реализации

### Пример 1: Простой список с удалением

#### models.py
```python
from django.db import models

class Todo(models.Model):
    title = models.CharField(max_length=200)
    description = models.TextField(blank=True)
    completed = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return self.title
```

#### views.py
```python
from django.shortcuts import render, get_object_or_404, redirect
from .models import Todo
from django.http import HttpResponse

def todo_list(request):
    todos = Todo.objects.all()
    return render(request, 'todos/list.html', {'todos': todos})

def todo_delete(request, pk):
    todo = get_object_or_404(Todo, pk=pk)
    todo.delete()
    
    # После удаления возвращаем пустой ответ
    # и обновляем список с помощью HTMX swap
    return HttpResponse('')

def todo_toggle(request, pk):
    todo = get_object_or_404(Todo, pk=pk)
    todo.completed = not todo.completed
    todo.save()
    
    # Возвращаем обновленную строку
    return render(request, 'todos/partials/todo_item.html', {'todo': todo})
```

#### urls.py
```python
from django.urls import path
from . import views

app_name = 'todos'

urlpatterns = [
    path('', views.todo_list, name='list'),
    path('<int:pk>/delete/', views.todo_delete, name='delete'),
    path('<int:pk>/toggle/', views.todo_toggle, name='toggle'),
]
```

#### templates/todos/list.html
```html
{% extends 'base.html' %}

{% block content %}
<div class="container">
    <h1>Мои задачи</h1>
    
    <ul id="todo-list" class="todo-list">
        {% for todo in todos %}
            {% include 'todos/partials/todo_item.html' with todo=todo %}
        {% endfor %}
    </ul>
</div>
{% endblock %}
```

#### templates/todos/partials/todo_item.html
```html
<li class="todo-item" id="todo-{{ todo.id }}">
    <span class="{% if todo.completed %}completed{% endif %}">
        {{ todo.title }}
    </span>
    <button hx-post="{% url 'todos:toggle' todo.id %}"
            hx-target="#todo-{{ todo.id }}"
            hx-swap="outerHTML"
            class="btn-toggle">
        {% if todo.completed %}Отменить{% else %}Готово{% endif %}
    </button>
    <button hx-delete="{% url 'todos:delete' todo.id %}"
            hx-confirm="Удалить эту задачу?"
            hx-target="#todo-{{ todo.id }}"
            hx-swap="outerHTML swap:1s"
            class="btn-delete">
        Удалить
    </button>
</li>
```

### Пример 2: Динамический поиск

#### views.py
```python
from django.shortcuts import render
from .models import Product

def product_search(request):
    query = request.GET.get('q', '')
    
    if query:
        products = Product.objects.filter(name__icontains=query)
    else:
        products = Product.objects.none()
    
    if request.htmx:
        # Возвращаем только список результатов
        return render(request, 'products/partials/search_results.html',
                     {'products': products})
    
    # Полная страница для обычного запроса
    return render(request, 'products/search.html', {'products': products})
```

#### templates/products/search.html
```html
{% extends 'base.html' %}

{% block content %}
<div class="search-container">
    <input type="text" 
           name="q"
           placeholder="Поиск товаров..."
           hx-get="{% url 'products:search' %}"
           hx-trigger="keyup changed delay:500ms"
           hx-target="#search-results"
           class="search-input">
    
    <div id="search-results"></div>
</div>
{% endblock %}
```

#### templates/products/partials/search_results.html
```html
{% if products %}
    <ul class="search-results">
        {% for product in products %}
            <li><a href="{% url 'products:detail' product.id %}">{{ product.name }}</a></li>
        {% endfor %}
    </ul>
{% else %}
    <p class="no-results">Товары не найдены</p>
{% endif %}
```

### Пример 3: Форма с валидацией в реальном времени

#### views.py
```python
from django.shortcuts import render
from .forms import ContactForm

def contact_form(request):
    if request.method == 'POST':
        form = ContactForm(request.POST)
        if form.is_valid():
            form.save()
            return render(request, 'forms/partials/success.html')
        else:
            if request.htmx:
                # Возвращаем форму с ошибками
                return render(request, 'forms/partials/contact_form.html',
                            {'form': form})
    else:
        form = ContactForm()
    
    if request.htmx:
        return render(request, 'forms/partials/contact_form.html',
                     {'form': form})
    
    return render(request, 'forms/contact.html', {'form': form})
```

#### templates/forms/partials/contact_form.html
```html
<form hx-post="{% url 'contact' %}"
      hx-target="#form-container"
      hx-swap="outerHTML"
      class="form">
    {% csrf_token %}
    
    <div class="form-group">
        {{ form.email.label_tag }}
        {{ form.email }}
        {% if form.email.errors %}
            <div class="errors">
                {% for error in form.email.errors %}
                    <span>{{ error }}</span>
                {% endfor %}
            </div>
        {% endif %}
    </div>
    
    <div class="form-group">
        {{ form.message.label_tag }}
        {{ form.message }}
        {% if form.message.errors %}
            <div class="errors">
                {% for error in form.message.errors %}
                    <span>{{ error }}</span>
                {% endfor %}
            </div>
        {% endif %}
    </div>
    
    <button type="submit" class="btn">Отправить</button>
</form>
```

---

## Лучшие практики

### ✅ Делай правильно

1. **Разделяй полные страницы и фрагменты**
   ```python
   if request.htmx:
       return render(request, 'fragments/item.html', context)
   return render(request, 'full_page.html', context)
   ```

2. **Используй правильные статус коды HTTP**
   ```python
   if form.is_valid():
       return render(request, 'success.html')  # 200 OK
   else:
       return render(request, 'form.html', {'form': form}, status=422)
   ```

3. **Используй CSRF токен для всех POST/PUT/DELETE запросов**
   ```html
   <body hx-headers='{"X-CSRFToken": "{{ csrf_token }}"}'>
   ```

4. **Кэшируй дорогостоящие операции**
   ```python
   from django.views.decorators.cache import cache_page
   
   @cache_page(60 * 5)  # 5 минут
   def expensive_view(request):
       ...
   ```

5. **Используй индикаторы загрузки для лучшего UX**
   ```html
   <button hx-get="/api/data/" hx-indicator="#spinner">
       Загрузить
   </button>
   <div id="spinner" class="spinner htmx-request">⏳</div>
   ```

6. **Добавляй уникальные ID элементам, которые будешь обновлять**
   ```html
   <div id="item-{{ item.id }}">...</div>
   ```

### ❌ Избегай этого

1. **Не возвращай полные страницы при HTMX запросе**
   ```python
   # ❌ Неправильно
   return render(request, 'full_page.html', context)
   ```

2. **Не забывай про CSRF защиту**
   ```html
   <!-- ❌ Неправильно -->
   <button hx-post="/api/delete/">Удалить</button>
   ```

3. **Не используй сложный JavaScript без необходимости**
   ```html
   <!-- ❌ Неправильно -->
   <button onclick="submitForm(); validateData(); updateUI();">
   ```

4. **Не игнорируй SEO при использовании HTMX**
   - Убедись, что основной контент доступен в HTML
   - Используй прогрессивное улучшение

5. **Не создавай слишком много маленьких фрагментов**
   - Группируй связанные элементы
   - Избегай избыточных запросов

### 🎯 Оптимизация производительности

```python
# Использование select_related и prefetch_related
def items_list(request):
    items = Item.objects.select_related('category').prefetch_related('tags')
    return render(request, 'items/partials/list.html', {'items': items})

# Пагинация
from django.core.paginator import Paginator

def paginated_items(request):
    items = Item.objects.all()
    paginator = Paginator(items, 10)
    page_number = request.GET.get('page', 1)
    page_obj = paginator.get_page(page_number)
    return render(request, 'items/partials/page.html', {'page_obj': page_obj})
```

---

## Полезные ресурсы

- **Официальная документация HTMX**: https://htmx.org/docs/
- **django-htmx документация**: https://django-htmx.readthedocs.io/
- **django-template-partials**: https://github.com/carltongibson/django-template-partials

---

## Резюме: Необходимые изменения структуры проекта

### Минимальные изменения:

1. ✅ **Добавить папку `partials/` в templates**
   ```
   templates/
   └── app_name/
       ├── list.html (полная страница)
       └── partials/
           ├── item.html
           ├── form.html
           └── row.html
   ```

2. ✅ **Установить django-htmx**
3. ✅ **Добавить CSRF токен в base.html**
4. ✅ **Проверять request.htmx в views**

### Рекомендуемые улучшения:

1. 🔧 **Создать helper функции для ответов**
   ```python
   # utils/responses.py
   def htmx_render(request, template_name, context):
       if request.htmx:
           # Автоматически выбирает фрагмент
           template_name = template_name.replace('.html', '_htmx.html')
       return render(request, template_name, context)
   ```

2. 🔧 **Отделить HTMX views (опционально)**
   ```
   views.py      # Обычные views
   htmx_views.py # Только для HTMX
   ```

3. 🔧 **Использовать django-template-partials для организации фрагментов**

---

**Автор:** Справка по HTMX + Django  
**Дата:** 2025  
**Версия:** 1.0
