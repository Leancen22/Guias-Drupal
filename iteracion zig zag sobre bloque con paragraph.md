# Zig Zag en bloque con paragraph

block--content--BLOCK_ID.html.twig

```
{% set items = content.field_servicio_con_caracteristic|default([]) %}
{% set count = items|filter((item, key) => key|first != '#')|length %}

<div{{ attributes }}>
    {{ title_prefix }}
    {% if label %}
        <h2{{ title_attributes }}>{{ label }}</h2>
    {% endif %}
    {{ title_suffix }}
    {% block content %}
        {% for key, item in items %}
            {% if key|first != '#' %}
            {% set classes = [
                'component-txt-img',
                loop.index is odd ? 'component-txt-imgRight' : 'component-txt-imgLeft',
                loop.index is odd ? 'component-txt-imgRight-alt' : 'component-txt-imgLeft-alt',
                loop.index is odd ? '' : 'bg-color',
            ] %}

            <div class="{{ classes|join(' ') }}">
                {{ item }}
            </div>
            {% endif %}
        {% endfor %}
    {% endblock %}
</div>
```
