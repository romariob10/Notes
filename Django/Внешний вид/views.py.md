## 1. Вывод маленьких кусочков HTML
Этот файл отвечает за внешний вид, для примера работы выведем `Этот конспект сделан Романом`

``` Python
from django.shortcuts import render
from django.http import HttpResponse  

def function(request):
	return HttpResponse("<h1>Этот конспект сделан Романом</h1>")
```

В `HttpResponse` можно вставит любой `html` код

## 2. Полноценная работа с HTML

Для полноценной работай с [[HTML + CSS]], мы можем использовать render
Дополнительный функционал, реализуется через [[Jinja]]

Пример простого использования:

``` Python
from django.shortcuts import render

def index(request):
	return render(request, 'main/index.html')
```
