## Setup

* Mostrar información de ayuda del Makefile

```
$ make

setup                Setup docker volume dirs
prune                Drop persistent data dirs
up                   Launch local stack
down                 Stop local stack
bootstrap-drupal     Create a new drupal codebase
bootstrap-civi       Add civi requirements to drupal codebase
install-drupal-civi  Install drupal and civi-crm
mysql-dump           Mysql dump
```

* Crear stack y código base drupal + civi en directorio src

```
docker-compose build
make setup
make bootstrap-drupal
make bootstrap-civi
```

* Instalar drupal + civi

```
make up
make install-drupal-civi
```

nota: Las credenciales del usuario admin apareceran en pantalla

 > [success] Installation complete.  User name: admin  User password: XXXXXXXXXX

 Al acceder al portal como admin debe aparecer la pestaña de divi-crm