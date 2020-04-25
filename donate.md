---
layout: page
title: Love what I do?
permalink: /donate/
---

{%- assign services = site.donate_services -%}

Support me on my journey to exploring the world.

<div class="donate-services">
  {%- for service in services -%}
    <div class="title">{{ service.title }}</div>
    {%- if service.description -%}
      <div class="description">{{ service.description }}</div>
    {%- endif -%}
    {%- if service.type == 'yandex' -%}
      {%- assign config = service.config -%}
      {%- for service in config.services -%}
        <div class="service">
          <div class="description">{{ service.description }}</div>
          {% if forloop.last %}
            <a class="button" href="https://money.yandex.ru/to/{{ config.wallet }}">Donate custom</a>
          {% else %}
            <a class="button" href="https://money.yandex.ru/to/{{ config.wallet }}/{{ service.amount }}/">Donate {{ service.amount }}₽</a>
          {% endif %}
        </div>
      {%- endfor -%}
    {%- endif -%}
    {%- if service.type == 'patreon' -%}
      <div class="service">
        <div class="description">This is a win-win</div>
        <div style="float: right;">
          <a href="https://www.patreon.com/bePatron?u=33921807" data-patreon-widget-type="become-patron-button">Become a Patron!</a><script async src="https://c6.patreon.com/becomePatronButton.bundle.js"></script>
        </div>
      </div>
    {%- endif -%}
  {%- endfor -%}
</div>
