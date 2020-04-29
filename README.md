# How to setup Django project for Heroku Prod Deployment?

## Setup the Django Project

Assuming python and pip is installed in your system<br>

- `pip install pipenv` Pipenv creates a seperate environment for all our dj projects<br>
- `mkdir <MY_DJANGO_PROJECT> && cd <MY_DJANGO_PROJECT>`
- `pipenv shell`
- `pipenv install django` creates a Pipfile and Pipfile.lock
- `django-admin startproject <my_dj_project>`
- `cd  <my_dj_project>`
- `python manage.py makemigrations`
- `python manage.py migrate`
- `python manage.py runserver [PORT]` PORT is optional

## Seperate the DEV and PROD environment

- `pipenv install django-environ`
- `touch .env` Create a dotenv file in <my_dj_project> and copy the SECRET_KEY from <my_dj_project>/settings.py
  ```bash
    #Sample .env file
    SECRET_KEY="=o)p$2&wd@%!x(@5^ujkb^g-di+t8zfk+h6g&6=notcy+7rq=$"
    DEBUG=True
  ```
- Make the following changes in <my_dj_project>/settings.py
  ```python
    import environ
    env = environ.Env()
    environ.Env.read_env()

    # SECURITY WARNING: keep the secret key used in production secret!
    SECRET_KEY = env.str('SECRET_KEY',default='=o)p$2&wd@%!x(@5^ujkb^g-di+t8zfk+h6g&6=notcy+7rq=$')

    # SECURITY WARNING: don't run with debug turned on in production!
    DEBUG = env.bool('DEBUG',default=False)
  ```

## 

