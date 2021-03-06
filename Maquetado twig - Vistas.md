# MAQUETADO TWIG DE VISTAS 

Para maquetar en twig una vista que se muestra en formato bloque, debemos identificar que componentes van a participar y como utilizarlos. 

Primero crearemos el archivo twig dentro de templates/block, que sera el encargado de "englobar" toda la seccion de la vista, 
usaremos la siguiente estructura para crear los archivos:

    block--views-block--NOMBREVISTA-block-1.html.twig 
                                      
La estructura dentro de este archivo sera como la siguiente: 

```json

          <div{{ attributes.addClass(['clase correspondiente']) }}>
              {{ title_prefix }}
              {{ title_suffix }}
              {% if label %}
                  <h3{{ attributes.addClass(['clase correspondiente']) }}>{{ label }}</h3>
              {% endif %}
              {% block content %}
                  <div class="novedades__list">
                      {{ content }}
                  </div>
              {% endblock %}
          </div>
          
``` 
Donde el h3, es donde se mostrará el nombre de la vista. 

Luego crearemos el archivo dentro de templates/views donde agregaremos las clases correspondientes a la visualizacion de la vista. En el caso de que se quiera usar la 
visualizacion genérica, se puede usar el archivo templates/views/views-view.html.twig

Como la visa creada tiene una previsualizacion en formato de lista html, creamos el archivo dentro de template/views:

    views-view-list--NOMBREVISTA.html.twig

En la cual agregamos la clase correspondiente del carrusel, y quedo de esta forma:


```json

<{{ list.type }}{{ list.attributes }}>

<div class="owl-carousel owl-carousel-novedades">
{% for row in rows %}
    <li{{ row.attributes.addClass('novedad__item') }}>
        {{- row.content -}}
        <div class="item__link">ver más</div>
    </li>
{% endfor %}
</div>

</{{ list.type }}>

```

Y finalmente empezaremos a maquetar los campos de el tipo de contenido que mostraremos en la vista. 

Primero, crearemos el archivo twig del nodo principal de la vista, el cual lo haremos dentro de templates/node y será de la siguiente forma:

    node--NOMBREVISTA--list-box.html.twig

en donde se colocaran las clases correspondientes a cada item para mostrar.

Y luego creamos los archivos correspondientes para maquetar cada campo del contenido, en el caso de que se quieran agregar clases especificas para cada campo,
estos los crearemos todos dentro de la carpeta template/fields

Por ejemplo, para el tipo de contenido novedades, se crearon los siguientes archivos:

    field--node--field-imagen-novedades.html.twig (para el campo de imagenes)
    field--node--field-titul.html.twig (para el campo titulo)
    field--node--field-categoria-novedades.html.twig (para el campo categorias)
    field--node--body--novedades.html.twig (para el campo body)
