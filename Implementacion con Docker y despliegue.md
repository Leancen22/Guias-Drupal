#  Implementación con Docker y despliegue

En esta guía se verá como implementar Docker para el mantenimiento y despliegue de un sitio Drupal.

Veremos como generar las imágen de Docker para el Drupal base que levantará el sitio y luego como
generar la imágen para en caso de que lo tengamos, poder desarrollar sobre un perfil de instalación e integrarlo 
mediante un paquete de composer.

#  Importante

Esta guía partirá desde un punto donde tengamos ya un sitio de Drupal instalado en nuestro equipo, 
y en caso de necesitar un perfil de instalación, también esté desarrollado e implementado.

En caso de necesitar ver como realizar esta instalación consultar el resto del material.

No es necesario para poder seguir la guía pero es util tener conocimientos básicos de como funciona Docker, git,
runners, drush, composer y ddev (este último para desarrollo).

Este desarrollo se hara utilizando la plataforma GitLab, la cual tiene sus diferencias con Github a la hora de trabajar
con CI/CD.

En caso de estar trabajando con Portal Express o derivados puede consultar directamente la guia correspondiente a Docker para portal express.

#  Paso a paso
# 1) Estructura del proyecto

 Vamos primero a conocer la estructura del proyecto en el que implementaremos Docker, para este proyecto como se menciono
 se utilizara un perfil de instalacion donde se desarrollara, en caso de no contar con un perfil de instalacion para el
 desarrollo y desarrollar sobre la misma instalacion de drupal, puede ignorar los pasos relacionados al perfil.
    
 Empecemos con la instalacion de drupal, esta puede ser simplemente una instalacion normal de drupal, utilizando por ejemplo
     
     composer require drupal/recommended-project
     
Para este desarrollo usaremos ddev, por lo que en caso de no usarlo se deben adaptar los pasos.

Empezaremos configurando nuestro entorno de trabajo

     mkdir ProjectoConDocker
     cd ProjectoConDocker
     ddev config --project-type=drupal9 --docroot=web --create-docroot
     ddev start
     ddev ssh
     composer require drupal/recommended-project ProjectoConDocker
     composer require drush/drush
     composer install
     
Con esto ya tendremos una instalacion de Drupal lista para usar, tendremos la siguiente estructura de carpetas:

     - .ddev (carpeta de configuracion de ddev)
     - web (carpeta donde se encuentra nuestro drupal (core))
     - composer.json (archivo de configuracion de modulos y versiones)
     - composer.lock (archivo de requerimientos para los modulos y configuraciones del .json)
     - vendor (carpeta necesaria para el funcionamiento de drupal con drush)
     
En caso de usar un perfil de instalacion este se alojara en /web/profiles/contrib/NOMBRE_DEL_PERFIL

Se puede consultar la documentacion de este mismo repositorio donde se explica como crear un perfil de instalacion o distribucion.
    
# 2) Repositorios de trabajo

Para este desarrollo es importante que a la hora de crear el repositorio este sea publico y tengamos configurada
la posibilidad de implementar runners.

Para trabajar usaremos GitLab, donde crearemos un repositorio con el nombre que queramos para alojar el projecto,
una vez creado se nos mostraran varias maneras de enlazar el repositorio creado con nuestro proyecto, vamos a usar una de estas maneras.

En la carpeta que creamos anteriormente (ProjectoConDocker) usaremos el siguiente comando:
   
    git init
    
Esto nos creara una instancia de git (carpeta .git) que nos permitira enlazar a un repositorio remoto.
Ahora toca decirle a donde debe mandar los cambios, esta sera la conexion con el repositorio, para esto
nos vamos a nuestro repositorio creado y compiamos la ruta de clonado del mismo, puede ser por HTTP o SSH, por 
simplicidad se puede usar HTTP, pero nosotros usaremos SSH

Supongamos tenemos la siguiente ruta:

    https://gitlab.isaltda.com.uy/portal/projectocondocker.git
    
Ahora solo deberiamos hacer lo siguiente (dentro de ProjectoConDocker):

    git remote add origin git@gitlab.isaltda.com.uy:portal/projectocondocker.git
    
En caso de usar HTTP:

    git remote add origin https://gitlab.isaltda.com.uy/portal/projectocondocker.git

A partir de ahora podremos subir a este repositorio creado nuestros cambios:
   
    git add .
    git commit -m "Primer commit del projecto"
    git push -u origin master
    
_IMPORTANTE_
No todo se debe subir al repositorio, Github por ejemplo avisa cuando credenciales o informacion sensible es subida, GitLab no
lo hace, por lo que tenemos que proibir la subida de esta informacion, ademas de esto, tampoco deben subirse archivos o carpetas que
sean generadas por drupal automaticamente, por ejemplo modulos, el core, etc.

Tambien en caso de querer desarrollar en un perfil aparte, no se debe subir junto con la instalacion.

Para todo esto se crea una archivo **.gitignore** en la raiz del proyecto, que como su nombre indica, git ignorara todo lo que haya en este archivo a la hora
de subir al repositorio, para un drupal normal, instalado como se indico mas arriba podriamos usar el siguiente contenido:

```
   # Ignora directorios creados por composer
   /drush/contrib/
   /vendor/
   /web/core/
   /web/modules/contrib/
   /web/themes/contrib/
   /web/profiles/contrib/
   /web/libraries/

   # Ignora informacion sensible
   #/web/sites/*/settings.php # Voy a probar con variables de entorno
   /web/sites/*/settings.local.php

   # Ignora los directorios donde Drupal guarda sus archivos
   /web/sites/*/files/

   # En caso de desarrollar con PHPStorm
   /.idea/
   .DS_Store

   # Ignora variables de entorno y archivos de configuracion de ddev
   /.ddev
   /.env
   /.editorconfig
   /.gitattributes
   /local_data/

   #Ignora configuracion SSH de gitlab (aqui se guarda el personal access token)
   auth.json

   #En caso de tener backups de base de datos
   backups

```

Para el perfil de instalacion se pueden seguir los mismos pasos, el git init se realizaria dentro de la carpeta 
/web/profiles/contrib/NOMBRE_DEL_PERFIL y se procederia como ya vimos, creando su repositorio respectivo y enlazandolo.
El .gitignore ya dependera de la informacion del perfil.

# 3) Runners

# 4) Implementacion de CI/CD para la instalacion de Drupal

Vamos a implementar CI/CD, lo que corresponde a integracion y desarrollo continuo.
Por defecto, el archivo que GitLab detecta para CI/CD es el archivo **.gitlab-ci.yml**, podemos ver la configuracion
de CI/CD de un proyecto en https://gitlab.EXTENSION.com.uy/NOMBRE_GRUPO/NOMBRE_PROYECTO/-/settings/ci_cd y modificar a nuestro gusto,
ahi tambien se encuentra la configuracion de runners y variables de entorno.

Se puede ver tambien la documentacion oficial de GitLab https://docs.gitlab.com/ee/ci/yaml/

Veamos la configuracion del archivo .gitlab-ci.yml para la instalacion de Drupal.

```
    stages:
      - build-image
      
    variables:
      GIT_SUBMODULE_STRATEGY: none

    docker-image:
      image: docker:20-git
      services:
        - docker:20-dind
      stage: build-image
      script:
        - ./.gitlab/build-image.sh
      only:
        - /^[0-9]+\.[0-9]+\.[0-9]+$/
      tags:
        - docker2
```

Vamos a ir explicando cada linea y luego veremos el contenido de ./.gitlab/build-image.sh que es el encargado de 
generar la imagen.

```
    stages:
      - build-image
```
Los stages corresponden a los pasos que seguira el archivo a la hora de ser leido, es el orden de las 
instrucciones, supongamos tenemos lo siguiente:

```
    stages:
      - Paso 1
      - Paso 2
      - Paso 3
```
Entonces, los scripts se ejecutaran en el orden que se vayan llamando, por ejemplo:

```
    stages:
      - Paso-1
      - Paso-2
      - Paso-3
    
    esto-corresponde-al-paso-1:
      image: docker:20-git
      services:
        - docker:20-dind
      stage: Paso-1
      script:
        - echo "Esta es la ejecucion del paso 1"
      tags:
        - runner_ejemplo
        
    esto-corresponde-al-paso-3:
      image: docker:20-git
      services:
        - docker:20-dind
      stage: Paso-3
      script:
        - echo "Esta es la ejecucion del paso 3, no estamos ejecutando el paso 2"
      tags:
        - runner_ejemplo
```

Se pueden ejecutar en ordenes comandos para poder en diferentes etapas por ejemplo, instalar requerimientos, luego
sobre lo anterior instalar modulos, etc.
Supongamos el ejemplo anterior, donde tenemos tags asociados al runner runner_ejemplo (runner que responde a esa etiqueta),
si subimos este archivo al repositorio podremos ver que se procesaran estas instrucciones por consola (https://gitlab.EXTENSION.com.uy/NOMBRE_GRUPO/NOMBRE_PROYECTO/-/jobs/ID_DEL_TRABAJO) los echo correspondientes a cada stage.

------------------------------------------------------------------------------------------------------------------------------

```
   variables:
         GIT_SUBMODULE_STRATEGY: none
```

En este caso no tenemos configurado un archivo **.gitmodules** para poder incluir subrepositorios a nuestro repositorio,
por lo que colocamos esta variable como none.

------------------------------------------------------------------------------------------------------------------------------
```
   docker-image:
      image: docker:20-git
      services:
        - docker:20-dind
      stage: build-image
      script:
        - ./.gitlab/build-image.sh
      only:
        - /^[0-9]+\.[0-9]+\.[0-9]+$/
      tags:
        - docker2
```

Como vimos anteriormente esta instruccion (docker-image), se ejecutara en el stage build-image, que en este caso es el unico
que tenemos.
Podemos ver que se usa una imagen base y un servicio:
```
  ...
  image: docker:20-git
  services:
    - docker:20-dind
  ...
```

Esto nos dice que imagen de docker usara para crear la imagen base, en este caso docker:20-git y el servicio que creara
todo esto es docker:20-dind, donde dind corresponde a las siglas de docker-in-docker.

Por ejemplo un servicio e imagen que se puede usar en caso de requerir de una consola de linux puede ser alpine.

```
   script:
     - ./.gitlab/build-image.sh
```
Esto como vimos en el ejemplo de stages, aqui se escriben las lineas de scripts que se ejecutaran, por comodidad, se esta usando un archivo .sh para
poder escribir el script, lo veremos al final.


```
   only:
     - /^[0-9]+\.[0-9]+\.[0-9]+$/
```
En este caso permitimos que se ejecute para cualquier entrada posible, podriamos solo fijarla a una rama (master por ejemplo) o a un merge entre
ramas, no es el caso.

```
   tags:
     - docker2
```
Esto corresponde al runner, el cual es necesario como vimos anteriormente.

------------------------------------------------------------------------------------------------------------------------------

Ahora si veremos lo que crea la imagen de docker, el archivo build-image.sh

# 5) Drupal y PHP para la imagen de la instalacion

# 6) Dockerfile y gitlab-ci para la instalacion de Drupal

# 7) Implementacion de CI/CD para el perfil de instalacion

# 8) Implementar generacion de paquete composer

# 9) Dockerfile y gitlab-ci para el perfil de instalacion

# 10) Subir los cambios y generar las imagenes (perfil e instalacion)

# 11) Despliegue usando la imagen

-------------------------------------------------------------------------------------------------------------

# Posibles errores

# Pasos previos necesarios para el despliegue
El despliegue de la imagen que generamos con los pasos anteriores requiere tener previamente en donde
se vaya a instalar algunos preparativos, pasaremos a ver la configuracion necesaria.
...
    
# Desarrollo y escalabilidad
En caso de tener el perfil, el desarrollo se hara sobre ese repositorio en concreto, y posteriormente
se actualizara el composer de la instalacion para generar la imagen actualizada, vamos a repasar esto y
la metodologia a seguir a la hora de querer escalar a otros proyectos.
...
