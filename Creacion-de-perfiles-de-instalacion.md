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

Vamos a ir completando cada archivo y carpeta con lo requirido, empezamos por test.info.yml.

# test.info.yml

AL igual que en modulos o temas, este archivo es el encargado de guardar la informacion del perfil, asi como el nombre del mismo,
la version de drupal en la que corre, los temas que utilizara el perfil y los modulos que instalara.
Un ejemplo es el siguiente:
        
        name: Portal express
        type: profile
        description: 'Install with commonly used features pre-configured.'
        version: 1.0.0
        core_version_requirement: ^9
        install:
          - node
          - history
          - block
          - breakpoint
          ...
        themes:
          - olivero
          - claro
        
En este caso en install le estamos diciendo que instale todos los modulos que por defecto instala starndar, por lo que podemos copiar la lista completa de standard.info.yml, ademas usara los temas olivero y claro. Todos los modulos que se coloquen en la lista se instalaran al instalar el perfil.

# test.install

Este archivo se ejecutara durante la instalacion del perfil, la forma mas facil de instalar el perfil es dejando el archivo de la siguiente manera:

        <?php
        /**
         * @file
         * Install, update and uninstall functions for the profilename install profile.
         */
        
        use Drupal\user\Entity\User;
        use Drupal\shortcut\Entity\Shortcut;

        /**
         * Implements hook_install().
         *
         * Perform actions to set up the site for this profile.
         *
         * @see system_install()
         */
        function test_install() {
          // First, do everything in standard profile.
          include_once DRUPAL_ROOT . '/core/profiles/standard/standard.install';
          standard_install();

          // Can add code in here to make nodes, terms, etc.
        }
        
Esto lo que hace es buscar y lanzar el instalador del perfil standard, ademas podemos agregar el rol de usuario administrador, agregando las siguientes lineas dentro de la funcion de arriba de la siguiente manera:

          $user = User::load(1);
          $user->addRole('administrator');
          $user->save();

En caso de querer no utilizar este metodo de instalacion y utilizar uno mas personalizado podemos optar por utilizar el archivo .profile y no agregar las dos lineas que refieren a standard.install:

# test.profile

En caso de querer personalizar mas la instalacion agregariamos lo siguiente, sino, lo dejariamos vacio:


        <?php

        /**
         * @file
         * Enables modules and site configuration for a demo_umami site installation.
         */

        use Drupal\contact\Entity\ContactForm;
        use Drupal\Core\Form\FormStateInterface;

        /**
         * Implements hook_form_FORM_ID_alter() for install_configure_form().
         *
         * Allows the profile to alter the site configuration form.
         */
        function test_form_install_configure_form_alter(&$form, FormStateInterface $form_state) {
            $form['site_information']['site_name']['#default_value'] = 'test';
            $form['#submit'][] = 'test_form_install_configure_submit';
          }

          /**
           * Submission handler to sync the contact.form.feedback recipient.
           */
          function test_form_install_configure_submit($form, FormStateInterface $form_state) {
            $site_mail = $form_state->getValue('site_mail');
            ContactForm::load('feedback')->setRecipients([$site_mail])->trustData()->save();

            $password = $form_state->getValue('account')['pass'];
            portal_express_set_users_passwords($password);
          }

          /**
           * Sets the password of admin to be the password for all users.
           */
          function test_set_users_passwords($admin_password) {
            // Collect the IDs of all users with roles editor or author.
            $ids = \Drupal::entityQuery('user')
              ->accessCheck(FALSE)
              ->condition('roles', ['author', 'editor'], 'IN')
              ->execute();

            $users = \Drupal::entityTypeManager()->getStorage('user')->loadMultiple($ids);

            foreach ($users as $user) {
              $user->setPassword($admin_password);
              $user->save();
            }
          }
          
Esto reemplaza las lineas de instalacion de install.standard.
 
 # Carpeta config
 
Dentro de la carpeta config contamos con otras dos carpetas, install y optional.
Dentro de install iran todas las configuraciones que se instalaran al instalar el perfil, detro de optional las que no sean estrictamente necesarias para su funcionamiento, pero puedan ser requeridas, personalmente prefiero no colocar las optional y agregarlas como modulos, pero depende de cada uno.

Lo mas sencillo para empezar es copiar las que vienen por defecto es standard/config/install e ir agregando o sacando en funcion de lo que requiramos.
Un ejemplo de configuracion que viene, para poder entender que son estos archivos, es system.site.yml y system.theme.yml:

system.site.yml cuenta con la informacion del sitio, nombre, slogan, correo, paginas 404 y 403, es en si la configuracion basica del sitio.

        langcode: en
        uuid: ''
        name: 'Test'
        mail: 'correo@gmail.com'
        slogan: 'Portal express site'
        page:
          403: ''
          404: ''
          front: /node
        admin_compact_mode: false
        weight_select_max: 100
        default_langcode: en

Mientras que system.theme.yml cuenta con los temas que el sitio usara, admin y default, en este caso olivero y claro.

        admin: claro
        default: olivero

Con estos archivos el sitio sabe que debe instalar y que confirguraciones tendra el sitio al arrancar por primera vez, modificando cualquiera de estos archivos podemos por ejemplo cambia rel tema con el que el perfil se instala, aunque no es cambiando estos archivos la unica forma, tambien se puede desde el test.install.

Un ejemplo es instalar un menu al instalar el pefil, podemos colocar el archivo de instalacion del menu dentro de la carpeta install y se instalara al arrancar.

# Modulos y temas

Ademas de esto podemos definirle al perfil modulos y temas como se definen normalmente

        Leer documentacion de modulos y temas de este repositorio para mas
        
# Updates

Podemos ademas definir una carpeta updates/updates.php para actualizar el perfil

        Leer actualizacion de base de datos de este repositorio para mas.

# composer.json

Aperte de todo esto podemos definir un composer.json para llevar registro de los modulos que se instalan, esto puede ser util para en un futuro generar una distribucion a partir de este perfil.
 
 
 
 
 
 
 
 
 

