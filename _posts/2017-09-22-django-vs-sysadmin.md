---
layout: post
title: Django vs Sysadmin
permalink: django-vs-sysadmin
comments: true
---

[![License: CC BY-NC-SA 4.0](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc-sa/4.0/)

## Prefacio
> Relataremos la historia de un brujo Sysdadmin que vivía en el Reino del lejano Backend, triste y enclaustrado entre terminales, conjurando hechizos en Perl y awk, hasta que un buen día (¿o malo quizás?) se le encomendó la noble misión de pregonar en los Siete Reinos HTML los datos que emanaban de las mazmorras SQL.
>
> Solo una gran magia podría satisfacer tan alta causa: Python.

Texto base sobre el que versará la charla [Djando vs Sysadmin][dvs-agenda] a impartir en Cáceres durante la [PyConES 2017][pycones2017-home].

Podéis acceder a las *slides* [aquí][dvs-slides].

### :warning: Disclaimer :warning:

Para ilustrar este `post` se ha creado un aplicación *demo* cuyo código reside en [este repo][repo-master] de [Github][github] y se ha estructurado en [ramas][git-branch].

Cada una de las ramas es _usable per se_ y demuestran cómo implementar una funcionalidad concreta.

Requisitos:

- Cliente [git][git-download].

- [Docker][docker-install] y [Docker Compose][docker-compose-install].

La dinámica de instalación es siempre la misma:

- `checkout` de la rama.

- Ejecución de comandos en `docker-compose`.

Las instrucciones detalladas se pueden encontrar en los `README.md` de las *branches* y en los vídeos *linkados* (¡gracias [asciinema][asciinema]!).

<hr>

Ramas:

1. Gestión de [usuarios y autenticación][repo-auth].

2. [Estructura y Apps][repo-apps]. Configurar el proyecto.

3. [DRF][repo-drf]. Diseñando nuestro *Backend*.

<hr>

Es necesario **aplicarlas en orden**, desde la `01` a la `x`, para observar cómo va creciendo y transformandose la aplicación mediante cambios incrementales, conservando las modificaciones previas en `BD`.

Finalmente podremos hacer `checkout` al *master* que contiene el código de la demo completa.

:point_right: **ATENCION**: *Material complementario*. **NO** es necesario instalar la **demo** para seguir la charla :point_left:

<hr>

## Introducción

Me llamo Juan Diego, soy Sysadmin y hasta hace cinco meses no sabía que era `HTML` :grimacing:.

Bueno ... quizás esta última afirmación sea un poco exagerada, para ser honesto había usado *templating* (`jinja2`) para generar mails *bonitos* y experimentado minimamente con `Flask`.

#### ¿Cómo me metí en este *embolao*?

Me ofrecí voluntario OMG :neckbeard: … lo cierto es que llevaba tiempo con *inquietudes web* pero nunca había dispuesto del tiempo necesario crear un *side Project*, así que cuando laboralmente se me se planteo la necesidad de elaborar una web app la ocasión la pintaron calva :godmode:.

Como suele pasar siempre cuando uno, o al menos yo, se encuentra, sin experiencia, ante una tarea de estas dimensiones. Se falla, se falla bastante :sob:.

El objetivo este `post` es claro y responde a una *pregunta*:

¿Cómo puedo ayudar a alguien que ya programe en **Python** pero sea un completo novato en el mundo web a comenzar con **Django**?

La respuesta es simple: compartiendo mi experiencia, lo que me sirvió y lo que no.

:warning: **ATENCION**: Este texto **NO** es un tutorial.

<hr>

## ¿Qué *Framework* escojo?

La respuesta es obvia :grin: y la selección natural:

- Hecho en **Python**: en mi opinión el mejor lenguaje para administrar sistemas.

- **Fiable**: GitHub Starts, comunidad, webs en producción.

- `models.py`: Mi portal debía hacer uso de **modelos** (¿?) **con persistencia** en BBDD y Django como [ORM][wiki-orm] es una pasada.

- **Extensible**: ¿os habéis dado cuenta que hablo de *portal*? ... pues este también fue un punto fundamental ya que no solo se me solicito la programción de una aplicación, además se deseaba que el sistema fuera extensible, es decir que se pudieran *acoplar* nuevas *apps* aprovechando una infraestructura común (autenticación, plantillas, etc) …. el famoso *po ya que*.

- [Django Rest Framework][drf-home] (lo comentaremos más adelante).

- El [*Admin Site*][admin-site].

- Miles de [paquetes][djangopackages].

<hr>

## Consideración previa y primer error

Caí en la tentación e intenté salir por la *vía rápida*, instale aplicaciones y boilerplate como un loco, mi objetivo era claro intentar encontrar el santo grial, un código que me lo diera todo (o mucho) hecho.

Mi primera idea fué ...¡¿lo mismo me sirve un [CSM][csm-wiki]?!

Descubrí [Awesome Django][awesome] *(highly recommended)* y probé casi todos (*Wagtail*, *Django-CMS*, *Mezzanine*) :blush:

En ocasiones, *raras*, conseguí que alguno funcionara :sunglasses:, pero lo que hacía por debajo era *magia negra* para mi :fearful: y eso me imposibilitaba poder adaptar el *CMS* a mis necesidades, el  *zen Django* todavía estaba muy lejos de mi espíritu .

Así que paré, reflexioné y decidí **empezar por los basics**: concretamente los tutoriales de [Django Girls][django-girls-tuto] y [Mozilla][mozilla-tuto] que os recomiendo fervientemente (en ese orden).

**Mi conclusión**: Para ser productivos con este framework debemos comprender que se está moviendo *under the hood*.

<hr>

## El entorno de desarrollo

El mínimo exigible es usar [virtualenv][virtualenv], con esto evitaremos al menos romper cosas.

Pero ¿Qué pasa con la base de datos? ¿y al subir a producción?

Clonar un repositorio y lanzar un comando… ya tenemos nuestra web levantada con su *proxy-server* y su *BD* ¿no resulta mágico? ¡Pues esa es la idea!

Mis herramientas favoritas para servir a este objetivo son [Git][git] y [Docker][docker]. El código del [repositorio][repo-master] ilustra cómo usar diferentes técnicas (*Dockerfile*, *docker-compose*) para levantar el entorno de forma automatizada.

<hr>

## ¿Qué motor de BBDD usaremos?

Para hacer los primeros experimentos nos vale [SQLite][sqlite], para entornos productivos cualquiera de las otras tres opciones: [Oracle][oracle], [MySQL][mysql] o [**PostgreSQL**][postgres].

Como administrador de BBDD esta última me parece la solución opensource mas óptima.

<hr>

## El `settings.py` y nuestro `.env`

El [`settings.py`][settings] contiene configuración global de nuestro proyecto, podríamos verlo como el *profile* que se carga  antes de ejecutar un script *bash*, es **absolutamente crítico**.

La regla fundamental es *ocultar* el valor de las variables que escondan secretos, una de las formas de hacerlo es esconderlas en un archivo `.env` que, por supuesto, **NO DEBE ESTAR VERSIONADO**.

Para acceder estos valores me ha sido realmente útil el módulo [Decouple][decouple].

```python
from decouple import config
SECRET_KEY = config('SECRET_KEY')
```

Otra punto a tener en cuenta es que Django dispone de un [módulo][settings-values] para acceder a las variables fijadas en settings por lo que este fichero puede ser un buen lugar para setear ciertos valores a los que deseemos acceder desde cualquier punto del proyecto.

Como último consejo añadir que la librería [DJ Database URL][dj-database-url] nos permite usar la configuración en *url* de nuestra BD.

`.env`

```bash
DB_URL=postgresql://postgres:postgres@postgres:5432/postgres
```

`settings.py`

```python
DATABASES = {
    'default': dj_database_url.config(default=config('DB_URL')),
    }
```

<hr>

## La estructura del proyecto o segundo error

La [documentación oficial][django-structure] propone la siguiente estructura a modo de ejemplo:

```
mysite/
    manage.py
    mysite/
        __init__.py
        settings.py
        urls.py
        wsgi.py
    polls/
        __init__.py
        admin.py
        migrations/
            __init__.py
            0001_initial.py
        models.py
        static/
            polls/
                images/
                    background.gif
                style.css
        templates/
            polls/
                detail.html
                index.html
                results.html
        tests.py
        urls.py
        views.py
    templates/
        admin/
            base_site.html
```

Por lo que el proyecto (a partir de ahora **sysgate**) empezó con dos apps:

 - Gestión de la autenticación (usuarios y grupos)
 - La aplicación en si (llamémosla *metrics*)

```
sysgate/
    manage.py
    sysgate/
        __init__.py
        settings.py
        urls.py
        wsgi.py
    account/
        __init__.py
        …
    metrics/
        __init__.py
        …
    templates/
        account/
            login.html
        metrics/
            home.html
```

Todo funcionaba de acuerdo a lo previsto pero se me planteo un problema a la hora de añadir una nueva aplicación *manager*.

Mi objetivo era el siguiente: Deseaba que el proyecto tuviera un *home común* donde se mostraran las aplicaciones disponibles al usuario en función de sus permisos.

**¿Dónde incluyo este home?**

- ¿En *account*?: no, se trata de separar funcionalidades y esta aplicación tiene un propósito único especifico que es la gestión de usuarios.

- ¿En *metrics*?: no way, esto nos obligaría a duplicar el código en *manager*, o lo que es peor usar una vista de metrics.

Finalmente opte por una de las soluciones más extendidas y que se basa en la creación de una aplicación *core*, donde agrupar precisamente todas las funcionalidades comunes que puedan ser llamadas desde cualquier otra aplicación, quedando la estructura así:

```
sysgate/
    manage.py
    sysgate/
        __init__.py
        settings.py
        urls.py
        wsgi.py
        routers.py
        account/
            __init__.py
            ...
        apps/
            __init__.py
            core/
                templatetags/
                    __init__.py
                    core_tags.py
                templates/
                    core/
                        home.html
                        ...
                __init__.py
                ...
            metrics/
                templates/
                    metrics/
                        home.html
                        ...
                __init__.py
                ...
            manager/
                templates/
                    manager/
                        home.html
                        ...
                __init__.py
                ...
        static/
            js/
                metrics.js
                ...
            css/
                metrics.css
                ...
        fixtures/
            metrics.json
            ...
        templates/
            account/
                login.html
                ...
            base.html
            ...
```

La vista *home*, se renderizará a partir de la plantilla core *¿make sense?*

De esta forma puedo añadir aplicaciones de forma indefinida aprovechando la infraestructura común expuesta por *ACCOUNT* y *CORE*.

Igualmente me permite trabajar independientemente en cualquier otra aplicación sin temor a romper nada, si no tocamos el código compartido todo seguirá funcionando lo que facilita el desarrollo paralelo a múltiples miembros del equipo.

:yum: **PRODUCTIVITY BOOST** :yum:

<hr>

## El enrutamiento

Es uno de mis mecanismos favoritos del Framework y se basa en expresiones regulares.

Cuando llamamos a una url, Django en primer lugar cargar el urls.py de la raíz del proyecto y a partir de ahí tratara de resolverla.

```python
urlpatterns = [
    url(r'^', include('apps.core.urls', namespace='core')),
    url(r'^account/', include('account.urls'), name='Autenticación'),
    url(r'^metrics/', include('apps.metrics.urls'), name='Métricas'),
    url(r'^manager/', include('apps.manager.urls'), name='Manager'),
    url(r'^admin/', admin.site.urls),
]
```

#### ¿Que pasa si introducimos nuestro dominio a secas?

Django recorrerá las cinco regex que tenemos parametrizadas, determinara que la que única que cumple el patrón  es la primera, donde le indicamos que cargue las *urls* de la aplicación *core*:

```python
urlpatterns = [
    url(r'^$', views.Home.as_view(), name='home'),
]
```

En este caso solo tenemos un patrón contemplado la cadena vacia, que enrutaremos hacia la vista `home` del proyect.

:bulb: **TIP** :bulb: La idea aquí es ser lo más modular posible, cada app gestionara sus propias *urls* intentando que la del proyecto raíz quede lo más simple y limpia posible.

<hr>

## Modelos y vistas ...¿Dónde está el controlador?

Para comprender Django es fundamental entender como funciona su [MVC][django-mvc]:

> Django parece ser un framework MVC, pero ustedes llaman al Controlador «vista», y a la Vista «plantilla». ¿Cómo es que no usan los nombres estándares?
>
> En nuestra interpretación de MVC, la «vista» describe el dato que es presentado al usuario. No es necesariamente cómo se ve el dato, sino qué dato se muestra. La vista describe cuál dato ve, no cómo lo ve. Es una distinción sutil.
>
> Entonces ¿donde entra el «controlador»? En el caso de Django, es probable que en el mismo framework: la maquinaria que envía una petición a la vista apropiada, de acuerdo a la configuración de URL de Django.
>
> Si busca acrónimos, usted podría decir que Django es un framework «MTV», esto es «modelo», «plantilla» y «vista». Ese desglose tiene mucho más sentido.

En la práctica nos basta con saber:

1. Diseñar los modelos.
2. Programar las vistas y su template.

Un modelo lo podríamos definir como una clase cuyos objetos instanciados están dotados de persistencia en BD.

Un modelo se traduce en una tabla cuyos campos se mapean con las variables de la clase modelo.

Diseñarlos requiere pensar detenidamente en los datos que se desean representar.

Para ahorrarnos curro, podemos tomar como ejemplo el modelo [User][user].

Los modelos además pueden relacionarse entre si, mediante campos / variables comunes (por ejemplo mediante [ForeignKey][foreignkey]).

No necesitamos sql, ya que nos proporcionan una API query pythonica

:bulb: **TIP** :bulb: Por defecto Django crea las tablas mediante las migrations, pero podemos establecer que el modelo no sea manejado y de esa forma podremos incorporar una tabla preexistente al ecosistema, es una buena forma de integrar datos provenientes de aplicaciones externas, y se nos proporcionan utilidades para inspeccionar el modelo.

:bulb: **TIP** :bulb: Campo único, django se lleva mal con los modelos formados por pks en multiples campos.

Las vistas, por norma general, se ocupan de representar ciertos datos provenientes de nuestros modelos.

```python
class Home(View):
    def get(self, request, *args, **kwargs):
        metricas = Metrica.objects.all().order_by('tipo')
        return render(request,
                      'metrics/home.html',
                      context={'metricas': metricas})
```

:bulb: **TIP** :bulb: Usar Class Based Views … pueden parecer mas complicadas pero a largo plazo compensa su aprendizaje por la potencia que ofrecen

Y finalmente el template devuelve el `html`.

{% raw %}
```html
<h1>Metrics App</h1>
<p>Bienvenid@ <strong>{{ request.user }}</strong></p>
<p>Disponibles:</p>
<table style="border: 1px solid black;">
{% for metrica in metricas %}
  <tr>
    <td>{{ metrica.nombre }}</td>
    <td>{{ metrica.get_tipo_display }}</td>
  </tr>
{% endfor %}
</table>
```
{% endraw %}

<hr>

## Autenticación y registro

En la estructura del proyecto ya hemos adelantado que lo ideal es mantener la funcionalidad común fuera del scope de las aplicaciones aislables, por lo que la primera de medida debe ser sacar la gestión de usuarios, en el ejemplo que estamos tratando se situa incluso en diferente nivel.

:bulb: **TIP** :bulb: **fundamental** es *SIEMPRE* heredar del [Abstractuser][abstractuser], nos permitirá olvidarnos del modelo `Profile`. Esto debe hacerse siempre en la creación del proyecto ya que la migración de el modelo de `User` es muy complicada.

:bulb: **TIP** :bulb: usar las auth views que ya nos viene de serie y permiten avanzar rápido en el proyecto sin perjuicio de una customización posterior.

<hr>

## Django Rest Framework (o GraphQL)

Otra de mis grandes *pifias*.

Mi primera aplicación se basaba en extraer un contexto en el `views.py` y enviársela al `template` para que renderizara los resultados.

Para páginas sencillas el resultado era el esperado pero la cosa se complica si tienes que representar miles de datos.

La plantilla se convierte en un infierno de llaves y todo tiende a romperse con facilidad.

*There must be a better way*.

:bulb: **TIP** :bulb: **separa el Backend (API) del Frontend**.

La idea es simple, nuestro proyecto expondrá una `API`, en nuestro caso [`REST`][rest-api] que será invocada directamente desde nuestras plantillas.

Al igual que ocurre con la separación de aplicaciones en la estructuración del proyecto, si separamos el *Backend* del *Frontend* podremos dividir el trabajo de forma más eficiente, además la API podrá ser integrada por otras aplicaciones por ejemplo móviles.

La tecnología *trending* es [GraphQL][graphql] y sin duda es el futuro, pero [DRF][drf-home] es sin duda el mejor añadido a Django, y para ciertas querys resulta de difícil substitución, un caso claro es el de las métricas donde necesitamos todos los campos y de forma secuencial.

<hr>

## Frontend

Los *templates* son base del Frontend y la tercera pata del [*MVT*][django-mvc], es donde vamos a escribir nuestro `HTML` (`JS` y demás).

Es un sistema diseñado de forma muy inteligente, teniendo en cuenta que Django no esta enfocado a la construcción de [*single-page applications*][spa-wiki].

Cada *app* tendrá sus propios *templates* encargados de renderizar los datos proporcionados por la vista que invoca o extraídos directamente desde la `API` (si la hemos usado claro).

Se sitúan en la ruta `template/app` que debe colgar de la carpeta donde esté el código de la aplicación.

Las plantillas del proyecto raíz colgarán del primer nivel.

Ejemplo en *sysgate*:

```
├── account
├── apps
│   ├── core
│   │   ├── templates
│   │   │   └── core
│   │   │       └── home.html
│   │   ├── templatetags
│   │   │   ├── __init__.py
│   │   │   └── core_tags.py
│   └── metrics
│       ├── templates
│       │   └── metrics
│       │       └── home.html
├── templates
│   ├── 404.html
│   ├── account
│   │   ├── login.html
│   │   ├── profile.html
│   │   └── registro.html
│   ├── base.html
│   └── base_generic.html
```

Para el neófito es vital entender tres conceptos:

1. La herencia
3. Lenguaje
2. Custom *Tags* y *Filtros*

La **herencia** nos permite extender (y modificar) la plantilla base que contendrá los elementos básicos y el look and feel de nuestra web, facilitándonos enormemente la tarea de construir una nueva pagina ya que los elementos comunes ya nos vienen dados ([*DRY*][dry-wiki]).

Podríamos verlo, construyendo un paralelismo, como la [*herencia*][python-inheritance] de de una *clase* Python, podemos heredar todo el código de la clase base, pero también podemos ampliarlo y modificarlo

Respecto al *lenguaje* poco que mencionar, *Django* proporciona su propia [*sintaxis*][template-language] y nos permite expresarnos en el *template* de modo programático proporcionándonos una enorme versatilidad.

Los **tags** y **filtros** nos permiten incorporar código Python y usarlo en nuestras plantillas.

Para *visualizar* estos conceptos, supongamos este `tag`:

```python
from django import template

register = template.Library()

@register.filter('en_grupo')
def en_grupo(user, group_name):
    groups = user.groups.all().values_list('name', flat=True)
    return True if group_name in groups else False
```

Y el `template` que lo *renderiza*:

```python
{% raw %}
{% extends 'base.html' %}

{% load core_tags %}

{% block content %}

<div class="container">
  {% if request.user|en_grupo:"pycones" or user.is_superuser %}
    <div class="jumbotron container-fluid">
      <h1>Metrics</h1>
      <a class="btn" href="{% url 'metrics:home' %}">Acceder</a>
    </div>
  {% endif %}

  {% if not user.is_authenticated %}
    <div class="alert" align="center" role="alert">
      <button type="button" class="close"></button>
      Login para acceder
    </div>
  {% endif %}
</div>

{% endblock %}
{% endraw %}
```

<hr>

## Mi web no es *cool* o tercer error

Andaba yo muy contento con los resultados del trabajo hasta la primera reunión de seguimiento con los usuarios donde la primera pregunta fué, … **¿y esto cómo se ve en el móvil?**

*¿OLA K ASE?*

No se me había pasado por la cabeza esa posibildad: **se veía mal, muy mal**.

La cruda (o no) realidad es esta: *Si deseas que tu web tenga un interfaz de usuario moderno tendrás que ir mas allá de Python y un HTML básico*.

Yo me he apoyado fundamentalmente en estos dos elementos:

- Bootstrap
- Javascript

[**Bootstrap**][bootstrap-wiki] es un *framework* web basado principalmente en HTML y CSS que mejora (o hace más vistosos) los elementos que incluimos en nuestros *templates*, además permite que nuestras paginas sean responsive sin demasiada dificultad (Sistema en grid)

[**Javascript**][js-mozilla] es un lenguaje que tiene la gran ventaja de poder ejecutarse en el navegador. Existen innumerables librerías `.js` muchas de ellas *super cool*, que nos permiten hacer verdaderas viguerías con los datos devueltos por nuestro *Backend*, no debemos convertirnos en talibanes del lenguaje, *JS* es una utilidad imprescindible en el Frontend.

Ejemplo:

```python
{% raw %}
{% extends 'base.html' %}

{% load static %}

{% block extrajs %}
<script src="https://cdnjs.cloudflare.com/ajax/libs/vue/2.4.2/vue.min.js"></script>
{% endblock %}

{% block content %}

<div id="container" class="container">
  <table>
    <thead>
      <tr>
        <th>Metric</th>
        <th>Tipo</th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="metrica in metricas">
        <td>[[ metrica.nombre ]]</td>
        <td>[[ metrica.long_tipo ]]</td>
      </tr>
    </tbody>
  </table>
</div>

<script>
  var v = new Vue({
    delimiters: ['[[', ']]'],
    el: '#container',
    data: {
      metricas: []
    },
    mounted: function(){
      $.get('/metrics/api/v1/metricas/', function(data){
        v.metricas = data;
      })}
  })
</script>
{% endblock content %}
{% endraw %}
```

<hr>

## Gestión de permisos

Otro de los temas básicos en una aplicación en producción, la infraestructura es común pero los usuarios no deben percibirlo salvo en el look and feel de la pagina, uno solo debe ver lo que debe ver.

:bulb: **TIP** :bulb: Django tiene un control muy fino de permisos por usuario, pero podemos simplificar mucho la gestión usando los grupos como si fueran roles.

<hr>

[pycones2017-home]: https://2017.es.pycon.org "PyConES 2017 - Cáceres"
[dvs-agenda]: https://2017.es.pycon.org/es/schedule/sysadmin-vs-django/ "Django vs Sysadmin - PyConES 2017"
[dvs-slides]: https://klashxx.github.io/slides/django/ "Django vs Sysadmin - Slides"
[github]: https://github.com "GitHub"
[asciinema]: https://asciinema.org/ "asciinema"
[git-download]: https://git-scm.com/downloads "git - Descarga"
[docker-install]: https://docs.docker.com/engine/installation/ "Docker - Instalación"
[docker-compose-install]: https://docs.docker.com/compose/install/ "Docker Compose - Instalación"
[git-branch]: https://git-scm.com/book/es/v1/Ramificaciones-en-Git-%C2%BFQu%C3%A9-es-una-rama%3F "¿Qué es una rama?"
[repo-master]: https://github.com/klashxx/PyConES2017/tree/master "Django vs Sysadmin - master"
[repo-auth]: https://github.com/klashxx/PyConES2017/tree/01_auth "Django vs Sysadmin - 01_auth"
[repo-apps]: https://github.com/klashxx/PyConES2017/tree/02_apps "Django vs Sysadmin - 02_apps"
[repo-drf]: https://github.com/klashxx/PyConES2017/tree/03_drf "Django vs Sysadmin - 03_drf"
[wiki-orm]: https://es.wikipedia.org/wiki/Mapeo_objeto-relacional "ORM - Wikipedia"
[drf-home]: http://www.django-rest-framework.org/ "Django REST framework"
[admin-site]: https://docs.djangoproject.com/en/dec/ref/contrib/admin/ "Admin Site"
[djangopackages]: https://djangopackages.org/categories/apps/?sort=repo_watchers&dir=desc "Django Packages"
[csm-wiki]: https://es.wikipedia.org/wiki/Sistema_de_gesti%C3%B3n_de_contenidos "Content Management System"
[awesome]: https://gitlab.com/rosarior/awesome-django "Awesome Django"
[django-girls-tuto]: https://tutorial.djangogirls.org/es/ "Django Girls"
[mozilla-tuto]: https://developer.mozilla.org/en-US/docs/Learn/Server-side/Django "Tutorial Mozilla"
[virtualenv]: https://virtualenv.pypa.io/en/stable/# "Virtualenv"
[git]: https://git-scm.com/ "Git"
[docker]: https://docs.docker.com/ "Docker"
[sqlite]: https://www.sqlite.org/ "SQLite"
[oracle]: http://www.oracle.com/technetwork/database/index.html "Oracle"
[mysql]: https://www.mysql.com/ "MySQL"
[postgres]: https://www.postgresql.org/ "PostgreSQL"
[django-structure]: https://docs.djangoproject.com/es/1.11/intro/reusable-apps/#your-project-and-your-reusable-app "Estructura Django propuesta"
[settings]: https://docs.djangoproject.com/ko/1.11/ref/settings/ "settings.py"
[decouple]: https://github.com/henriquebastos/python-decouple/ "Decouple"
[settings-values]: https://docs.djangoproject.com/en/1.11/topics/settings/#using-settings-in-python-code "Settings values"
[dj-database-url]: https://github.com/kennethreitz/dj-database-url/ "DJ-Database-URL"
[django-mvc]: https://docs.djangoproject.com/es/1.11/faq/general/#django-appears-to-be-a-mvc-framework-but-you-call-the-controller-the-view-and-the-view-the-template-how-come-you-don-t-use-the-standard-names "Django MVC"
[user]: https://docs.djangoproject.com/en/dev/ref/contrib/auth/#django.contrib.auth.models.User) "Django Modelo de Usuario"
[foreignkey]: https://docs.djangoproject.com/en/dev/ref/models/fields/#django.db.models.ForeignKey "ForeignKey"
[abstractuser]: https://docs.djangoproject.com/en/dev/topics/auth/customizing/#using-a-custom-user-model-when-starting-a-project "Abstractuser"
[rest-api]: https://es.wikipedia.org/wiki/Transferencia_de_Estado_Representacional "REST API"
[graphql]: http://graphql.org/ "GraphQL"
[spa-wiki]: https://es.wikipedia.org/wiki/Single-page_application "Single-page application"
[dry-wiki]: https://en.wikipedia.org/wiki/Don%27t_repeat_yourself "Don't repeat yourself"
[template-extends]: https://docs.djangoproject.com/en/dev/ref/templates/builtins/#std:templatetag-extends "Template extends"
[python-inheritance]: https://docs.python.org/3.6/tutorial/classes.html#inheritance "Python Inheritance"
[template-language]: https://docs.djangoproject.com/en/dev/ref/templates/language/ "The Django template language"
[bootstrap-wiki]: https://es.wikipedia.org/wiki/Bootstrap_(framework) "Bootstrap Wikipedia"
[js-mozilla]: https://developer.mozilla.org/es/docs/Web/JavaScript "Javascript Mozilla Tutorial"
