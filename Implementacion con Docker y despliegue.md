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

Esta guia se hara para Drupal 9 y PHP 8.x

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

Veamos la configuracion del archivo .gitlab-ci.yml para la instalacion de Drupal, este archivo lo que hara sera prepararnos todo
para cuando integremos el Dockerfile.

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

Ahora si veremos lo que prepara el escenario para la imagen de docker, el archivo build-image.sh:

Este archivo prepara la generacion de las imagenes basandose en el tag donde se pushean los commits, veremos esto en el analisis.

Las variables de entorno que se ven en el codigo son globales, puede verse mas de ellas en el siguiente link:
https://docs.gitlab.com/ee/ci/variables/predefined_variables.html

Si queremos ver que contiene cada etiqueta podemos ejecutar el .gitlab-ci.yml haciendo un echo de cada una.

```
   docker login ${CI_REGISTRY} -u ${CI_REGISTRY_USER} -p $CI_JOB_TOKEN

   VERSION=${CI_COMMIT_REF_NAME}
   TAG="latest"

   if [[ $CI_COMMIT_REF_NAME =~ ^[0-9]+\.[0-9]+\.[0-9]+$ ]]; then
     VERSION="stable"
     TAG=${CI_COMMIT_REF_NAME}
   fi

   IMAGE_TAG="${CI_REGISTRY_IMAGE}/${VERSION}:${TAG}"

   echo "========>> Build image <<=================="
   docker build -t "${IMAGE_TAG}" \
   --build-arg GITLAB_TOKEN=${PERSONAL_ACCESS_TOKEN} \
   --build-arg GITLAB_DOMAIN=${GITLAB_DOMAIN} . || { echo "Build image filed" ; exit 1; }


   echo "========>> Push image <<=================="
   docker push "${IMAGE_TAG}" || { echo "Push image filed" ; exit 1; }


   if [ "${VERSION}" = "stable" ]; then
     echo "========>> Push latest image <<=================="
     IMAGE_TAG_LATEST="${CI_REGISTRY_IMAGE}/${VERSION}:latest"
     docker tag ${IMAGE_TAG} ${IMAGE_TAG_LATEST}
     docker push "${IMAGE_TAG_LATEST}"
   fi
```
Analicemos como hicimos antes linea por linea.

    docker login ${CI_REGISTRY} -u ${CI_REGISTRY_USER} -p $CI_JOB_TOKEN

Esto inicia sesion en un registro de Docker para GitLab, en Github se realiza de forma similar. 

**${CI_REGISTRY}** Es la direccion del registro de contenedores de GitLab, es importante previamente ver
si tenemos esto activo
**${CI_REGISTRY_USER}** Es el nombre del usuario que tiene habilitado enviar los contenedores, en este caso
es necesario previamente ver que nuestro proyecto tenga habilitada esta posibilidad.
**$CI_JOB_TOKEN** Esto representa lo mismo que **$CI_REGISTRY_PASSWORD** lo cual es la credencial de acceso.

------------------------------------------------------------------------------------------------------------------------------

```
   VERSION=${CI_COMMIT_REF_NAME}
   TAG="latest"

   if [[ $CI_COMMIT_REF_NAME =~ ^[0-9]+\.[0-9]+\.[0-9]+$ ]]; then
     VERSION="stable"
     TAG=${CI_COMMIT_REF_NAME}
   fi

   IMAGE_TAG="${CI_REGISTRY_IMAGE}/${VERSION}:${TAG}"
```

Esta parte del codigo es la que toma la etiqueta con la que subiremos nuestros cambios al repositorio y 
prepara la direccion donde se construira y pusheara con Docker, le dice a Docker donde guardar la imagen que creara.

**${CI_COMMIT_REF_NAME}** Nombre de la rama o etiqueta que se detecta, veamos un ejemplo subiendo algo al repositorio:
   
    git add .
    git commit -m "Ejemplo para ver los tags"
    git push -u origin master
    
    git tag 1.0.0 ---> Este es el numero del tag, es incremental y unico.
    git push origin 1.0.0  ---> Esto pushea a ese tag el ultimo push realizado a una rama.
    
En este punto la variable global tendra por valor 1.0.0.

    TAG="latest"

Esto simplemente es una declaracion de variable, es una forma de decirle a GitLab la version que debe tomar, en este caso
se setea que sea "latest".

    if [[ $CI_COMMIT_REF_NAME =~ ^[0-9]+\.[0-9]+\.[0-9]+$ ]]; then
      VERSION="stable"
      TAG=${CI_COMMIT_REF_NAME}
    fi

Este if tiene una comparacion utilizando el operador de expresiones =~, que devuelve 0 o 1 dependiendo si el lado izquierdo coincide con
la expresion del lado derecho entra al if, en este caso la expresion corresponde a secuencia exclusivamente numericas, correspondientes a un 
tag numerico y no a una rama, por lo que al entrar define el tag con el ultimo valor seteado por nosotros al crearlo, en el ejemplo 1.0.0.
Se define por conveniencia la variable version en stable, lo que se usa para definir una version en correcto funcionamiento.


    IMAGE_TAG="${CI_REGISTRY_IMAGE}/${VERSION}:${TAG}"

Con las variables creadas definimos una nueva variable, que sera usada para definir la ruta del contenedor.
Un ejemplo seria el siguiente, para valor de IMAGE_TAG:
     
     gitlab.EXTENSION.com.uy:5005/NOMBRE_GRUPO/NOMBRE_PROYECTO/stable:1.0.0

Esto se puede ver en https://gitlab.EXTENSION.com.uy/NOMBRE_GRUPO/NOMBRE_PROYECTO/container_registry una vez que generemos
el tag y se ejecute correctamente el .gitlab-ci.yml.

El resto del archivo no se explicara, ya que son comandos de Docker que permiten crear la correspondiente imagen y luego
pushearla a la localizacion dada por IMAGE_TAG, veremos que se guarda ahi cuando estudiemos el Dockerfile

# 5) Dockerfile para la instalacion de Drupal

Ahora veremos el archivo Dockerfile, el cual es la informacion que se guardara en la imagen, son las instrucciones que Docker
ejecutara cuando este ejecutando el comando Docker build.

Crearemos un archivo llamado **Dockerfile** en la raiz de nuestro proyecto, hasta el momento deberia versenos asi:

```
     - .ddev
     - web
     - composer.json
     - composer.lock
     - vendor
     - .gitlab-ci.yml
     - .gitignore
     - Dockerfile
     - build-image.sh (Este en la carpeta que indica el .gitlab-ci.yml o se puede cambiar la direccion del script)
```

El archivo Dockerfile contendra lo siguiente:

```
   FROM gitlab.isaltda.com.uy:5005/portal/infra-express/drupal-php8:v1.14

   ARG GITLAB_TOKEN
   ARG GITLAB_DOMAIN

   COPY composer.*json ./
   COPY composer.*lock ./

   RUN apk update php-ldap

   RUN git config --global http.sslverify false && \
       composer config --global gitlab-domains ${GITLAB_DOMAIN} && \
       composer config --global gitlab-token.${GITLAB_DOMAIN} ${GITLAB_TOKEN} && \
       composer update && \
       composer global require drush/drush -W && \ 
       sed -e 's/;extension=ldap/extension=ldap/' -i /etc/php81/php.ini

   COPY . ./
```

Este es el archivo **mas importante** ya que en el se guarda la imagen del drupal que se desplegara, por lo
que es importante agregar todo lo necesario para su funcionamiento.

Veamos linea por linea como veniamos haciendo:

    FROM gitlab.isaltda.com.uy:5005/portal/infra-express/drupal-php8:v1.14
    
Esto lo analizaremos mas en profundidad en el proximo paso, pero esta es la direccion de donde se obtiene
la base de drupal para la instalacion, es un drupal 9 vacio con php 8.1, es totalmente necesario ya que sobre esta instalacion vacia
instalaremos todos los modulos y dependencias que desarrollamos y posteriormente el perfil de instalacion si lo tenemos.

```
   ARG GITLAB_TOKEN
   ARG GITLAB_DOMAIN
```

Se solicitan variables globales a GitLab para poder ejecutar comandos especiales.

```
   COPY composer.*json ./
   COPY composer.*lock ./
```

Estas dos lineas copian el composer.json y composer.lock de nuestra instalacion a la imagen que se creara, 
El archivo Dockerfile fue creado en la raiz, por lo que para especificar a donde mandarlo basta con ./
Copiando esto tenemos las dependencias que se deberan instalar en el despliegue.

```
    RUN git config --global http.sslverify false && \
       composer config --global gitlab-domains ${GITLAB_DOMAIN} && \
       composer config --global gitlab-token.${GITLAB_DOMAIN} ${GITLAB_TOKEN} && \
       composer update && \
       composer global require drush/drush -W && \ 
       sed -e 's/;extension=ldap/extension=ldap/' -i /etc/php81/php.ini
```

Esto variara dependiendo las necesidades del proyecto, pero nos podemos quedar con dos lineas en particular,
"composer update" y "composer global require drush/drush -W", esto ejecutara el composer.json que se copio previamente y
se instalara el drush. Es recomendable tambien colocar los tres primeros comandos, que configuran el composer
y desactivan la proteccion ssl para evitar problemas en el proceso, pero puede no ser necesario.

    COPY . ./
    
Copia el resto de informacion del repositorio a la imagen.

# 6) Drupal y PHP para la imagen de la instalacion

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
