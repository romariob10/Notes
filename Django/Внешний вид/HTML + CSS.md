## 1. Создание папки `templates`
Внутри папки ***приложения***, не проекта, необходимо создать папку templates (с англ. шаблон)
## 2. Создание папки `nameofapplication`
Внутри `templates` необходимо создать папку с названием вашего приложения, т.к. при сборке все папке `templates` объединяются и могут возникнуть конфликты
## 3.  Добавление HTML
Добавляем свой HTML в подпапку `nameofapplication`
## 4. Использование 
пример
``` Python
from django.shortcuts import render

def function(request):
	return render(request, 'nameofapplication/yourhtml.html')
```
