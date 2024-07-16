#  Kint

Kint es una herramienta que nos permite depurar el codigo en los hook y plantillas Twig, permitiendonos tener total conocimiento de las variables 
que se reciben en los templates.

# Instalación de Kint

Para instalar kint necesitamos previamente instalar el módulo devel, para ello se debe ejecutar:

    composer require drupal/devel
    
    drush en devel devel-generate
    
    Más sobre devel ---> leer (Herramientas-para-desarrollo-backend.md)
    
Posterior a esto instalamos el kint para depurar usando devel:

    composer require kint-php/kint
    
Esto ya nos permite en las plantillas twig o archivos hooks ejecutar la funcion:

    kint($variables)  ---> En caso de hooks
    
    {{ kint($variables) }}  ---> En caso de twig
   
# Alternativa y posibles errores de visualización

Una alternativa, no tan potente para kint es la funcion dump:

    dump($variables)  ---> En caso de hooks
    
    {{ dump($variables) }}  ---> En caso de twig
    
Un posible error de visualización puede ocurrir si contamos con kint como depurador por defecto en devel, lo ideal es que no sea asi.

# Mostrar suggestions y archivos twigs

Al trabajar con kint o dump es muy util conocer sobre que plantilla debemos lanzarlo para ver las variables que se reciben en ella, para ello activaremos
el visualizador de plantillas, siguiendo los pasos siguientes:

    En la carpeta /sites deberemos agregar (o modificar) el archivo development.services.yml, quedandonos de la siguiente manera:
	
```json
        parameters:
          http.response.debug_cacheability_headers: true
          twig.config:
            debug: true
            auto_reload: true
            cache: false
        services:
          cache.backend.null:
            class: Drupal\Core\Cache\NullBackendFactory
```
     
    Esto nos activa la depuración twig.

    Posterior a esto en la carpeta /sites/default/ crearemos (o modificaremos) el archivo settings.local.php, quedandonos 
    de la siguiente manera:
    
```json
        <?php
	#assert_options(ASSERT_ACTIVE, TRUE); Deprecated in 10.1
	assert_options(ASSERT_EXCEPTION, TRUE)
	\Drupal\Component\Assertion\Handle::register();
	$settings['container_yamls'][] = DRUPAL_ROOT . '/sites/development.services.yml';
	$config['system.logging']['error_level'] = 'verbose';
	$config['system.performance']['css']['preprocess'] = FALSE;
	$config['system.performance']['js']['preprocess'] = FALSE;
	$settings['cache']['bins']['render'] = 'cache.backend.null';
	$settings['cache']['bins']['discovery_migration'] = 'cache.backend.memory';
	$settings['cache']['bins']['page'] = 'cache.backend.null';
	$settings['cache']['bins']['dynamic_page_cache'] = 'cache.backend.null';
	$settings['rebuild_access'] = TRUE;
	$settings['skip_permissions_hardening'] = TRUE;
```
	
    Con esto requerimos el archivo anterior y activamos configuraciones, como por ejemplo, desactivar el cache dinámico, interno,
    agregacción de css y js, etc.
    
    Lo que nos faltaría ahora simplemente sería descomentar (o agregar) las siguientes lineas en el settings.php:
    

    if (file_exists($app_root . '/' . $site_path . '/settings.local.php')) {
      include $app_root . '/' . $site_path . '/settings.local.php';
    }

# Recomendación

Se recomienda instalar además el modulo twig_tweak, que mejora la experiencia de desarrollo con twigÑ

    composer require drupal/twig_tweak
