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

Para trabajar usaremos GitLab, donde crearemos un repositorio con el nombre que queramos para alojar el projecto,
una vez creado se nos mostraran varias maneras de enlazar el repositorio creado con nuestro proyecto, vamos a usar una de estas maneras:

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

Para esto se crea una archivo **.gitignore**

# 3) Runners

# 4) Implementacion de CI/CD para la instalacion de Drupal

# 5) Drupal y PHP para la imagen de la instalacion

# 6) Dockerfile y gitlab-ci para la instalacion de Drupal

# 7) Implementacion de CI/CD para el perfil de instalacion

# 8) Implementar generacion de paquete composer

# 9) Dockerfile y gitlab-ci para el perfil de instalacion

# 10) Subir los cambios y generar las imagenes (perfil e instalacion)

# 11) Despliegue usando la imagen

-------------------------------------------------------------------------------------------------------------

# Pasos previos necesarios para el despliegue
El despliegue de la imagen que generamos con los pasos anteriores requiere tener previamente en donde
se vaya a instalar algunos preparativos, pasaremos a ver la configuracion necesaria.
    
# Desarrollo y escalabilidad
En caso de tener el perfil, el desarrollo se hara sobre ese repositorio en concreto, y posteriormente
se actualizara el composer de la instalacion para generar la imagen actualizada, vamos a repasar esto y
la metodologia a seguir a la hora de querer escalar a otros proyectos.

Desde el lado del servidor es necesario tirar composer update, esto actualizara el paquete...
