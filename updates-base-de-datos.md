# Updates base de datos

Las actualizaciones de la base de datos para el perfil de instalación se hacen desde updates/updates.php

Para ello se utiliza el hook HOOK_update_N donde HOOK es el nombre del perfil y N el numero del update.

En el caso de portal express:
    
    isa_update_9001() {
      ...
    }

    9001 ---> 9, version de drupal en la que esta corriendo el perfil
              001, update a realizar
              
# 999 updates

En caso de alcanzar el limite de updates, es necesario tener una base de datos en la última versión posible con todos los updates realizados.
En ese momento se puede devolver el estado del HOOK_update a 9000

    Para saber en que version vamos se puede utilizar el siguiente comando:

        PARA DRUPAL 8/9
        drush php-eval "echo drupal_get_installed_schema_version('NAME');"

        PARA DRUPAL 9.3/10+
        drush php-eval "echo \Drupal::service('update.update_hook_registry')->getInstalledVersion('NAME');"
        
    Para restablecer el update a 9000:
    
        drush php-eval "echo drupal_set_installed_schema_version('NAME', '9000');"
        
        
# Actualizar base

Para que el archivo updates.php sea visto por drush debe estar requerido al final del archivo nameProfile.install:

    require_once 'updates/updates.php';
  
  
  
# Ejemplo de uso (Instalar o desinstalar módulos)

Supongamos que queremos instalar un módulo y desinstalar otro, en este caso nuestro update sería el siguiente:

    function NameProfile_update_9001(){
      \Drupal::service('module_installer')->install([
        'modulo1',
      ]);
      
      \Drupal::service('module_installer')->uninstall([
        'modulo2'
      ]);
    }
    
Para ejecutar el update usamos el siguiente comando de drush:

    drush updb --yes
    
Si todo es correcto se nos reportará en consola el update pendiente y se insertarán los cambios en la base de datos.

Finalmente lanzamos la siguiente serie de comandos para finalizar: 

    drush locale-check && drush locale-update && drush cr
    
    
# Actualizar configuraciones especificas de los módulos

Si estamos modularizando nuestros contenidos, bloques, etc, este HOOK nos permite actualizar las configuraciones del modulo sin la necesidad de desinstalarlo, utilizando el siguiente servicio dentro del HOOK_update:

    \Drupal::service('config.installer')->installDefaultConfig($type, $name)
    
    Donde $type corresponde al tipo de actualizacion, ya sea module o theme y
    $name el nombre del módulo o tema a instalar
    
Este proceso se puede automatizar con una funcion en el nameProfile.install

    function _nameProfile_import_config($modules) {
      if (!is_array($modules)) {
        $modules = [$modules];
      }
      foreach ($modules as $module) {
        \Drupal::service('config.installer')
          ->installDefaultConfig('module', $module);
      }
    }
    
    # Ejemplo de uso
    
    function NameProfile_update_9001(){
      _nameProfile_import_config([
        'modulo1',
        'modulo2',
        ...
      ])
    }
    
Previamente en la carpeta de estos modulos se deberia haber colocado las respectivas configuraciones, las cuales se pueden recuperar haciendo un drush cex.
