#  Implementación con Docker y despliegue

En esta guía se verá como implementar Docker para el mantenimiento y despliegue de un sitio Drupal.

Veremos como generar las imágen de Docker para el Drupal base que levantará el sitio y luego como
generar la imágen para en caso de que lo tengamos, poder desarrollar sobre un perfil de instalación e integrarlo 
mediante un paquete de composer.

#  Importante

En caso de necesitar ver como realizar la instalación de Drupal en profundidad consultar el resto del material.

No es necesario para poder seguir la guía pero es util tener conocimientos básicos de como funciona Docker, git,
runners, drush, composer y ddev (este último para desarrollo).

Este desarrollo se hara utilizando la plataforma GitLab, la cual tiene sus diferencias con Github a la hora de trabajar
con CI/CD.

Esta guia se hara para Drupal 9 y PHP 8.x

En caso de estar trabajando con Portal Express o derivados puede consultar directamente la guia correspondiente a Docker para portal express.

#  Paso a paso
# 1) Estructura del proyecto

 Vamos primero a conocer la estructura del proyecto en el que implementaremos Docker, para este proyecto como se menciono
 se utilizara un perfil de instalacion donde se desarrollara, en caso de no contar con uno para el
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

Para trabajar usaremos GitLab, donde crearemos un repositorio con el nombre que queramos para alojar el proyecto,
una vez creado se nos mostraran varias maneras de enlazar el repositorio creado con nuestro proyecto, vamos a usar una de estas maneras.

En la carpeta que creamos anteriormente (ProjectoConDocker) usaremos el siguiente comando:
   
    git init
    
Esto nos creara una instancia de git (carpeta .git) que nos permitira enlazar a un repositorio remoto.
Ahora toca "decirle" a donde debe mandar los cambios, esta sera la conexion con el repositorio, para esto
nos vamos a nuestro repositorio creado y compiamos la ruta de clonado del mismo, puede ser por HTTP o SSH.

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
lo hace, por lo que tenemos que prohibir la subida de esta informacion, ademas de esto, tampoco deben subirse archivos o carpetas que
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
Tal como hicimos con la instalacion de Drupal, debemos crear el archivo **.gitlab-ci.yml** e indicar las instrucciones que 
debe seguir gitlab cuando se suba un nuevo tag, este es el contenido que se usa en este caso.

```
    stages:
      - build

    variables:
      GIT_SUBMODULE_STRATEGY: none

    build_composer:
      stage: build
      image: alpine/curl
      script:
        - |-
            TARGET=tag
            if [ "${CI_COMMIT_REF_NAME}" = "master" ]; then
              TARGET=branch
            fi

            curl -k --show-error --fail --header "Job-Token: $CI_JOB_TOKEN" \
                 --data ${TARGET}=${CI_COMMIT_REF_NAME} \
                 "$CI_API_V4_URL/projects/$CI_PROJECT_ID/packages/composer"
      only:
        - /^[0-9]+\.[0-9]+\.[0-9]+$/
      tags:
        - docker2
```
Recordemos que la idea con el perfil es generarlo como un paquete de composer que se detectara cuando se haga un update en nuestro sitio,
por lo que lo que estamos haciendo en el codigo de arriba es establecer la ruta a donde se mandara el paquete, recordar es necesario tener
habilitada la posibilidad de crear paquetes de composer en nuestro repositorio.

Ya vimos que representaban algunas de las variables que se ven en el codigo, veremos las dos que no vimos.
**$CI_API_V4_URL** URL de la api v4 de GitLab, por ejemplo https://gitlab.EXTENSION.com.uy/api/v4
**$CI_PROJECT_ID** Id del proyecto en el que estamos

Los paquetes se van a ir creando en https://gitlab.EXTENSION.com.uy/NOMBRE_GRUPO/NOMBRE_PROYECTO/-/packages

# 8) Implementar generacion de paquete composer
Para que se genere el paquete de composer es necesario especificar un archivo composer que indique los parametros que se
guardaran en estos paquetes.
Hay varias formas de hacer esto, pero veamos solo una de ellas.

En la ruta raiz del perfil de instalacion ejecutamos:
    
    composer init
    
Esto nos comenzara a preguntar alguas cosas, podemos dejarlo por dejecto.
Recordemos que el nombre que asignemos a este composer.json que se genera sera el nombre del paquete composer y
con lo que se identificara en el composer de la instalacion de Drupal.

```
{
    "name": "nombre/paquete",  <--- Este es el nombre que representa el paquete
    "description": "Profile base for generic portals.",
    "type": "drupal-profile",
    "license": "GPL-2.0-or-later",
    "authors": [
        {
            "name": "Leandro Mesa",
            "email": "leandro.mesa@isaportal.uy"
        }
    ],

```

Esto depende siempre del proyecto, se podria aregar un parametro require y colocar los modulos necesarios a instalar,
parches y demas.

Ejemplo:

```
     "require": {
        "cweagans/composer-patches": "^1.7",
        "drupal/admin_toolbar": "3.0.3",
        "drupal/autosave_form": "^1.2",
        "drupal/back_to_top": "^1.1",
        "drupal/block_content_permissions": "^1.10",
        "drupal/blockgroup": "^1.5",
        "drupal/bootstrap_layout_builder": "^2.0",
        "drupal/bootstrap_styles": "^1.0",
        "drupal/captcha": "^1.1",
        "drupal/checklistapi": "^2.0",
        "drupal/ckeditor_scayt": "^1.1",
        "drupal/ckwordcount": "^1.1",
        "drupal/config_split": "^2",
        "drupal/config_update": "^1.7",
        "drupal/console": "~1.0",
        "drupal/contact_block": "^1.5",
        "drupal/ctools": "^3.4",
        "drupal/default_content": "^2.0",
        "drupal/devel": "4.1.1",
        "drupal/easy_breadcrumb": "2.0.1",
        "drupal/editor_advanced_link": "2.0.0",
        "drupal/empty_page": "^3.0",
        "drupal/entity_hierarchy": "^3.0",
        "drupal/entity_reference_hierarchy": "^1.0@beta",
        "drupal/entity_reference_revisions": "^1.8",
        "drupal/features": "^3.11",
        "drupal/file_entity": "^2.0@beta",
        "drupal/fixed_block_content": "^1.1",
        "drupal/google_analytics": "4.0.0",
        "drupal/group": "^1.4",
        "drupal/health_check_url": "3.1",
        "drupal/hreflang": "^1.3",
        "drupal/imce": "^2.2",
        "drupal/key_value_field": "^1.2",
        "drupal/layout_builder_restrictions": "^2.8",
        "drupal/linkit": "^6.0@beta",
        "drupal/magnific_popup": "^1.5",
        "drupal/menu_block": "^1.7",
        "drupal/menu_trail_by_path": "^1.3",
        "drupal/metatag": "^1.14",
        "drupal/metatag_routes": "^1.2",
        "drupal/notification": "^1.1",
        "drupal/openid_connect": "^1.0@RC",
        "drupal/paragraphs": "^1.12",
        "drupal/pathauto": "^1.8",
        "drupal/permissions_filter": "^1.1",
        "drupal/recaptcha": "^3.0",
        "drupal/recaptcha_v3": "^1.3",
        "drupal/redirect": "^1.6",
        "drupal/search404": "2.0.0",
        "drupal/search_api": "^1.17",
        "drupal/search_api_page": "^1.0@beta",
        "drupal/seo_checklist": "5.1.0",
        "drupal/simple_gmap": "^3.0",
        "drupal/simple_sitemap": "^4.0@alpha",
        "drupal/sitemap": "^2",
        "drupal/smart_trim": "^1.3",
        "drupal/smtp": "^1.0",
        "drupal/social_media": "^1.9@RC",
        "drupal/subpathauto": "^1.1",
        "drupal/svg_image": "^1.14",
        "drupal/textarea_widget_for_text": "^1.2",
        "drupal/time_picker": "^5.3",
        "drupal/toastr": "^1.0",
        "drupal/token": "^1.7",
        "drupal/tvi": "^1.0@RC",
        "drupal/twig_field_value": "^2.0",
        "drupal/twig_tweak": "^3.1",
        "drupal/twigsuggest": "^1.0@beta",
        "drupal/w3c_validator": "^1.4",
        "drupal/webform": "^6.1@beta",
        "drupal/yoast_seo": "^2",
        "drush/drush": "^10.6",
        "drupal/iframe": "^2.12",
        "drupal/views_block_placement_exposed_form_defaults": "^1.0@RC",
        "drupal/maxlength": "^2.0@RC",
        "drupal/scheduler": "^2.0@alpha",
        "drupal/extlink": "^1.6",
        "drupal/material_icons": "^1.0",
        "drupal/multivalue_form_element": "^1.0@beta"
     },
     "extra": {
         "enable-patching": true,
         "patches": {
             "drupal/default_content": {
                 "Add a Normalizer and Denormalizer to support Layout Builder": "https://www.drupal.org/files/issues/2020-08-24/default_content-add-normalizer-denormalizer-for-layout-builder-3160146-25.patch"
             },
             "drupal/time_picker":{
                 "Console error when using time range picker": "https://www.drupal.org/files/issues/2020-05-05/console-error-fix.patch"
             },
             "drupal/views_block_placement_exposed_form_defaults":{
                 "All results displayed in block when single exposed filter value selected": "https://www.drupal.org/files/issues/2021-10-21/views_block_placement_exposed_form_defaults-filter_displays_all_results-3244432-4.patch"
             }
         }
     }

```

# 9) Dockerfile para el perfil de instalacion
Como hicimos con el Dockerfile de la instalacion, este sera el archivo que busque Docker para armar la imagen.
Depende de cada perfil y proyecto el contenido del Dockerfile, pero podemos ver algunos patrones generales.

```
FROM gitlab.EXTENSION.com.uy:5005/NOMBRE_GRUPO/NOMBRE_PROYECTO/stable

ARG GITLAB_TOKEN
ARG GITLAB_DOMAIN
ARG GITLAB_USER
ARG GITLAB_TOKEN
ARG GROUP_ID
ARG APP_PACKAGIST
ARG API_V4_URL
ARG GITLAB_ISA_USER
ARG GITLAB_ISA_TOKEN

RUN composer config --global gitlab-domains ${GITLAB_DOMAIN} && \
    composer config --global gitlab-token.${GITLAB_DOMAIN} ${GITLAB_TOKEN} && \
    composer create-project drupal/recommended-project:^9 ./ && \
    composer config repositories.${GITLAB_DOMAIN}/${GROUP_ID} \
     '{"type": "composer", "url": "'${API_V4_URL}'/group/'${GROUP_ID}'/-/packages/composer/packages.json"}' && \
    composer config --json extra.enable-patching true && \
    composer config minimum-stability dev && \
    composer require "${APP_PACKAGIST}" --no-interaction && \
    composer update --lock
```

Puede ser que se quiera agregar un tema que se encuentre en otro repositorio o modulos, etc, todo eso se puede hacer en el RUN.

# 10) Subir los cambios y generar las imagenes (perfil e instalacion)
Ya hemos ido viendo como se suben los cambios a ambios repositorios, y que estas imagenes que hemos ido creando se 
crear una vez subimos los tags, vamos a ver un ejemplo de como se implementa el perfil de instalacion para generar la imagen 
actualizada con los cambios que desarrollemos.

recordemos que el paquete que creamos como ejemplo se llama nombre/paquete, en este punto es necesario indicarle al composer.json
de la instalacion de donde tiene que obtener la informacion, osea el repositorio, donde alojamos el perfil.

En nuestro **composer.json**, antes de los require, colocamos lo siguiente:

```
"repositories": {
    "gitlab.EXTENSION.com.uy/ID_DEL_GRUPO": {
        "type": "composer",
        "url": "https://gitlab.EXTENSION.com.uy/api/v4/group/ID_DEL_GRUPO/-/packages/composer/packages.json",
        "options": {
            "ssl": {
                "verify_peer": false,
                "allow_self_signed": true,
                "secure-http": false
            }
        }
    },
    "0": {
        "type": "composer",
        "url": "https://packages.drupal.org/8"
    }
}
```

La variable ID_DEL_GRUPO se puede obtener viendo el id del grupo donde tenemos el proyecto del perfil, esto verifica el paquete composer que
creamos anteriormente y desactiva algunas opciones de seguridad.

Posterior a esto podemos en los require agregar el paquete en este caso:

```
...
"require": {
    ...
    "nombre/paquete": "^1",
    ...
...
```
Con esto ya estamos detectando el paquete, la version "^1", corresponde al major del tag, en caso de que nuestros
tag pasen a 2.x.x, sera necesario cambiarla manuealmente por "^2", para cualquier valor de 1.x.x podemos usar esa
configuracion.

Sera necesario reconstruir el composer.lock con estas nuevas configuraciones agregadas, para esto lo eliminamos
y ejecutamos **composer update** nuevamente, un ejemplo de composer.lock actualizado podria tener lineas similares
a estas:

```
{
     "name": "nombre/paquete",
     "version": TAG_VERSION,
     "source": {
         "type": "git",
         "url": "https://gitlab.EXTENSION.com.uy/NOMBRE_GRUPO/NOMBRE_PROYECTO.git",
         "reference": "8d46a79300522974702bfdd4813528006499cb3c"
     },
     "dist": {
         "type": "zip",
         "url": "https://gitlab.EXTENSION.com.uy/api/v4/projects/810/packages/composer/archives/XXX/isa.zip?sha=8d46a79300522974702bfdd4813528006499cb3c",
         "reference": "8d46a79300522974702bfdd4813528006499cb3c",
         "shasum": ""
     },
     "require": {
     ...
```
Esta configuracion la genera composer.lock automaticamente.

**IMPORTANTE**
Es necesario crear una imagen para cada una, primero el perfil y luego la instalacion.
Veamos un ejemplo, aunque se explayara un poco en la seccion de desarrollo.

```
**Perfil de instalacion**

git add . (todos los cambios que se deseen)
git commit -m "ejemplo de subida"
git push -u origin master
git tag 1.0.1
git push origin 1.0.1

Se crea el paquete de composer

Para impactar los cambios en la imagen es necesario actualizar el paquete de composer del perfil, antes de subir
los cambios y el tag de la instalacion es necesario un composer update.

**Instalacion de Drupal**

git add . (todos los cambios que se deseen)
git commit -m "ejemplo de subida"
git push -u origin master
git tag 1.0.0
git push origin 1.0.0
```

Explayaremos en la seccion de desarrollo y escalabilidad.

# 11) Despliegue usando la imagen
Leer primero el apartado de "pasos previos necesarios para el despliegue".
...

-------------------------------------------------------------------------------------------------------------

# Posibles errores

# Pasos previos necesarios para el despliegue
El despliegue de la imagen que generamos con los pasos anteriores requiere tener previamente en donde
se vaya a instalar algunos preparativos, pasaremos a ver la configuracion necesaria.

Necesitamos tener un drupal con un composer.json con los siguientes modulos, como minimo:
     
     drush (modulo para el manejo del cache, bases de datos, otros modulos entre otros)
     drupal_console (modulo necesario para poder ejecutar comandos relacionados con drupal, modulos y temas)

Estos requerimientos se suplen instalando en el servidor la imagen del drupal generico que se usa para contruir nuestra imagen
con nustras configuraciones y modulos.
Simplemente basta desplegar esta imagen de drupal generico en el servidor y "pasar" por encima con nuestra imagen.
    
# Desarrollo y escalabilidad
En caso de tener el perfil, el desarrollo se hara sobre ese repositorio en concreto, y posteriormente
se actualizara el composer de la instalacion para generar la imagen actualizada, vamos a repasar esto y
la metodologia a seguir a la hora de querer escalar a otros proyectos.

Con lo desarrollado en esta guia podemos simplemente crear nuevos repositorios cada vez que queramos implementar
un nuevo perfil con otro tema, ya sea para otro portal que estemos desarrollando u otro proyecto aparte.

El cambio que podemos hacer para diferenciar entre proyectos, portales o clientes es el nombre con el que se crea el
paquete de composer en el composer init, basta simplemente cambiar el name, y luego el require del composer.json, esto detectara,
una vez eliminado el composer.lock y generado nuevamente con el composer update, se detectara a que repositorio corresponde el paquete.

Tambien es necesario ajustar el Dockerfile en el perfil de instalacion.

    FROM gitlab.EXTENSION.com.uy:5005/NOMBRE_GRUPO/NOMBRE_PROYECTO/stable

Esto es el enlace de donde se alimenta el Dockerfile y corresponde a cada repositorio del perfil en particular.

------------------------------------------------------------------------------------------------------------------

Para desarrollar en local tenemos que clonar ambos repositorios primero la instalacion y luego el perfil donde corresponda.
Para desarrollar con ddev, configuramos ddev como habiamos hecho en el paso 1.

    git clone RUTA_DE_LA_INSTALACION_DRUPAL
    cd NOMBRE_DE_LA_CARPETA_QUE_GENERA_EL_CLONADO
    cd web/profiles/contrib
    git clone RUTA_DEL_PERFIL_DE_INSTALACION
    
    cd
    cd NOMBRE_DE_LA_CARPETA_QUE_GENERA_EL_CLONADO
    ddev composer update
   
    //Instalacion con drush 
    drush si --locale=es \
      --account-name=Admin \
      --account-pass="Admin" \
      --site-name="Nombre sitio" \
      --site-mail=nombre-correo@gmail.com \
      --yes \
      && drush locale-check && drush locale-update \
      && drush sapi-r \
      && drush pag all all \
      && drush cr
    
    //Conectar base de datos
    ddev ssh
    drush sqlq --file=/var/www/html/ARCHIVO_BASE_DE_DATOS
    
    drush cr
...

**IMPORTANTE**
Cuando se ejecuta el composer update para actualizar el paquete de composer se pierde el enlace al repositorio para el perfil,
osea, pierde la carpeta .git, por lo que cada vez que actualicemos el composer.json para actualizar el paquete composer es necesario volver
a clonar el repositorio del profile

    cd web/profiles/contrib
    rm -rf NOMBRE_CARPETA_PERFIL_INSTALACION
    git clone RUTA_DEL_PERFIL_DE_INSTALACION
    
**ENLACE SIMBOLICO**
Tambien podemos generar un enlace simbolico para no tener que clonar cada vez, simplemente seria reconstruir el enlace cada vez
que se actualice, en este caso en lugar de clonar el perfil donde corresponde dentro de la instalacion, se clonara en otro sitio.

    Donde se configura ddev, tendremos la carpeta .ddev
    cd .ddev
    git clone RUTA_DEL_PERFIL_DE_INSTALACION
    
    //Crea el enlace simbolico
    ln -sr .ddev/NOMBRE_CARPETA_PERFIL_INSTALACION web/profiles/contrib/NOMBRE_CARPETA_PERFIL_INSTALACION
    
    //Luego del update en la instalacion para actualizar el paquete composer
    rm -r web/profiles/contrib/NOMBRE_CARPETA_PERFIL_INSTALACION
    ln -sr .ddev/NOMBRE_CARPETA_PERFIL_INSTALACION web/profiles/contrib/NOMBRE_CARPETA_PERFIL_INSTALACION
    
