```
BASE_DIR = Path(__file__).resolve().parent.parent  
```

└─ вычисляет абсолютный путь к корневой папке проекта (две папки вверх от текущего файла)

```
SECRET_KEY = 'django-insecure-yey....'   
```
└─ секретный ключ для шифрования сессий и CSRF-токенов; нельзя раскрывать в публичных репозиториях

```
DEBUG = True          
```
└─ включает подробные сообщения об ошибках; в проде должен быть False

```
ALLOWED_HOSTS = []                 
```
└─ список хостов/доменов, на которых разрешён приём запросов

```
INSTALLED_APPS = [
    'django.contrib.admin',                    # административная панель
    'django.contrib.auth',                     # система аутентификации и авторизации
    'django.contrib.contenttypes',             # фреймворк для работы с типами контента
    'django.contrib.sessions',                 # механизм сессий
    'django.contrib.messages',                 # система flash-сообщений
    'django.contrib.staticfiles',              # работа со статическими файлами
]
```

```
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',          # базовые меры безопасности
    'django.contrib.sessions.middleware.SessionMiddleware',   # привязка сессий к запросам
    'django.middleware.common.CommonMiddleware',              # общие правки запросов/ответов
    'django.middleware.csrf.CsrfViewMiddleware',              # защита от CSRF-атак
    'django.contrib.auth.middleware.AuthenticationMiddleware',# добавляет user в request
    'django.contrib.messages.middleware.MessageMiddleware',   # поддержка flash-сообщений
    'django.middleware.clickjacking.XFrameOptionsMiddleware', # защита от clickjacking
]
```

```
ROOT_URLCONF = 'prj1.urls'                             
```
└─ модуль корневых URL проекта

```

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        # └─ движок шаблонов Django
        'DIRS': [],                                    # доп. папки для поиска шаблонов
        'APP_DIRS': True,                              # искать шаблоны внутри apps/<app>/templates
        'OPTIONS': {
            'context_processors': [                   # функции, добавляющие переменные в контекст шаблонов
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```

```
WSGI_APPLICATION = 'prj1.wsgi.application'             # точка входа для WSGI-сервера
```

```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',         # движок БД: SQLite
        'NAME': BASE_DIR / 'db.sqlite3',                # файл БД в корне проекта
    }
}
```

```
AUTH_PASSWORD_VALIDATORS = [
    {
        'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator',
        # └─ запрещает слишком похожие на имя пользователя пароли
    },
    {
        'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
        # └─ проверяет минимальную длину пароля
    },
    {
        'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator',
        # └─ запрещает распространённые (словесные) пароли
    },
    {
        'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator',
        # └─ запрещает полностью цифровые пароли
    },
]
```

```
LANGUAGE_CODE = 'en-us'            # язык по умолчанию

TIME_ZONE = 'UTC'                  # часовой пояс проекта

USE_I18N = True                    # включить поддержку интернационализации

USE_TZ = True                      # использовать timezone-aware объекты дат/времени

STATIC_URL = 'static/'             # URL-префикс для раздачи static-файлов
```

```
DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'
```
└─ тип поля для автогенерируемого первичного ключа (большое целое число)
