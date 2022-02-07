# Django Heroku S3 Template
An easy to start template currently at Django version 4.0.2 with Heroku and AWS S3 Static and Media file settings built-in.


## Split Settings Structure

Settings are stored now in a `settings/` directory in the config folder and split into 3 modules:
1. **base.py**: For base settings that are used both locally and in production.
2. **dev.py**: For settings only used in development (like `DEBUG = True`)
3. **prod.py**: For settings only used in production (like `DEBUG = False`) 

---
## Heroku Settings

```python
import django_on_heroku
import dj_database_url
from dotenv import load_dotenv, find_dotenv
from decouple import config

# In base.py

INSTALLED_APPS = [
    # ...
    'django_extensions',
    # ...
]

load_dotenv(find_dotenv())

DATABASES = {'default': dj_database_url.config(default='sqlite3:///db.sqlite', conn_max_age=600)}


# In prod.py

SECRET_KEY = config('SECRET_KEY')

DEBUG_PROPAGATE_EXCEPTIONS = True

LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'verbose': {
            'format' : "[%(asctime)s] %(levelname)s [%(name)s:%(lineno)s] %(message)s",
            'datefmt' : "%d/%b/%Y %H:%M:%S"
        },
        'simple': {
            'format': '%(levelname)s %(message)s'
        },
    },
    'handlers': {
        'console': {
            'level': 'DEBUG',
            'class': 'logging.StreamHandler',
        },
    },
    'loggers': {
        'MYAPP': {
            'handlers': ['console'],
            'level': 'DEBUG',
        },
    }
}

django_on_heroku.settings(locals(), staticfiles=False)
del DATABASES['default']['OPTIONS']['sslmode']
```

## AWS S3 Settings

```python
from decouple import config

# In base.py

INSTALLED_APPS = [
    # ...
    'storages',
    # ...
]

# In prod.py

AWS_ACCESS_KEY_ID = config('AWS_ACCESS_KEY_ID')

AWS_SECRET_ACCESS_KEY = config('AWS_SECRET_ACCESS_KEY')

AWS_STORAGE_BUCKET_NAME = config('AWS_STORAGE_BUCKET_NAME')

AWS_S3_CUSTOM_DOMAIN = f'{AWS_STORAGE_BUCKET_NAME}.s3.amazonaws.com'

AWS_DEFAULT_ACL = 'public-read'

AWS_S3_OBJECT_PARAMETERS = {'CacheControl': 'max-age=86400'}

AWS_LOCATION = 'static'

AWS_QUERYSTRING_AUTH = False

AWS_HEADERS = {'Access-Control-Allow-Origin': '*'}

DEFAULT_FILE_STORAGE = 'storages.backends.s3boto3.S3Boto3Storage'

STATICFILES_STORAGE = 'storages.backends.s3boto3.S3StaticStorage'

STATIC_URL = f'https://{AWS_S3_CUSTOM_DOMAIN}/static/'

MEDIA_URL = f'https://{AWS_S3_CUSTOM_DOMAIN}/media/'
```