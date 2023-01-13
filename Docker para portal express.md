#  Docker para Portal Express o similares
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
