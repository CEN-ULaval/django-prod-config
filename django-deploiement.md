# Django: deploiement
### Etienne Godin 17/06/2021

Guide et référence: [https://www.youtube.com/watch?v=Sa_kQheCnds](https://www.youtube.com/watch?v=Sa_kQheCnds)

1. requirements.txt à la racine du projet. Un projet peut être structuré comme cela : [https://stackoverflow.com/questions/22841764/best-practice-for-django-project-working-directory-structure](https://stackoverflow.com/questions/22841764/best-practice-for-django-project-working-directory-structure)
2. copier le projet au bon endroit dans /var/www/html/?
3. Installer python divers:
```sh
$ pip install python3-pip, python3-venv
```
4. créer l’environnement virtuel python: 
```sh
$ python3 -m venv projet_path/venv
```
5. activer le projet: 
```sh
$ source venv/bin/activate
```

6. installer les dépendances: 
```sh
$ pip install -r requirements.txt
```

7. vérifier les paramètres du projet django projet/settings.py. STATIC_ROOT pour que les fichiers ‘static’ soient indexés par l’application
```python
ALLOWED_HOSTS = ['IP or hostname cryobase.cen.']
STATIC_ROOT = os.path.join(BASE_DIR, ‘static’)
```

8. Pour copier/récupérer le contenu de static: 
```sh
$ python manage.py collectstatic
```

9. Tester le serveur DEV voir si tout marche à date. Essayer dans le browser cryobase.cen…:8000 | le serveur est live pour tous, mais toujours en mode développement
```sh
$ python manage.py runserver 0.0.0.0:8000
``` 
10. Bonne pratique à ce moment: tester toutes les fonctionnalités avec le serveur de développement

11. Stopper le serveur de développement quand complété dans la console: CTRL+C

12. Web serveur de production: Apache2 avec libapache2-mod-wsgi-py3
```sh
$ sudo apt install apache2 libapache2-mod-wsgi-py3
```

13. Dans /etc/apache2/sites-available il y a des fichiers de configuration '.conf'. Il y a le défaut web:80 et le défaut web ssl:443 qui sont aussi dans sites-enabled. Il faut maintenant copier un des défaut vers ‘nom-application-django.conf' et éditer celui-ci si un nouveau port, si 443, éditer le 'ssl’. Personnaliser si requis:

```apache
Alias /static /chemin/de/application/django/static|media|autre
```

14. Sécurité pour ces alias:

```apache
<Directory /chemin/de/application/django/static|media|autre>
  Required all granted
</Directory>
```

15. Sécurité pour wsgi.py:

```apache
<Directory /chemin/de/application_django/application_django>
  <Files wsgi.py>
    Required all granted
  </Files>
</Directory>
```

16. Alias pour wsgi.py : référer à son chemin dans l’aborescence

```apache
WSGIScriptAlias / /chemin/de/application_django/application_django/wsgi.py>
```
17. Spécifier le daemon:

```apache
WSGIDaemonProcess nom_de_django_app(juste une etiquette) python-path=/chemin/de/application_django python-home=/chemin/ou/se/trouve/le/venv
```

18. Spécifier le groupe du daemon. On doit utiliser en référence le ‘nom_de_django_app spécifié au point '17’

```apache
WSGIProcessGroup nom_de_django_app
```

19. Rafraîchir apache2: (désactiver site 000 si nécessaire)

```sh
$ sudo a2ensite nom_du_fichier_choisi_sites-available
```

20. S’assurer qu’on a les crédentiels pour Postgresql dans settings.py

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'mydatabase',
        'USER': 'mydatabaseuser',
        'PASSWORD': 'mypassword',
        'HOST': '127.0.0.1',
        'PORT': '5432',
    }
}
```

21. Donner les permissions apache au projet Django pour changer le groupe seulement vers www-data (apache):

```sh
$ sudo chown :www-data django_project 
```

22. Appliquer chown/chmod tel que requis pour d’autres éléments dans l’aborescence de l’application

23. créer un fichier d’environnement avec les infos importantes dans /etc par exemple. Ex: django_app_config.json

```json
{
  "SECRET_KEY": "asecretkey",
  "EMAIL_USER": "anemail",
  "EMAIL_PASS": "apass"
}
```

24. Ajuster dans settings.py pour référer au json:

```python
import json

with open('/etc/config.json') as config_file:
  config = json.load(config_file)
  
SECRET_KEY = config['SECRET_KEY']

DEBUG = False
```

25. Désactiver port 8000 avec le firewall; s’assurer que ça marche sur 80 (ou celui qu’on a prévu)

26. Tester (restart apache)

