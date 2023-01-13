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

# Ejemplo con portal express

        **Pasos posteriores a configurar ddev**
        git clone git@gitlab.isaltda.com.uy:portal/intranet-maldonado-base.git
        (Base de portal express usado en maldonado intranet)
        
        cd web/profiles/
        
        git clone git@gitlab.isaltda.com.uy:portal/intranet-maldonado-perfil.git contrib/isa
        (Copia el repositorio del perfil a la capeta isa)
        
En el perfil clonado, en el archivo composer.json, cambiaremos el nombre del paquete que se creara.

        ```
            {
                "name": "(AQUI VA EL NOMBRE DEL PAQUETE)/isa",
                "description": "Profile base for generic portals.",
                "type": "drupal-profile",
                "license": "GPL-2.0-or-later",
                "authors": [
                    {
                        "name": "Leandro Mesa",
                        "email": "leandro.mesa@isaportal.uy"
                    }
                ],
                "require": {
                ...
        ```
Esto nos aseguramos que se genere el correspondiente paquete que se implementara en el sitio.

Ahora configuramos el Dockerfile de este perfil para que sepa donde general el paquete anterior.

        ```
            #FROM registry.gitlab.com/digitalprojex-public/dockerfiles/drupal
            FROM gitlab.isaltda.com.uy:5005/NOMBRE_GRUPO/NOMBRE_REPOSITORIO_PERFIL/stable

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

            # Aqui se obtiene la ultima version del diseño, se pudiera hacer de otra forma
            RUN git clone https://${GITLAB_ISA_USER}:${GITLAB_ISA_TOKEN}@gitlab.isaltda.com.uy/FBalboa/portal-express-drupal.git \
                web/profiles/contrib/isa/themes/inten/design

        ```

En este punto desarrollamos lo que precisemos en el perfil, posterior hacemos lo siguiente:

        (En web/profiles/contrib/isa)
        git add .
        (Se agregan todos los cambios realizados)
        git commit -m "Comentario"
        git push -u origin master
        
        git tag 1.1.x
        (se agrega un tag con 1.1.x, con x numero entre 0 y 9)
        git push origin 1.1.x
        (Se agrega todo lo desarrollado al tag)

En este punto se tiene el perfil actualizado y el paquete de composer pronto, ahora en la base del sitio (el primer
paquete que clonamos), en el composer.json agregamos el paquete que se generara del perfil, correspondiente al nombre
asignado en el composer.json del perfil.

        ```
            {
              "name": "drupal/recommended-project",
              "description": "Project template for Drupal 9 projects with a relocated document root",
              "type": "project",
              "license": "GPL-2.0-or-later",
              "homepage": "https://www.drupal.org/project/drupal",
              "support": {
                "docs": "https://www.drupal.org/docs/user_guide/en/index.html",
                "chat": "https://www.drupal.org/node/314178"
              },
              "repositories": {
                "gitlab.isaltda.com.uy/154": {
                    "type": "composer",
                    "url": "https://gitlab.isaltda.com.uy/api/v4/group/154/-/packages/composer/packages.json",
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
              },
              "require": {
                "composer/installers": "^1.9",
                "cweagans/composer-patches": "^1.7",
                "(AQUI VA EL NOMBRE DEL PAQUETE)/isa": "^1",
                ...
        ```

Ahora es necesario eliminar el composer.lock, para que regenere la referencia del paquete que colocamos en los require.
Posterior a esto subimos al repositorio.

        git add .
        (Se agregan todos los cambios realizados)
        git commit -m "Comentario"
        git push -u origin master
        
        git tag 1.1.x
        (se agrega un tag con 1.1.x, con x numero entre 0 y 9)
        git push origin 1.1.x
        (Se agrega todo lo desarrollado al tag)
        
De esta forma ya queda subida la imagen para subir al servidor, con el paquete correspondiente.

# Base de datos
Al igual que en otros portales la base de datos se carga desde el contenedor que creamos (en el servidor) o desde el repositorio
clonado (en caso de desarrollo local)
        
        ddev drush sqlq --file=(URL ABSOLUTA DE LA BASE DE DATOS)
        (En caso de no tener ddev se utiliza vendor/bin/drush)
        ddev drush cr

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
