Этот файл отвечает за внешний вид, для проверки работы выведем `Этот конспект сделан Романом`

``` Python
from django.shortcuts import render
from django.http import HttpResponse  

def index(request):
	return HttpResponse("Этот конспект сделан Романом")
```

В `HttpResponse` можно вставит любой `html` код