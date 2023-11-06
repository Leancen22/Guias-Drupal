# Vista en formato de tabs en drupal, mediante la carga de bloques.

<img width="557" alt="Screenshot 2023-11-06 182834" src="https://github.com/Leancen22/Guias-Drupal/assets/64225835/0859d9f5-4fce-4e14-a16a-11b9a9e8d55a">

La vista debe ser creada a partir de bloques, itera bloques de contenido que traen la informacion de los campos, en el formato de la vista debe
agruparse en funcion de un campo que tenga el nombre de la categoria del tab.

Archivo container--view--VIEW_ID--block-1.html.twig
```
<div class="component-tabs-content">
  <div class="page-container">

    {{ title_prefix }}
        <h2 class="section__title fadeInUp"> {{ element['#title']['#markup'] }} </h2>
    {{ title_suffix }}
    <p class="section__undertitle fadeInUp"> Escuchamos a nuestros clientes y trabajamos mano a mano para lograr soluciones que se ajusten a sus necesidades. </p>

    <div class="tabs">
        {{ children }}
    </div>
  </div>
</div>
```

Archivo block--views-block--VIEW_ID-block-1.html.twig
```
<div{{ attributes }}>
    {{ title_prefix }}
    {% if label %}
        <h2{{ title_attributes }}>{{ label }}</h2>
    {% endif %}
    {{ title_suffix }}
    {% block content %}
        {{ content }}
    {% endblock %}
</div>
```

Archivo views-view--VIEW_ID.html.twig
```
{%
  set classes = [
    'view',
    'view-' ~ id|clean_class,
    'view-id-' ~ id,
    'view-display-id-' ~ display_id,
    dom_id ? 'js-view-dom-id-' ~ dom_id,
  ]
%}

<div{{ attributes.addClass(classes) }}>
  {{ title_prefix }}
  {% if title %}
    {{ title }}
  {% endif %}
  {{ title_suffix }}
  {% if header %}
    <div class="view-header">
      {{ header }}
    </div>
  {% endif %}
  {% if exposed %}
    <div class="page-container">
      {{ exposed }}
    </div>
  {% endif %}
  {% if attachment_before %}
    <div class="attachment attachment-before">
      {{ attachment_before }}
    </div>
  {% endif %}

  {% if rows %}
        <div class="tab-header">
            <div class="tab-slider">
                <div class="tab-inputs fadeInUp">
        {% for item in rows %}
            <input type="radio" class="tab-input" name="tab" id="tab{{ loop.index }}" checked>
            <label for="tab{{ loop.index }}" class="label-tab"> <span> {{item['#title']}} </span> </label>      
        {% endfor %}
                </div>
            </div>
        </div>
        <div class="tab-body">
            {# {{dump(rows.0['#rows'].0['#row']._entity)}} #}
            {% for value in rows %}
                <div class="tab-content" data-tab="tab{{ loop.index }}">
                {% for valor in value['#rows'] %}
                    {{valor}}
                {% endfor %}
                </div>
            {% endfor %}
        </div>
  {% elseif empty %}
    <div class="view-empty">
      {{ empty }}
    </div>
  {% endif %}

  {% if pager %}
    {{ pager }}
  {% endif %}
  {% if attachment_after %}
    <div class="attachment attachment-after">
      {{ attachment_after }}
    </div>
  {% endif %}
  {% if more %}
    {{ more }}
  {% endif %}
  {% if footer %}
    <div class="view-footer">
      {{ footer }}
    </div>
  {% endif %}
  {% if feed_icons %}
    <div class="feed-icons">
      {{ feed_icons }}
    </div>
  {% endif %}
</div>
```

Archivo views-view-unformatted--VIEW_ID.html.twig
```
{% if title %}
  <h3>{{ title }}</h3>
{% endif %}
{% for row in rows %}
    {{- row.content -}}
{% endfor %}
```
