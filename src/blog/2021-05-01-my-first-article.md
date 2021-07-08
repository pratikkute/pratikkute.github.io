---
title: One click deploy Django website with apache server on ubuntu
author: Pratik Kute
date: 2021-05-01
image: /assets/blog/article-1.jpg
tags: ["post", "featured"]
imageAlt: Django deployment
description: One click deploy Django website with apache server on ubuntu ...
---


# One click deploy Django website with apache server on ubuntu ...

## Get all files here [Github repo](https://github.com/pratikkute/deploy_new_websiite).


Replace the 'COMPANY' with your project from all files from repo!
Copy files to server home
Run the following command

``` bash
sudo chmod u+x deploy_website.sh
./deploy_website.sh
```

## Update settings file
Add ip and domain name to allowed host
```py
ALLOWED_HOSTS = ["*"]
```

Add "home" to installed app in settings
```py
INSTALLED_APPS = ['home']
```

### Add template DIRS
```py
'DIRS': [os.path.join(BASE_DIR, 'templates')],
```
### Email
```py
EMAIL_HOST = "smtp.gmail.com"
EMAIL_HOST_USER = ""
EMAIL_HOST_PASSWORD = ""
EMAIL_PORT = 587
EMAIL_USE_TLS = True
```

### Internationalization
### https://docs.djangoproject.com/en/2.2/topics/i18n/
```py
LANGUAGE_CODE = 'en-us'
TIME_ZONE = 'Asia/Kolkata'
USE_I18N = True
USE_L10N = True
USE_TZ = True
```

### Static files (CSS, JavaScript, Images)
### https://docs.djangoproject.com/en/2.2/howto/static-files/

```py
STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(BASE_DIR, 'static')

MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
```