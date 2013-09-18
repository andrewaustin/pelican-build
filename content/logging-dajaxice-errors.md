Title: Logging Dajaxice Errors
Date: 2012-09-23
Tags: python, django

One of the problems when using Dajaxice in Django is that obtaining error
 messages is not well documented. Luckily, Dajaxice makes use of Python’s
 logging module. This means you can modify your settings.py to include a
 log handler for Dajaxice.

In your django settings module, look for the LOGGING directive. It should
 look similar to the following block of code:

    :::python
    LOGGING = {
        'version': 1,
        'disable_existing_loggers': False,
        'handlers': {
            'mail_admins': {
                'level': 'ERROR',
                'class': 'django.utils.log.AdminEmailHandler'
            }
        },
        'loggers': {
            'django.request': {
                'handlers': ['mail_admins'],
                'level': 'ERROR',
                'propagate': True,
            },
        }
    }

By default, Django tries to mail errors sent to the django.requests logger to
 the admin email(s) by using the error handler mail_admins, which is provided
 by the django.utils.log.AdminEmailHandler class.

Instead of using the django.requests logger, Dajaxice sends its error to the
 dajaxice logger. To capture the errors sent to this log, we must define the
 logger and set up a handler to handle these errors.

Because I use Heroku and like to see errors in real time with the command
 heroku logs -tail, I opted to set up an additional handler to log output
 to the console. The following snippet, which goes in the handlers dictionary
 listed above, defines a handler called console that outputs anything sent to it:

    :::python
    'console': {
        'level': 'DEBUG',
        'class': 'logging.StreamHandler'
    }

Now that a handler is defined, it’s a few short lines to set up the logger
 using this handler. The following logger, named dajaxice, responds to log
 level WARNING or higher:

    :::python
    'dajaxice': {
        'handlers': ['console'],
        'level': 'WARNING',
        'propagate': False,
    },

Now when Dajaxice spits out an error, it will be outputted to the console. See
 the full code below:

    :::python
    LOGGING = {
        'version': 1,
        'disable_existing_loggers': False,
        'handlers': {
            'mail_admins': {
                'level': 'ERROR',
                'class': 'django.utils.log.AdminEmailHandler'
            },
            'console': {
                'level': 'DEBUG',
                'class': 'logging.StreamHandler'
            }
        },
        'loggers': {
            'django.request': {
                'handlers': ['mail_admins'],
                'level': 'ERROR',
                'propagate': True,
            },
            'dajaxice': {
                'handlers': ['console'],
                'level': 'WARNING',
                'propagate': False,
            },
        }
    }
