# ansible-django-deploy

Ansible-плейбук, который провижинит Django + Gunicorn + Nginx + PostgreSQL
на чистый Ubuntu-сервер одной командой.

## Что разворачивает

- Python 3.10 (через deadsnakes PPA) + virtualenv
- Django + Django REST Framework + Gunicorn
- PostgreSQL: создаёт БД и пользователя
- Nginx: reverse proxy с проксированием на Gunicorn
- systemd-сервис для Gunicorn (автозапуск)
- healthcheck-эндпоинт для проверки состояния стека

## Структура

├── files/
│   └── my_healthcheck/   # Django-приложение с healthcheck-эндпоинтом
│       ├── views.py
│       └── urls.py
├── templates/
│   ├── nginx.conf.j2     # Nginx reverse proxy конфиг
│   └── gunicorn.service.j2  # systemd unit
├── inventory.ini         # Целевой сервер (заполнить перед запуском)
└── site.yml              # Основной плейбук

## Быстрый старт

**1. Заполни inventory.ini**

```ini
[app_servers]
your_server_ip ansible_port=22 ansible_user=root ansible_password=your_password ansible_python_interpreter=/usr/bin/python3

[app_servers:vars]
vault_db_password=your_secure_password
```

> В продакшене пароль лучше передавать через ansible-vault:
> `ansible-vault encrypt_string 'your_password' --name 'vault_db_password'`

**2. Запусти плейбук**

```bash
ansible-playbook -i inventory.ini site.yml
```

**3. Проверь результат**

```bash
curl http://your_server_ip/app_healthcheck/
```

Ожидаемый ответ:

```json
{
    "status": "healthy",
    "components": {
        "postgresql": "up",
        "nginx": "up",
        "django": "up"
    }
}
```

## Примечания

- `pg_hba.conf` переводится в режим `trust` для локального подключения postgres —
  учебный шорткат, в продакшене использовать `md5` или `scram-sha-256`.
- Плейбук идемпотентен по большинству задач (`creates:`, `state: present`).
