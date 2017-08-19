---
layout: post
title: Django vs Sysadmin
permalink: django-vs-sysadmin
comments: true
---

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

:warning: **ATENCION**:  Esto **NO** es un tutorial.

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

