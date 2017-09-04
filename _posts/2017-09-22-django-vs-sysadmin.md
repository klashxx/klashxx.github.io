---
layout: post
title: Django vs Sysadmin
permalink: django-vs-sysadmin
comments: true
---

[![License: CC BY-NC-SA 4.0](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc-sa/4.0/)

## Prefacio

Texto base sobre el que versará la charla [Djando vs Sysadmin][dvs-agenda] a impartir en Cáceres durante la [PyConES 2017][pycones2017-home].

Podéis acceder a las *slices* [aquí][dvs-slides].

### :warning: Disclaimer :warning:

Para ilustrar este `post` se ha creado un aplicación *demo* cuyo código reside en este [repo][repo-master] de [Github][github] y se ha estructurado en [ramas][git-branch].

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

Me llamo Juan Diego, soy Sysdamin y hasta hace 5 meses no sabía que era `HTML` :grimacing:.

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

- [Django Rest Framework][drf-home] (lo comentaremos más adelante)

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
[csm-wiki]: https://es.wikipedia.org/wiki/Sistema_de_gesti%C3%B3n_de_contenidos "Content Management System"
[awesome]: https://gitlab.com/rosarior/awesome-django "Awesome Django"
[django-girls-tuto]: https://tutorial.djangogirls.org/es/ "Django Girls"
[mozilla-tuto]: https://developer.mozilla.org/en-US/docs/Learn/Server-side/Django "Tutorial Mozilla"
[virtualenv]: https://virtualenv.pypa.io/en/stable/# "Virtualenv"
[git]: https://git-scm.com/ "Git"
[docker]: https://docs.docker.com/ "Docker"
