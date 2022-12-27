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
    Vamos primero a conocer la estructura del proyecto en el que implementaremos Docker, para este proyecto como se menciono se utilizara 
    un perfil de instalacion donde se desarrollara, en caso de no contar con un perfil de instalacion para el desarrollo y desarrollar sobre
    la misma instalacion de drupal, puede ignorar los pasos relacionados al perfil.



Desde el lado del servidor es necesario tirar composer update, esto actualizara el paquete...
