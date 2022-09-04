# Perfiles de instalación

Aunque Standard cuente con todo lo necesario para contruir un portal con Drupal,
ocasiones se puede requerir tener una instalación unica para nuestro sitio, para esto se crean perfiles
de instalación especificos los cuales cuentan con módulos, temas y configuraciones especificas para nuestro portal,
las cuales se impactaran cuando este se instale.

# Por donde empezar?
Si queremos crear un perfil desde cero, lo primero es entender como es la estructura de carpetas que lo componen, 
los perfiles customizados se crean en:

    web/profiles
   
dentro debemos generar la carpeta custom, donde guardaremos la carpeta de nuestro perfil, supongamos en este caso test.

La composicion de la carpeta test sera la siguiente:

    test
      -config
        --install
        --optional
      -modules (opcional)
      -themes (opcional)
      -test.info.yml
      -test.install
      -test.profile

Adicionalmente se pueden colocar otras carpetas o archivos de configuracion, pero estas son las minimas requeridas para que el perfil funcione.


