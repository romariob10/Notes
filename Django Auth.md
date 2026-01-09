## Django Auth: Полный разбор работы системы аутентификации

## Архитектура и компоненты

Django auth система состоит из трёх взаимодействующих частей: **User модель**, **Password Hasher**, и **Session Framework**. Вместе они создают безопасный механизм для сохранения пользователей и управления их сессиями без хранения паролей в открытом виде.

---

## 1. Сохранение пользователя и пароля

## User модель структура

Django использует `AbstractBaseUser` как основу для User модели. Ключевые поля:

|Поле|Тип|Описание|
|---|---|---|
|`id`|AutoField|Первичный ключ|
|`password`|CharField (255)|Хеш пароля в специальном формате|
|`last_login`|DateTimeField|Последний логин (nullable)|
|`is_active`|BooleanField|Активен ли аккаунт|
|`username`|CharField (150, unique)|Уникальный идентификатор|
|`email`|EmailField|Email адрес|
|`first_name`, `last_name`|CharField|Имя и фамилия|
|`is_staff`, `is_superuser`|BooleanField|Права доступа|
|`date_joined`|DateTimeField|Дата создания|

## Как сохраняется пароль: PBKDF2-HMAC-SHA256youtube​

Пароль **никогда** не сохраняется в открытом виде. Django использует **PBKDF2** (Password-Based Key Derivation Function) с HMAC-SHA256:

**Формат в БД:**

text

`algorithm$iterations$salt$hash`

**Пример реального пароля:**

text

`pbkdf2_sha256$870000$wyEbTcPvbTOet9npIyftqZ$Xv274JNGq1ZMQQHzmZx8q5n+Nly/U5Wf1WYLRO4d8YY=`

**Разбор компонентов:**

|Компонент|Значение|Описание|
|---|---|---|
|Алгоритм|`pbkdf2_sha256`|PBKDF2 с HMAC-SHA256|
|Итерации|`870000`|Кол-во раз применить функцию (повышается с версией)|
|Соль (Salt)|`wyEbTcPvbTOet9npIyftqZ`|Base64 random 128-bit значение|
|Хеш|`Xv274JNGq1ZMQQHzmZx8q5n+...`|Base64 32-байтовый выход|

**Количество итераций по версиям:**[djangoproject+3](https://docs.djangoproject.com/en/5.2/topics/auth/passwords/)​

- Django 1.4-1.10: 10,000 итераций
    
- Django 1.11-2.0: 12,000 итераций
    
- Django 2.1-3.1: 216,000 итераций
    
- Django 3.2+: 260,000 итераций
    
- Django 5.0+: 870,000 итераций
    

Итерации увеличиваются каждый выпуск приблизительно на 10%, чтобы противостоять росту вычислительной мощности.

## Как происходит хеширование (set_password)[djangoproject+1](https://docs.djangoproject.com/en/6.0/topics/auth/passwords/)​

Когда вы создаёте пользователя через `user.set_password()`:

python

`from django.contrib.auth.hashers import make_password # Внутри set_password: raw_password = "mySecurePassword123"  # открытый пароль от пользователя # 1. Генерируется случайная соль (128 бит) salt = secrets.token_bytes(16)  # ~17 символов в base64 # 2. PBKDF2 применяется с солью 870,000 раз pbkdf2_result = hashlib.pbkdf2_hmac(     'sha256',    raw_password.encode('utf-8'),    salt.encode('utf-8'),    870000,    dklen=32  # 32 байта = 256 бит ) # 3. Результат кодируется в base64 hash_b64 = base64.b64encode(pbkdf2_result).decode('ascii') # 4. Собирается финальная строка password_field = f"pbkdf2_sha256$870000${salt}${hash_b64}" # Этот хеш сохраняется в БД, оригинальный пароль уничтожается`

---

## 2. Как работает аутентификация (login)

## Полный цикл authenticate → login → session

text

`1. HTTP POST с username и password    ↓ 2. authenticate(request, username=..., password=...)    - Проходит по AUTHENTICATION_BACKENDS (по умолчанию [ModelBackend])   - ModelBackend.authenticate():     a) Ищет User по username в БД     b) Вызывает user.check_password(submitted_password)     c) Если пароль верен → возвращает user объект     d) Если неверен → возвращает None   ↓ 3. Если authenticate вернул user, вызывается login(request, user)    - Сохраняет в session:     * '_auth_user_id' = user.pk (например, 42)     * '_auth_user_backend' = 'django.contrib.auth.backends.ModelBackend'     * '_auth_user_hash' = HMAC пароля (для обнаружения изменений)   ↓ 4. SessionMiddleware в process_response:    - Кодирует session dict в JSON   - Сохраняет в django_session таблицу   - Отправляет клиенту Set-Cookie: sessionid=<random_key>   ↓ 5. На каждом следующем запросе:    - SessionMiddleware берёт sessionid из cookie   - Загружает данные из django_session таблицы   - AuthenticationMiddleware вызывает get_user(request)   - get_user восстанавливает User объект из session['_auth_user_id']`

## Проверка пароля: check_password[djangoproject+2](https://docs.djangoproject.com/en/5.0/_modules/django/contrib/auth/hashers/)​

python

`def check_password(raw_password, encoded_hash):     """    Параметры:    - raw_password: пароль, введённый пользователем (открытый текст)    - encoded_hash: хеш из БД (pbkdf2_sha256$870000$salt$hash)    """         # 1. Разбор хеша из БД    algorithm, iterations, salt, stored_hash = encoded_hash.split('$')    # algorithm='pbkdf2_sha256', iterations='870000', salt='wyEbT...',    # stored_hash='Xv274...'         # 2. Повторное вычисление с теми же параметрами    computed_hash = pbkdf2_hmac(        'sha256',        raw_password.encode('utf-8'),  # пароль из формы        salt.encode('utf-8'),           # та же соль        int(iterations),                # те же итерации        dklen=32    )    computed_hash_b64 = base64.b64encode(computed_hash).decode()         # 3. КРИТИЧНО: Сравнение в constant-time[42]    # Это предотвращает timing attacks    return hmac.compare_digest(stored_hash, computed_hash_b64)         # ПОЧЕМУ constant_time_compare?    # Обычное == сравнение выходит рано, если первый символ не совпадает    # Злоумышленник может измерить время ответа и постепенно угадать хеш    # constant_time_compare всегда сравнивает ВСЕ символы`

**Почему это работает:**

- На вход даются данные из БД и от пользователя
    
- Если пользователь ввёл правильный пароль → вычисленный хеш совпадёт с сохранённым
    
- Если неправильный → хеши отличаются (даже на 1 символ)
    
- Сравнение происходит за одинаковое время независимо от совпадения
    

---

## 3. Сессии: как они хранятся в БД

## Таблица django_session[eli.thegreenplace+1](https://eli.thegreenplace.net/2011/06/29/django-sessions-part-ii-how-sessions-work)​

sql

`CREATE TABLE django_session (     session_key VARCHAR(40) PRIMARY KEY,    session_data TEXT NOT NULL,    expire_date DATETIME NOT NULL );`

**Структура полей:**

|Поле|Тип|Пример|Описание|
|---|---|---|---|
|`session_key`|CharField(40, PK)|`2b1189a188b44ad18c35e113ac6ceead`|Уникальный ID сессии (32 chars hex)|
|`session_data`|TextField|`KGRwMQpTJ19hdXRoX3VzZXJfaWQnCnAyCkkxCnMuMT...`|Base64 JSON с данными|
|`expire_date`|DateTimeField|`2025-12-22 17:45:30`|Когда сессия истёкает|

## Что хранится в session_data[eli.thegreenplace+1](https://eli.thegreenplace.net/2011/07/17/django-sessions-part-iii-user-authentication)​

После логина session_data (в развёрнутом виде) содержит:

python

`{     '_auth_user_id': '42',                    # ID пользователя в БД    '_auth_user_backend': 'django.contrib.auth.backends.ModelBackend',    '_auth_user_hash': 'a7f3e9c1b...',       # HMAC пароля текущего юзера    # Любые другие данные, которые вы добавите:    'cart_items': ['product_1', 'product_2'],    'preferences': {'language': 'ru', 'theme': 'dark'} }`

## Процесс сохранения сессии[djangoproject+1](https://docs.djangoproject.com/en/5.2/topics/http/sessions/)​

text

`1. Пользователь выполняет действие:    request.session['user_choice'] = 'option_a'    2. Django отмечает session.modified = True     3. В конце обработки запроса (process_response):    - Кодирует session dict в JSON   - Кодирует JSON в base64 (для безопасности)   - Сохраняет в django_session таблицу   - Если session.session_key не установлен:     a) Генерирует новый 32-символьный random ключ     b) Вставляет новую строку в БД   - Если session.session_key уже есть:     a) UPDATE существующую строку    4. Отправляет HTTP Set-Cookie:    Set-Cookie: sessionid=<session_key>;               expires=<expire_date>;               Path=/;               HttpOnly;               Secure (в production)`

## Когда сессия сохраняется[djangoproject+1](https://docs.djangoproject.com/en/6.0/topics/http/sessions/)​

python

`# По умолчанию Django сохраняет сессию ТОЛЬКО если её модифицировали: request.session['foo'] = 'bar'  # СОХРАНИТСЯ (было изменение) del request.session['foo']      # СОХРАНИТСЯ (было изменение) request.session['foo']['bar'] = 'baz'  # НЕ сохранится! # Изменяется только вложенный dict, сама session не отмечена modified # Решение: request.session.modified = True # ИЛИ в settings.py (сохранять при каждом запросе): SESSION_SAVE_EVERY_REQUEST = True  # Медленнее, но гарантирует сохранение`

---

## 4. SessionMiddleware: под капотом[djangoproject+2](https://docs.djangoproject.com/en/1.8/_modules/django/contrib/sessions/middleware/)​

## Как middleware перехватывает запрос

python

`# settings.py MIDDLEWARE порядок (часть) MIDDLEWARE = [     'django.middleware.security.SecurityMiddleware',    'django.contrib.sessions.middleware.SessionMiddleware',  # ЗДЕСЬ    'django.middleware.common.CommonMiddleware',    'django.middleware.csrf.CsrfViewMiddleware',    'django.contrib.auth.middleware.AuthenticationMiddleware',    # ... ] # process_request (перед view): def process_request(request):     # 1. Читает cookie 'sessionid' (или SESSION_COOKIE_NAME)    session_key = request.COOKIES.get('sessionid', None)         # 2. Создаёт SessionStore объект    request.session = SessionStore(session_key)    # SessionStore - это dict-like объект, который лениво загружает данные         # На данном этапе БД ещё не запрошена (lazy loading) # view выполняется # process_response (после view): def process_response(request, response):     if request.session.modified or SESSION_SAVE_EVERY_REQUEST:        # Сохраняет session в БД и отправляет cookie клиенту        request.session.save()                 # Добавляет в ответ:        # Set-Cookie: sessionid=<key>; Path=/; HttpOnly; Secure;        # Max-Age=1209600; expires=<date>`

## Lazy loading сессии

python

`# Когда вы впервые обращаетесь к сессии, происходит это: @property def _session(self):     if '_session_dict' not in self.__dict__:        # Только СЕЙЧАС происходит запрос в БД!        self._session_dict = self.load()  # SELECT из django_session    return self._session_dict # До обращения к request.session никакие запросы в БД не выполняются # Это оптимизирует производительность для view'ей, не использующих сессию`

---

## 5. Можно ли использовать произвольное поле для сессий в БД?[djangoproject+1](https://docs.djangoproject.com/en/5.2/topics/http/sessions/)​

## Да, это полностью поддерживается![djangoproject](https://docs.djangoproject.com/en/5.2/topics/http/sessions/)​

Вы можете создать **custom Session модель** с дополнительными полями:

**Шаг 1: Создать custom Session модель**

python

`# myapp/models.py from django.contrib.sessions.base_session import AbstractBaseSession from django.db import models class CustomSession(AbstractBaseSession):     """Стандартные поля из AbstractBaseSession:    - session_key (40 chars, PK)    - session_data (TextField)    - expire_date (DateTime)         Добавляем свои:    """    user_id = models.IntegerField(null=True, db_index=True)    # Теперь можно быстро найти все сессии пользователя!    # А не искать в session_data         ip_address = models.GenericIPAddressField(null=True)    user_agent = models.TextField(blank=True)    created_at = models.DateTimeField(auto_now_add=True)         @classmethod    def get_session_store_class(cls):        from django.contrib.sessions.backends.db import SessionStore        return SessionStore`

**Шаг 2: Создать custom SessionStore**

python

`# myapp/backends.py from django.contrib.sessions.backends.db import SessionStore as DBSessionStore class CustomSessionStore(DBSessionStore):     @classmethod    def get_model_class(cls):        from myapp.models import CustomSession        return CustomSession         def create_model_instance(self, data):        """Переопределяем создание модели для автоматического сохранения user_id"""        obj = super().create_model_instance(data)                 # Автоматически сохранить user_id из session data        try:            user_id = int(data.get('_auth_user_id'))        except (ValueError, TypeError):            user_id = None                 obj.user_id = user_id        return obj`

**Шаг 3: Настроить settings.py**

python

`# settings.py SESSION_ENGINE = 'myapp.backends.CustomSessionStore' # Указываем на MODULE (.py файл), НЕ на класс! # Убедитесь, что в INSTALLED_APPS есть ваше приложение: INSTALLED_APPS = [     'django.contrib.contenttypes',    'django.contrib.sessions',    'myapp',  # ДО app, которое использует сессии ]`

**Шаг 4: Создать migration и применить**

bash

`python manage.py makemigrations python manage.py migrate`

**Теперь вы можете делать запросы:**

python

`# Найти все активные сессии пользователя 42 from myapp.models import CustomSession user_sessions = CustomSession.objects.filter(user_id=42) # Выкинуть сессию конкретного пользователя CustomSession.objects.filter(user_id=42, ip_address='192.168.1.1').delete() # Получить сессии, созданные более суток назад from django.utils import timezone from datetime import timedelta old_sessions = CustomSession.objects.filter(     created_at__lt=timezone.now() - timedelta(days=1) )`

---

## 6. Хеширование в действии: конкретный пример

## Полный цикл: от пароля в форме до сохранения в БД

python

`# Этап 1: РЕГИСТРАЦИЯ # form.py from django.contrib.auth.models import User user = User.objects.create_user(     username='alex',    email='alex@example.com',    password='MySecure123Pass'  # Открытый пароль в памяти ) # Внутри create_user() вызывается user.set_password('MySecure123Pass') # Это НЕМЕДЛЕННО: # 1. Генерирует соль (16 random байт) # 2. Применяет PBKDF2-HMAC-SHA256 870k раз # 3. Сохраняет только результат в user.password # 4. Оригинальный пароль забывается # В БД сохранится: # user.password = 'pbkdf2_sha256$870000$RaNd0mSaltHere$Xv274JNGq1ZM...' print(user.password) # Output: pbkdf2_sha256$870000$...$... # -------- # Этап 2: ЛОГИН from django.contrib.auth import authenticate # Пользователь заполнил форму: raw_password = 'MySecure123Pass'  # Из request.POST['password'] user = authenticate(request, username='alex', password=raw_password) # Внутри ModelBackend.authenticate(): # 1. user = User.objects.get(username='alex') #    → Получает из БД user.password = 'pbkdf2_sha256$870000$RaNd0m...$Xv274...' # # 2. user.check_password(raw_password) #    → Распаковывает хеш из БД: algorithm, iterations, salt, stored_hash #    → Вычисляет: pbkdf2('MySecure123Pass', salt, 870000) → computed_hash #    → Сравнивает: hmac.compare_digest(stored_hash, computed_hash) #    → Возвращает True или False if user is not None:     from django.contrib.auth import login    login(request, user)    # В session['_auth_user_id'] = 42    # В session['_auth_user_hash'] = HMAC(password_field)    # → Сохранится в django_session`

---

## 7. PASSWORD_HASHERS: настройка алгоритмов[django-collin.readthedocs+3](https://django-collin.readthedocs.io/en/latest/topics/auth/passwords.html)​

python

`# settings.py # По умолчанию (Django 5.0+): PASSWORD_HASHERS = [     'django.contrib.auth.hashers.PBKDF2PasswordHasher',      # Для новых паролей    'django.contrib.auth.hashers.PBKDF2SHA1PasswordHasher',  # Для старых    'django.contrib.auth.hashers.Argon2PasswordHasher',      # Если установлена argon2-cffi    'django.contrib.auth.hashers.BCryptSHA256PasswordHasher',# Если установлена bcrypt    'django.contrib.auth.hashers.ScryptPasswordHasher',      # Если установлена scrypt ] # ПОРЯДОК ВАЖЕН! # - Первый используется для новых паролей # - Остальные используются для проверки старых паролей # Пример: переход на Argon2 PASSWORD_HASHERS = [     'django.contrib.auth.hashers.Argon2PasswordHasher',      # НОВЫЕ пароли    'django.contrib.auth.hashers.PBKDF2PasswordHasher',      # Проверка СТАРЫХ    'django.contrib.auth.hashers.BCryptSHA256PasswordHasher', ] # Пользователи с PBKDF2 паролями смогут логиниться # При успешном логине пароль автоматически перехешируется в Argon2 # Это называется "password upgrade" # Увеличить количество итераций для PBKDF2: from django.contrib.auth.hashers import PBKDF2PasswordHasher class MyPBKDF2PasswordHasher(PBKDF2PasswordHasher):     iterations = PBKDF2PasswordHasher.iterations * 2  # В 2 раза больше PASSWORD_HASHERS = [     'myapp.hashers.MyPBKDF2PasswordHasher',    'django.contrib.auth.hashers.PBKDF2PasswordHasher', ]`

---

## 8. Безопасность и best practices

## Чего Django НЕ делает

- **Не отправляет пароли по HTTP** - только по HTTPS (настраивается через SESSION_COOKIE_SECURE)
    
- **Не логирует пароли** - они отфильтровываются в логах и ошибках
    
- **Не дублирует пароли** - если вы меняете пароль, старый уничтожается
    
- **Не хранит session ID в URL** - только в cookies (защита от Referer header утечек)
    

## Обновление session auth при изменении пароля[fossies](https://fossies.org/dox/django-5.2.7/namespacedjango_1_1contrib_1_1auth.html)​

python

`from django.contrib.auth import update_session_auth_hash # Когда пользователь меняет свой пароль: user.set_password(new_password) user.save() # БЕЗ этого вызова пользователь будет разлогинен! # Потому что _auth_user_hash в session больше не совпадает update_session_auth_hash(request, user) # Это обновляет _auth_user_hash в текущей сессии # Так пользователь остаётся залогинен с новым паролем`

---

## 9. Отладка и инспекция сессии

python

`# Посмотреть содержимое сессии в view def debug_view(request):     print("Session key:", request.session.session_key)    # Output: 2b1189a188b44ad18c35e113ac6ceead         print("Session data (raw, кодированное):")    print(request.session._session_cache)         print("Decoded session:")    from django.contrib.sessions.models import Session    s = Session.objects.get(session_key=request.session.session_key)    print(s.get_decoded())    # Output: {'_auth_user_id': '42', '_auth_user_backend': '...'}         print("Current user from session:")    print(request.user)  # Восстанавливается из session['_auth_user_id'] # Очистить все истекшие сессии python manage.py clearsessions # Посмотреть все активные сессии пользователя (если используете custom Session) from myapp.models import CustomSession user_sessions = CustomSession.objects.filter(user_id=42) for session in user_sessions:     print(f"Session {session.session_key}: expires {session.expire_date}")`

---

## Заключение

Django auth система использует **криптографически надёжный PBKDF2-HMAC-SHA256** для хеширования паролей с **870,000 итераций** по умолчанию, что делает перебор практически невозможным. Сессии хранятся в отдельной таблице `django_session` с **JSON сериализацией** и содержат только ID пользователя и путь к бэкенду аутентификации, а не сам пароль. Вся система **полностью кастомизируется** - от алгоритмов хеширования до структуры таблицы сессий.

1. [https://www.youtube.com/watch?v=Ukau9Ut9BSo](https://www.youtube.com/watch?v=Ukau9Ut9BSo)
2. [https://docs.djangoproject.com/en/5.2/topics/auth/passwords/](https://docs.djangoproject.com/en/5.2/topics/auth/passwords/)
3. [https://django-collin.readthedocs.io/en/latest/topics/auth/passwords.html](https://django-collin.readthedocs.io/en/latest/topics/auth/passwords.html)
4. [https://stackoverflow.com/questions/64883778/why-does-django-default-password-hashing-algorithm-use-216000-iterations](https://stackoverflow.com/questions/64883778/why-does-django-default-password-hashing-algorithm-use-216000-iterations)
5. [https://git.ivran.ru/mirrors/django/commit/a075e2ad0dcce65cb5cf4cb654ac8a6839db0baf?style=split&whitespace=ignore-change&show-outdated=](https://git.ivran.ru/mirrors/django/commit/a075e2ad0dcce65cb5cf4cb654ac8a6839db0baf?style=split&whitespace=ignore-change&show-outdated=)
6. [https://docs.djangoproject.com/en/6.0/topics/auth/passwords/](https://docs.djangoproject.com/en/6.0/topics/auth/passwords/)
7. [https://docs.djangoproject.com/en/5.0/_modules/django/contrib/auth/hashers/](https://docs.djangoproject.com/en/5.0/_modules/django/contrib/auth/hashers/)
8. [https://eli.thegreenplace.net/2011/07/17/django-sessions-part-iii-user-authentication](https://eli.thegreenplace.net/2011/07/17/django-sessions-part-iii-user-authentication)
9. [https://xxx-cook-book.gitbooks.io/django-cook-book/Auths/Password.html](https://xxx-cook-book.gitbooks.io/django-cook-book/Auths/Password.html)
10. [https://eli.thegreenplace.net/2011/06/29/django-sessions-part-ii-how-sessions-work](https://eli.thegreenplace.net/2011/06/29/django-sessions-part-ii-how-sessions-work)
11. [https://stackoverflow.com/questions/75925138/is-it-safe-to-add-user-id-column-to-django-db-session-model](https://stackoverflow.com/questions/75925138/is-it-safe-to-add-user-id-column-to-django-db-session-model)
12. [https://fossies.org/dox/django-5.2.7/namespacedjango_1_1contrib_1_1auth.html](https://fossies.org/dox/django-5.2.7/namespacedjango_1_1contrib_1_1auth.html)
13. [https://docs.djangoproject.com/en/5.2/topics/http/sessions/](https://docs.djangoproject.com/en/5.2/topics/http/sessions/)
14. [https://docs.djangoproject.com/en/6.0/topics/http/sessions/](https://docs.djangoproject.com/en/6.0/topics/http/sessions/)
15. [https://docs.djangoproject.com/en/1.8/_modules/django/contrib/sessions/middleware/](https://docs.djangoproject.com/en/1.8/_modules/django/contrib/sessions/middleware/)
16. [https://docs.djangoproject.com/en/5.2/topics/http/middleware/](https://docs.djangoproject.com/en/5.2/topics/http/middleware/)
17. [https://docs.djangoproject.com/fr/2.2/topics/auth/passwords/](https://docs.djangoproject.com/fr/2.2/topics/auth/passwords/)
18. [https://python.plainenglish.io/how-to-use-sessions-in-django-for-persistent-data-d397d5980309](https://python.plainenglish.io/how-to-use-sessions-in-django-for-persistent-data-d397d5980309)
19. [https://cjh5414.github.io/django-user-model-password-hashing/](https://cjh5414.github.io/django-user-model-password-hashing/)
20. [https://stackoverflow.com/questions/79300152/c-sharp-pbkdf2-password-hashing-with-django-compatible-format-pbkdf2-sha256-i](https://stackoverflow.com/questions/79300152/c-sharp-pbkdf2-password-hashing-with-django-compatible-format-pbkdf2-sha256-i)
21. [https://www.reddit.com/r/django/comments/104shkm/how_to_ensure_password_is_hashed_when_saving_user/](https://www.reddit.com/r/django/comments/104shkm/how_to_ensure_password_is_hashed_when_saving_user/)
22. [https://stackoverflow.com/questions/74478134/how-to-migrate-password-hashes-from-passlib-bcrypt-to-djangos-default-pbkdf2-sh](https://stackoverflow.com/questions/74478134/how-to-migrate-password-hashes-from-passlib-bcrypt-to-djangos-default-pbkdf2-sh)
23. [https://docs.djangoproject.com/en/6.0/topics/auth/customizing/](https://docs.djangoproject.com/en/6.0/topics/auth/customizing/)
24. [https://dev.to/doridoro/understanding-creating-an-authentication-backend-in-a-django-project-14po](https://dev.to/doridoro/understanding-creating-an-authentication-backend-in-a-django-project-14po)
25. [https://stackoverflow.com/questions/79371985/django-http-response-always-sets-sessionid-cookie-and-session-data-do-not-pers](https://stackoverflow.com/questions/79371985/django-http-response-always-sets-sessionid-cookie-and-session-data-do-not-pers)
26. [https://stackoverflow.com/questions/21514354/abstractuser-vs-abstractbaseuser-in-django](https://stackoverflow.com/questions/21514354/abstractuser-vs-abstractbaseuser-in-django)
27. [https://stackoverflow.com/questions/46272414/in-django-after-a-login-how-can-i-detect-which-auth-backend-authenticated-the-u](https://stackoverflow.com/questions/46272414/in-django-after-a-login-how-can-i-detect-which-auth-backend-authenticated-the-u)
28. [https://stackoverflow.com/questions/28696395/django-no-sessionid-in-cookie](https://stackoverflow.com/questions/28696395/django-no-sessionid-in-cookie)
29. [https://shecode3.hashnode.dev/using-abstractbaseuser-in-django](https://shecode3.hashnode.dev/using-abstractbaseuser-in-django)
30. [https://docs.djangoproject.com/en/5.2/topics/auth/customizing/](https://docs.djangoproject.com/en/5.2/topics/auth/customizing/)
31. [https://www.reddit.com/r/django/comments/t3k359/decoupled_django_stop_passing_sessionid_and_csrf/](https://www.reddit.com/r/django/comments/t3k359/decoupled_django_stop_passing_sessionid_and_csrf/)
32. [https://openclassrooms.com/en/courses/7107341-intermediate-django/7262933-customize-the-user-model](https://openclassrooms.com/en/courses/7107341-intermediate-django/7262933-customize-the-user-model)
33. [https://www.geeksforgeeks.org/python/python-sessions-framework-using-django/](https://www.geeksforgeeks.org/python/python-sessions-framework-using-django/)
34. [https://til.simonwillison.net/python/password-hashing-with-pbkdf2](https://til.simonwillison.net/python/password-hashing-with-pbkdf2)
35. [https://stackoverflow.com/questions/235950/how-to-lookup-django-session-for-a-particular-user](https://stackoverflow.com/questions/235950/how-to-lookup-django-session-for-a-particular-user)
36. [https://docs.djangoproject.com/fr/2.2/topics/http/sessions/](https://docs.djangoproject.com/fr/2.2/topics/http/sessions/)
37. [https://www.reddit.com/r/crypto/comments/jy5o01/reversing_password_and_salt_in_pbkdf2hmacsha256/](https://www.reddit.com/r/crypto/comments/jy5o01/reversing_password_and_salt_in_pbkdf2hmacsha256/)
38. [https://github.com/django/django/blob/main/django/contrib/sessions/models.py](https://github.com/django/django/blob/main/django/contrib/sessions/models.py)
39. [https://stackoverflow.com/questions/47339431/django-new-style-middleware-process-response](https://stackoverflow.com/questions/47339431/django-new-style-middleware-process-response)
40. [https://django.readthedocs.io/en/stable/genindex.html](https://django.readthedocs.io/en/stable/genindex.html)
41. [https://testdriven.io/blog/django-spa-auth/](https://testdriven.io/blog/django-spa-auth/)
42. [https://docs.djangoproject.com/en/6.0/ref/models/fields/](https://docs.djangoproject.com/en/6.0/ref/models/fields/)
43. [https://www.djbook.ru/rel1.9/topics/auth/customizing/](https://www.djbook.ru/rel1.9/topics/auth/customizing/)
44. [https://github.com/django-polymorphic/django-polymorphic/issues/366](https://github.com/django-polymorphic/django-polymorphic/issues/366)
45. [https://git.ivran.ru/mirrors/django/commit/a0878b5f95b5fc5bac02d818e864cab507b73564?files=django%2Fcontrib](https://git.ivran.ru/mirrors/django/commit/a0878b5f95b5fc5bac02d818e864cab507b73564?files=django%2Fcontrib)
46. [https://code.djangoproject.com/ticket/15627](https://code.djangoproject.com/ticket/15627)
47. [https://stackoverflow.com/questions/13978828/django-session-key-changing-upon-authentication](https://stackoverflow.com/questions/13978828/django-session-key-changing-upon-authentication)
48. [https://django.readthedocs.io/en/1.4.X/topics/http/sessions.html](https://django.readthedocs.io/en/1.4.X/topics/http/sessions.html)
49. [https://docs.djangoproject.com/en/6.0/topics/auth/default/](https://docs.djangoproject.com/en/6.0/topics/auth/default/)
50. [https://stackoverflow.com/questions/60810887/how-to-verify-passwords-created-by-djangomake-password-without-django](https://stackoverflow.com/questions/60810887/how-to-verify-passwords-created-by-djangomake-password-without-django)
51. [https://stackoverflow.com/questions/24076572/django-session-serializer-help-choosing-between-pickle-json](https://stackoverflow.com/questions/24076572/django-session-serializer-help-choosing-between-pickle-json)
52. [https://stackoverflow.com/questions/29192340/how-to-use-make-password-and-check-password-manually](https://stackoverflow.com/questions/29192340/how-to-use-make-password-and-check-password-manually)
53. [https://runebook.dev/ja/articles/django/ref/settings/std:setting-SESSION_SERIALIZER](https://runebook.dev/ja/articles/django/ref/settings/std:setting-SESSION_SERIALIZER)