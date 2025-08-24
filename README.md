##### _разработка Sayfullin R.R.

Это репозиторий кода для Django 4 by Example, написанный Антонио Меле. Он содержит все вспомогательные файлы проекта,
необходимые для работы с книгой от начала до конца.

Инструкция актуальна для Linux-систем.
========================================================================================================================
Используемые технологии:
    python = "^3.11"
    Django = "^4.1"
    PostgreSQL

Скопируйте репозиторий с помощью команды:
$ git clone https://github.com/RuslanSayfullin/django-by-example.git
Перейдите в основную директорию с помощью команды: 
$ cd django-5-by-example

Создать и активировать виртуальное окружение:
========================================================================================================================
$ poetry env use python3.13
Установить зависимости:
$ poetry install 
Сохранить, адрес созданного виртуального окружения из вывода(рекомендуется)
$ poetry env activate
$ source path_to_env/bin/activate
(djsite-py3.13) $
Выход:
$ exit

Добавить в директорию django-5-by-example/backend файл psw.py
========================================================================================================================
В данный файл, необходимо добавить две переменные:

secret_key = 'ваш секретный ключ'   # секретный ключ приложения
dbase_psw = 'ваш ключ к БД'         # Пароль для подключения к БДpoetry init

Создание БД
========================================================================================================================
$ sudo su - postgres
Теперь запускаем командную оболочку PostgreSQL:
$ psql 

=# CREATE DATABASE django_example;
=# CREATE USER portaluser WITH PASSWORD 'myPassword';
=# GRANT ALL PRIVILEGES ON DATABASE django_example TO portaluser;
=# \q
$ exit

В директорий django-4-by-example
========================================================================================================================
Для запуска выполнить следующие команды:
Команда для создания миграций приложения для базы данных
$ python3 manage.py makemigrations
$ python3 manage.py migrate

Создание суперпользователя
$ python3 manage.py createsuperuser

Будут выведены следующие выходные данные. Введите требуемое имя пользователя, электронную почту и пароль:
по умолчанию почта admin@admin.com пароль: 12345

Username (leave blank to use 'admin'): admin
Email address: admin@admin.com
Password: ********
Password (again): ********
Superuser created successfully.


Запуск программы.
========================================================================================================================
Откройте окно терминала и запустите все программы:
    Отправьте свое веб-приложение на сервер разработки Django в первом окне:
         python3 manage.py runserver
        $ python manage.py runserver_plus --cert-file cert.crt



Django-приложение будет доступно по адресу: http://127.0.0.1:8000/




===================================================================================

Тестирование Django-проекта с использованием pytest

Тестирование Django-приложений с pytest требует некоторой настройки, но предоставляет более гибкий и мощный инструментарий по сравнению со стандартным Django test runner. Правильно организовать процесс:
1. Установка необходимых пакетов

    Установите pytest и необходимые плагины:
        $ pip install pytest pytest-django pytest-cov pytest-mock pytest-xdist
        pytest-django - интеграция pytest с Django
        pytest-cov - проверка покрытия кода тестами
        pytest-mock - удобные моки
        pytest-xdist - параллельное выполнение тестов

2. Настройка pytest

    Создайте файл pytest.ini в корне проекта:
        [pytest]
        DJANGO_SETTINGS_MODULE = your_project.settings
        python_files = tests.py test_*.py *_tests.py
        addopts = --reuse-db

    Или setup.cfg:
        [tool:pytest]
        DJANGO_SETTINGS_MODULE = your_project.settings
        python_files = tests.py test_*.py *_tests.py
        addopts = --reuse-db

3. Структура тестов

    Рекомендуемая структура проекта:
        your_app/
            tests/
                __init__.py
                conftest.py
                factories.py
                test_models.py
                test_views.py
                test_forms.py
                test_apis.py

4. Фикстуры (Fixtures)
    Базовые фикстуры

        Создайте conftest.py для общих фикстур:

            import pytest
            from django.contrib.auth.models import User

            @pytest.fixture
            def user(db):
                return User.objects.create_user(username='testuser', password='12345')

            @pytest.fixture
            def client():
                from django.test import Client
                return Client()

            @pytest.fixture
            def auth_client(client, user):
                client.login(username='testuser', password='12345')
                return client

    Фикстуры для моделей

        Используйте factory_boy или pytest-factoryboy для создания тестовых данных:

            # factories.py
            import factory
            from myapp.models import MyModel

            class MyModelFactory(factory.django.DjangoModelFactory):
                class Meta:
                    model = MyModel
                
                name = factory.Sequence(lambda n: f"Object {n}")
                is_active = True

5. Написание тестов
    Тестирование моделей

        # test_models.py
        from myapp.models import MyModel
        from .factories import MyModelFactory

        def test_model_creation():
            obj = MyModelFactory()
            assert obj.pk is not None

        def test_model_str():
            obj = MyModelFactory(name="Test")
            assert str(obj) == "Test"

    Тестирование views

        # test_views.py
        from django.urls import reverse

        def test_home_page(client):
            url = reverse('home')
            response = client.get(url)
            assert response.status_code == 200

        def test_protected_view(auth_client):
            url = reverse('protected')
            response = auth_client.get(url)
            assert response.status_code == 200

    Тестирование форм

        # test_forms.py
        from myapp.forms import MyForm

        def test_valid_form():
            data = {'name': 'Test', 'email': 'test@example.com'}
            form = MyForm(data=data)
            assert form.is_valid()

        def test_invalid_form():
            data = {'name': '', 'email': 'invalid'}
            form = MyForm(data=data)
            assert not form.is_valid()

    Тестирование API (DRF)

        # test_apis.py
        from rest_framework.test import APIClient

        @pytest.fixture
        def api_client():
            return APIClient()

        def test_api_endpoint(api_client):
            url = reverse('api:items-list')
            response = api_client.get(url)
            assert response.status_code == 200

6. Тестирование асинхронного кода

    Для тестирования асинхронных представлений в Django 3.1+:

        import pytest
        from django.test import Client

        @pytest.mark.asyncio
        async def test_async_view():
            client = Client()
            response = await client.get('/async-url/')
            assert response.status_code == 200

7. Запуск тестов

Основные команды:

# Все тесты
pytest

# Конкретный файл
pytest myapp/tests/test_models.py

# Конкретный тест
pytest myapp/tests/test_models.py::test_model_creation

# С покрытием кода
pytest --cov=myapp

# Параллельно (используя все ядра)
pytest -n auto

8. Полезные советы

    Используйте маркеры:
    python

@pytest.mark.django_db
def test_with_db():
    pass

Параметризуйте тесты:
python

@pytest.mark.parametrize("input,expected", [
    ("3+5", 8),
    ("2+4", 6),
])
def test_eval(input, expected):
    assert eval(input) == expected

Используйте моки для внешних сервисов:
python

def test_external_api(mocker):
    mock_get = mocker.patch('requests.get')
    mock_get.return_value.status_code = 200
    # тестируем код, использующий requests.get

Тестируйте исключения:
python

    def test_raises_exception():
        with pytest.raises(ValueError):
            function_that_raises()

    Используйте pytest-django фикстуры:

        client - Django test client

        admin_client - аутентифицированный admin user

        rf - RequestFactory

        settings - временное изменение настроек

Правильно организованное тестирование с pytest сделает ваш Django-проект более надежным и упростит поддержку кода в будущем.