Этот файл отвечает за внешний вид, для проверки работы выведем `Этот конспект сделан Романом`

``` Python
from django.shortcuts import render
from django.http import HttpResponse  

def function(request):
	return HttpResponse("<h1>Этот конспект сделан Романом</h1>")
```

В `HttpResponse` можно вставит любой `html` код