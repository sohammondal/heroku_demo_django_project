# How to setup Django project for Heroku Prod Deployment?

## Setup the Django Project

Assuming python and pip are alreay installed in your system<br>

- `pip install pipenv` Pipenv creates a seperate environment for all our dj projects<br>
- `mkdir <MY_DJANGO_PROJECT> && cd <MY_DJANGO_PROJECT>`
- `pipenv shell`
- `pipenv install django` creates a Pipfile and Pipfile.lock
- `django-admin startproject <my_dj_project>`
- `cd  <my_dj_project>`
- `python manage.py makemigrations`
- `python manage.py migrate`
- `python manage.py runserver [PORT]` PORT is optional

## Setup the DEV and PROD environment

- `pipenv install django-environ`
- `touch .env` Create a dotenv file in <my_dj_project> and copy the SECRET_KEY from <my_dj_project>/settings.py
  
  `.env`
  ```bash
    #Sample .env file
    SECRET_KEY="=o)p$2&wd@%!x(@5^ujkb^g-di+t8zfk+h6g&6=notcy+7rq=$"
    DEBUG=True
  ```
- Make the following changes to <my_dj_project>/settings.py

  `settings.py`
  ```python
    import environ
    env = environ.Env()
    environ.Env.read_env() # reading .env file

    # SECURITY WARNING: keep the secret key used in production secret!
    SECRET_KEY = env.str('SECRET_KEY',default='=o)p$2&wd@%!x(@5^ujkb^g-di+t8zfk+h6g&6=notcy+7rq=$')

    # SECURITY WARNING: don't run with debug turned on in production!
    DEBUG = env.bool('DEBUG',default=False) 
  ```
- Make sure to add the `.env` file in `.gitignore`

## Setup for Heroku Deployment

#### PART 1 - Configuring the basics
- Make sure you are currently in <MY_DJANGO_PROJECT>
- `pipenv install gunicorn django_heroku`
- `touch Procfile` Heroku web applications require a `Procfile`
  
  `Procfile`
  ```txt
    web: gunicorn <my_dj_project>.wsgi
  ```
- Make the following changes to <my_dj_project>/settings.py
  
  `settings.py`
  ```python
    import django_heroku # at the top of settings.py
    ....
    ....
    ....
    # Activate Django-Heroku.
    django_heroku.settings(locals())  # at the bottom of settings.py
  ```

### PART 2 - Configuring the static assets
- `pip install whitenoise` Django does not support serving static files in production. However, the fantastic WhiteNoise project can integrate into your Django application, and was designed with exactly this purpose in mind.
- Make the following changes to <my_dj_project>/`settings.py`. Make sure to follow the instructions in comments -

  `settings.py`
  ```python
    
    # add WhiteNoise to the MIDDLEWARE_CLASSES list, above all 
    # other middleware apart from Django’s SecurityMiddleware

    MIDDLEWARE = [
      # 'django.middleware.security.SecurityMiddleware',
      'whitenoise.middleware.WhiteNoiseMiddleware',
      # ...
    ]
    ....
    ....
    ....
    ....

    BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
    
    ## Add the following static file configurations
    # Static files (CSS, JavaScript, Images)
    # https://docs.djangoproject.com/en/1.9/howto/static-files/
    STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
    STATIC_URL = '/static/'

    # Extra places for collectstatic to find static files.
    STATICFILES_DIRS = (
        os.path.join(BASE_DIR, 'static'),
    )
    
    STATICFILES_STORAGE = 'whitenoise.storage.CompressedManifestStaticFilesStorage'
  ```
- You can have a look at the entire [settings.py](https://github.com/sohammondal/heroku_demo_django_project/blob/master/heroku_demo_django/settings.py).
  
- This is important. Deployment in Heroku will fail w/o this
  - `mkdir static` Create a static folder in <MY_DJANGO_PROJECT>.
  - `touch static/.keep` A .keep file so that Git commits this folder as well
  - run `python manage.py collectstatic --noinput` to see if the command ran successfully otherwise fix errors.

- stop the server and re-run it `python manage.py runserver [PORT]` to make sure the server is running smoothly

### PART 3 - Setup the Heroku environment

Assuming you have heroku account & heroku cli installed in your system
- `heroku login -i` login to your heroku account
- `pip freeze > requirements.txt` in <MY_DJANGO_PROJECT> dir
- Create the Heroku app
  - `heroku create <my-heroku-app-name>` in <MY_DJANGO_PROJECT> dir
  - `git remote -v` to confirm that a remote named heroku has been set for your app
    ```bash
      $ git remote -v
        heroku  https://git.heroku.com/<my-heroku-app-name>.git (fetch)
        heroku  https://git.heroku.com/<my-heroku-app-name>.git (push)
    ```
- For an existing Heroku app, set the git remote of your poject to the app git
    ```bash
      $ heroku git:remote -a <my-heroku-app-name>
        set git remote heroku to https://git.heroku.com/<my-heroku-app-name>.git
    ```
- Set Heroku env varibales
  
  ```bash
     $ heroku config:set SECRET_KEY="=o)p$2&wd@%!x(@5^ujkb^g-di+t8zfk+h6g&6=notcy+7rq=$"
     $ heroku config:set DEBUG=True # turn this to false later
  ```

### PART 4 - Deploy the app
- Checklist before deploying - 
  - `Procfile`
  - `requirements.txt`
  - `Pipfile` & `Pipfile.lock`
  - `static` dir
  - all configurations committed
- `git push heroku master` push the changes
- `heroku logs -t` for viewing logs

### Part 5 - Post build tasks
- `heroku run bash` in <MY_DJANGO_PROJECT> dir to access the app shell environment on Heroku
- `python manage.py makemigrations`
- `python manage.py migrate`
- And you are good to go 😃 👍⚡
