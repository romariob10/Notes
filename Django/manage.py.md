`manage.py` в Django:

1. **Настройка окружения**  
    Автоматически задаёт `DJANGO_SETTINGS_MODULE` и добавляет корень проекта в `sys.path`.
2. **Запуск команд**  
    Через `python manage.py <команда>` — `runserver`, `migrate`, `makemigrations`, `createsuperuser`, `shell`, `test` и другие.
3. **Пользовательские команды**  
    Позволяет добавлять свои скрипты в `app/management/commands/…`.
4. **Изоляция проекта**  
	Гарантирует, что команды выполняются в контексте именно этого проекта.