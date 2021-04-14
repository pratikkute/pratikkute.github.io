# One click deploy Django website

replace the name of company  
copy files to server home  
run the following command  
sudo chmod u+x deploy_website.sh

then run
````sh
./deploy_website.sh
````
## Update settings file
add ip and domain name to allowed host  
```
ALLOWED_HOSTS = ["*"]
```
add "home" to installed app in settings  
````
INSTALLED_APPS = ['home']
````



### Add template DIRS
````
'DIRS': [os.path.join(BASE_DIR, 'templates')],
````
### Email 
````
EMAIL_HOST = "smtp.gmail.com"
EMAIL_HOST_USER = ""
EMAIL_HOST_PASSWORD = ""
EMAIL_PORT = 587
EMAIL_USE_TLS = True
````

### Internationalization
### https://docs.djangoproject.com/en/2.2/topics/i18n/
````
LANGUAGE_CODE = 'en-us'
TIME_ZONE = 'Asia/Kolkata'
USE_I18N = True
USE_L10N = True
USE_TZ = True
````

### Static files (CSS, JavaScript, Images)
### https://docs.djangoproject.com/en/2.2/howto/static-files/

````
STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(BASE_DIR, 'static')

MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
````