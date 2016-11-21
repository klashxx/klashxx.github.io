---
layout: post
title: awk, power para tu command line
permalink: awk-power-para-tu-cmd
image: /images/awk_flow.png
comments: true
---

### Guía de uso de la navaja suiza de los *nix para principiantes

<hr>
English version [here][english].
<hr>

## :warning: Disclaimer :warning:

Este texto se refiere exclusivamente a la implementación **GNU** de `awk` conocida como `gawk` que es la usada mayoritariamente en cualquier distribución **Linux** actual.

En su redacción la referencia fundamental ha sido [The GNU Awk User’s Guide][gnu-awk] y mis aportaciones a [Stackoverflow][so].

<hr>

> AWK es un lenguaje similar a PERL, pero considerablemente más elegante.
>
> - Arnold Robbins


#### *¿Qué es?*

`AWK` es un lenguaje de programación diseñado para procesamiento de texto, se usa normalmente como herramienta de extracción y reporting.

Es un standard en prácticamente cualquier sistema *nix.


#### ``awk``award name ...

El nombre del lenguaje se deriva del apellido de sus autores: Alfred **A**ho, Peter **W**einberger, y Brian **K**ernighan.


### Así que ``awk`` ...

* Busca líneas que contengan determinados patrones en ficheros o en la entrada estándar.

* Se usa fundamentalmente para reporting y extracción de datos, por ejemplo para *sumarizar* la salida de otros programas.

* Tiene una sintaxis similar a `C`.

* Orientado a los datos: se trata de describir con que datos queremos trabajar y que acción hacer al encontrarlos.


```` shell
pattern { action }
pattern { action }
````

<hr>

### Lo básico

#### ¿Cómo ejecutarlo?

Si el programa es **corto**:

```` shell
awk 'program' input-file1 input-file2
````

:warning: **Nota:** presta atención a los posibles problemas con el *shell quoting*[^1].

```` shell
cmd | awk 'program'
````

El ``pipe`` redirige  la salida del comando a la izquierda `cmd` a la *entrada* del comando `awk`[^2].

Cuando el código es **largo**, normalmente es preferible usar un fichero como contendor y ejecutarlo de esta forma:

```` shell
awk -f program-file input-file1 input-file2

cmd | awk -f program-file
````

O a través de un interprete, a lo `shebang`.

```` shell
#!/bin/awk -f

BEGIN { print "hello world!!" }
````

#### Otros parámetros interesantes

`-F fs` Establece el valor de `FS` (*Field Separator*) `fs`.

`-v var=val` Pasa la variable `var` con valor `val` al `awk` antes de que la ejecución comience.

*:point_right: Nota*: se puede usar múltiples veces, para *setear* el número deseado de valores.

#### BEGIN y END

Estas etiquetas marcan los bloques o patrones especiales que proporcionan un espacio para las acciones de *inicialización* y *limpieza*.

Sigue el siguiente esquema:

```` shell
BEGIN{
    // initialize variables
}
{
    /pattern/ { action }
}
END{
    // cleanup
}
````

Ambos bloques se ejecutan solo una vez, `BEGIN` antes de que se lea el primer registro, `END` después de consumir todo el *input*.

```` shell
$ echo "hello"| awk 'BEGIN{print "BEGIN";f=1}
                     {print $f}
                     END{print "END"}'
BEGIN
hello
END
````

<hr>

#### Porque *grepear* si tenemos `awk`?

```` shell
$ cat lorem.dat
Lorem ipsum dolor sit amet, consectetur adipiscing elit.
Maecenas pellentesque erat vel tortor consectetur condimentum.
Nunc enim orci, euismod id nisi eget, interdum cursus ex.
Curabitur a dapibus tellus.
Lorem ipsum dolor sit amet, consectetur adipiscing elit.
Aliquam interdum mauris volutpat nisl placerat, et facilisis.
````

```` shell
$ grep dolor lorem.dat
Lorem ipsum dolor sit amet, consectetur adipiscing elit.
Lorem ipsum dolor sit amet, consectetur adipiscing elit.
````

```` shell
$ awk '/dolor/' lorem.dat
Lorem ipsum dolor sit amet, consectetur adipiscing elit.
Lorem ipsum dolor sit amet, consectetur adipiscing elit.
````

:warning: **Nota:** Cuando no se proporciona una acción el comportamiento por defecto es volcar la línea o registro *macheado* al _**stdout**_.

Pero ... ¿como podríamos mostrar la *primera* y *ultima* palabra de cada línea?

Se puede hacer con `grep`, por supuesto, pero necesitamos dos paso:

```` shell
$ grep -Eo '^[^ ]+' lorem.dat
Lorem
Maecenas
Nunc
Curabitur
Lorem
Aliquam
````

```` shell
$ grep -Eo '[^ ]+$' lorem.dat
elit.
condimentum.
ex.
tellus.
elit.
ultrices.
````

Veamos `awk` en acción solventado este problema:

```` shell
$ awk '{print $1,$NF}' lorem.dat
Lorem elit.
Maecenas condimentum.
Nunc ex.
Curabitur tellus.
Lorem elit.
Aliquam ultrices.
````

### ¿Mejor no :sunglasses:?  , Yeah but ... ¿Como funciona?

Para averiguarlo es necesario conocer dos estructuras básicas en `awk` los _**Registros**_ (*Records*) y _**Campos**_ (*Fields*) en los que se dividen cualquier entrada al programa.

<hr>

#### Registros

Los *registros* están delimitados por un carácter o expresión regular que se conoce como *Record Separator* `RS`.

Su valor **por defecto** es el salto de línea unix `\n`, por este motivo los registros por defecto equivalen a una línea individual.

Adicionalmente disponemos de la variable `ORS`  *Output Record Separator* que nos va a permitir controlar la delimitación de estos registros al volcarlos sobre el *stdout*.

Tanto el `RS` como el `ORS` deben estar *encomillados*, lo que indica que estamos ante constantes literales.

Usar un carácter o `regex` distinto es tan simple como asignárselo a las variable `RS` u `ORS`:

- Generalmente, el mejor momento para hacerlo es al comienzo de la ejecución, en el `BEGIN`, antes de que comience el proceso de la entrada de modo que el primer registro se lea con el separador deseado.

- La otra forma de cambiar el `RS` (o el `ORS`) es a través de la línea de comando mediante los mecanismos de asignación de variables.

Ejemplos:

```` shell
$ awk 'BEGIN{RS=" *, *";ORS="<<<---\n"}{print $0}' lorem.dat
Lorem ipsum dolor sit amet<<<---
consectetur adipiscing elit.
Maecenas pellentesque erat vel tortor consectetur condimentum.
Nunc enim orci<<<---
euismod id nisi eget<<<---
interdum cursus ex.
Curabitur a dapibus tellus.
Lorem ipsum dolor sit amet<<<---
consectetur adipiscing elit.
Aliquam interdum mauris volutpat nisl placerat<<<---
et facilisis neque ultrices.
<<<---
````

```` shell
$ awk '{print $0}' RS=" *, *" ORS="<<<---\n" lorem.dat
Lorem ipsum dolor sit amet<<<---
consectetur adipiscing elit.
Maecenas pellentesque erat vel tortor consectetur condimentum.
Nunc enim orci<<<---
euismod id nisi eget<<<---
interdum cursus ex.
Curabitur a dapibus tellus.
Lorem ipsum dolor sit amet<<<---
consectetur adipiscing elit.
Aliquam interdum mauris volutpat nisl placerat<<<---
et facilisis neque ultrices.
<<<---
````

#### Campos

Los registros `awk` se dividen de forma automática en pedazos denominados *campos* (*fields*).

El separador se contiene en la variable `FS` (*Field Separator*) su valor **por defecto** entre *campos* **es el espacio blanco** que en `awk` se define como una cadena compuesta por uno o mas *espacios*, *TABs* o *saltos de línea*.

Nos referimos a un *campo* en `awk` mediante el símbolo dólar `$` seguido por el número del campo que deseamos tratar.

De esta forma `$1` se referirá al primer campo, `$2` al segundo y así sucesivamente.

**IMPORTANTE**: `$0` hace referencia al *registro completo*.

```` shell
$ awk '{print $3}' lorem.dat
dolor
erat
orci,
dapibus
dolor
mauris
````

`NF` es una variable *predefinida* que devuelve el *número de campos* de el registro actual.

En la práctica esto trae como consecuencia que `$NF` siempre apuntará al último campo de un registro.

```` shell
$ awk '{print NF, $NF}' lorem.dat
8 elit.
7 condimentum.
10 ex.
4 tellus.
8 elit.
10 ultrices.
````

Del mismo modo que existe un `ORS` también disponemos de un `OFS` (*Output Field Separator*) para controlar la forma en que delimitaremos los campos al mandarlos al *output stream*.

```` shell
$ cat /etc/group
nobody:*:-2:
nogroup:*:-1:
wheel:*:0:root
daemon:*:1:root
kmem:*:2:root
sys:*:3:root
tty:*:4:root
````

```` shell
$ awk '!/^(_|#)/&&$1=$1' FS=":" OFS="<->" /etc/group
nobody<->*<->-2<->
nogroup<->*<->-1<->
wheel<->*<->0<->root
daemon<->*<->1<->root
kmem<->*<->2<->root
sys<->*<->3<->root
tty<->*<->4<->root
````

:warning: **Nota**: ¿¿Y ese?? ... `$1=$1`[^3]

<hr>

Una vez que interiorizamos *registros* y *campos* resulta muy sencillo entender nuestro código inicial:

```` shell
$ awk '{print $1,$NF}' lorem.dat
Lorem elit.
Maecenas condimentum.
Nunc ex.
Curabitur tellus.
Lorem elit.
Aliquam ultrices.
````

#### NR y FNR

Estas son dos variables *built-in* muy interesantes:

`NR` : Número de registros que `awk` ha procesado desde el inicio de la ejecución del programa.

`FNR` : Número de registro del fichero procesado actualmente, `awk` *resetea* `FNR` a cero cada vez que comienza a procesar un nuevo archivo.

```` shell
$ cat n1.dat
one
two
````

```` shell
$ cat n2.dat
three
four
````

```` shell
$ awk '{print NR,FNR,$0}' n1.dat n2.dat
1 1 one
2 2 two
3 1 three
4 2 four
````

<hr>

#### Mejorando la salida

Para conseguir una salida más adecuada podemos usar `printf`:

`printf format, item1, item2, …`

La mascara de formato es muy similar a la del ISO C.

```` shell
$ awk '{printf "%20s <-> %s\n",$1,$NF}' lorem.dat
               Lorem <-> elit.
            Maecenas <-> condimentum.
                Nunc <-> ex.
           Curabitur <-> tellus.
               Lorem <-> elit.
             Aliquam <-> ultrices.
````

#### Redirigiendo el Output

La salida de `print` y  `printf` se dirige por defecto al `stdout` pero podemos redireccionarla de diferentes modos.

Estas redirecciones en `awk` se escriben de forma similar a como se hacen en los comandos sobre `shell`, con la salvedad de que se incorporan en el código del programa.

```` shell
$ awk 'BEGIN{print "hello">"hello.dat"}'
````

```` shell
$ awk 'BEGIN{print "world!">>"hello.dat"}'
````

```` shell
$ cat hello.dat
hello
world!
````

Otra posibilidad es enviar la salida a otro programa mediante *pipes*:

```` shell
$ awk 'BEGIN{sh="/bin/sh";print "date"|sh;close(sh)}'
dom nov 13 18:36:25 CET 2016
````

Podemos apuntar estas redirecciones a los *streams* estándar:

- `/dev/stdin`: La entrada estándar (descriptor 0).

- `/dev/stdout`: La salida estándar (descriptor 1).

- `/dev/stder`: La salida de error estándar (descriptor 2).

Un ejemplo de como escribir mensajes de error sería:

```` shell
$ awk 'BEGIN{print "Serious error detected!" > "/dev/stderr"}'
Serious error detected!
````

<hr>

#### Trabajando con Arrays

En `awk` los  *arrays* son **asociativos** lo que en la practica se traduce en que cada array es una colección de parejas índice - valor , siendo el índice cualquier valor numérico o cadena de texto, donde el orden es irrelevante.

No necesitan de declaración previa y se pueden añadir nuevos valores en cualquier momento.

Índice | Valor
-------|---------
"perro" | "dog"
"gato"  | "cat"
"uno" | "one"
1  | "one"
2  | "two"

Para referirnos a un array usaremos la sintaxis:

`array[index-expression]`

Si queremos asignarle valores:

`array[index-expression] = value`

Para determinar si un índice está presente:

`indx in array`

Para recorrer los elementos del *array*:

```` shell
for (var in array) {
    var, array[var]
    }
````

Si hemos usado valores numéricos podemos recuperar los elementos preservando el orden:

```` shell
for (i = 1; i <= max_index; i++) {
    print array[i]
    }
````

O usar algo más avanzado (exclusivo de `gawk`) como `@ind_str_asc`.

Por ejemplo, partiendo de:

```` shell
$ cat dict.dat
uno one
dos two
tres three
cuatro four
````

```` shell
awk '{dict[$1]=$2}
     END{if ("uno" in dict)
           print "Yes we have uno in dict!"
         if (!("cinco" in dict))
           print "No , cinco is not in dict!"
         for (esp in dict){
            print esp, "->" ,dict[esp]
            }
     }'  dict.dat
````

Devolvería:

```` shell
Yes we have uno in dict!
No , cinco is not in dict!
uno -> one
dos -> two
tres -> three
cuatro -> four
````

Podemos ver como mantiene el orden original:

```` shell
awk 'BEGIN{
      a[4]="four"
      a[1]="one"
      a[3]="three"
      a[2]="two"
      a[0]="zero"
      exit
      }
      END{for (idx in a){
             print idx, a[idx]
             }
      }'
````

```` shell
4 four
0 zero
1 one
2 two
3 three
````

La ordenación podemos controlarla mediante va variable `PROCINFO`:

```` shell
awk 'BEGIN{
      PROCINFO["sorted_in"] = "@ind_num_asc"
      a[4]="four"
      a[1]="one"
      a[3]="three"
      a[2]="two"
      a[0]="zero"
      exit
      }
      END{for (idx in a){
             print idx, a[idx]
             }
      }'
````

```` shell
0 zero
1 one
2 two
3 three
4 four
````

<hr>

### Funciones Build-in

`gensub(regexp, replacement, how [, target])` : Es la función más avanzada de substitución de texto.

Y sus variantes más simples:

`gsub(regexp, replacement [, target])`

`sub(regexp, replacement [, target])`

Su uso más simple permite sencillas substituciones

Partiendo de:

```` shell
$ cat lorem.dat
Lorem ipsum dolor sit amet, consectetur adipiscing elit.
Maecenas pellentesque erat vel tortor consectetur condimentum.
Nunc enim orci, euismod id nisi eget, interdum cursus ex.
Curabitur a dapibus tellus.
Lorem ipsum dolor sit amet, consectetur adipiscing elit.
Aliquam interdum mauris volutpat nisl placerat, et facilisis.
````

Supongamos que deseamos *invertir* la posición de las palabras que se encuentran a un lado y otro de las comas.

```` shell
$ awk '{print gensub(/([^ ]+)( *, *)([^ ]+)/,
                     "\\3\\2\\1", "g")}' lorem.dat
Lorem ipsum dolor sit consectetur, amet adipiscing elit.
Maecenas pellentesque erat vel tortor consectetur condimentum.
Nunc enim euismod, orci id nisi interdum, eget cursus ex.
Curabitur a dapibus tellus.
Lorem ipsum dolor sit consectetur, amet adipiscing elit.
Aliquam interdum mauris volutpat nisl et, placerat facilisis.
````

Mediante `gensub` usamos *grupos* para realizar tres *capturas* he invertir los resultados.

Una acción más simple sería la substitución de *puntos* por *comas*:

```` shell
awk '$0=gensub(/\./, ",", "g")' lorem.dat
Lorem ipsum dolor sit amet, consectetur adipiscing elit,
Maecenas pellentesque erat vel tortor consectetur condimentum,
Nunc enim orci, euismod id nisi eget, interdum cursus ex,
Curabitur a dapibus tellus,
Lorem ipsum dolor sit amet, consectetur adipiscing elit,
Aliquam interdum mauris volutpat nisl placerat, et facilisis,
````

Aunque también podríamos optar por la función simplificada `gsub`:

```` shell
awk 'gsub(/\./, ",")' lorem.dat
Lorem ipsum dolor sit amet, consectetur adipiscing elit,
Maecenas pellentesque erat vel tortor consectetur condimentum,
Nunc enim orci, euismod id nisi eget, interdum cursus ex,
Curabitur a dapibus tellus,
Lorem ipsum dolor sit amet, consectetur adipiscing elit,
Aliquam interdum mauris volutpat nisl placerat, et facilisis,
````

Personalmente la considero mas adecuada cuando no es necesario la captura de grupos en `regex`.

Otras funciones de procesamiento de cadenas interesantes son `index` y `substr`.

`index(in, find)`

`substr(string, start [, length ])`

Su funcionamiento queda claro con estos sencillos ejemplos:

```` shell
$ awk 'BEGIN{t="hello-world";print index(t, "-")}'
6
````

```` shell
$ awk 'BEGIN{t="hello-world";print substr(t,index(t, "-")+1)}'
world
````

La función `split` permite generar un array a partir de una *cadena* y un *separador*, *retorna* el número de elementos del *vector* resultante:

`split(string, array [, fieldsep [, seps ] ])`

```` shell
$ cat passwd
jd001:x:1032:666:Javier Diaz:/home/jd001:/bin/rbash
ag002:x:8050:668:Alejandro Gonzalez:/home/ag002:/bin/rbash
jp003:x:1000:666:Jose Perez:/home/jp003:/bin/bash
ms004:x:8051:668:Maria Saenz:/home/ms004:/bin/rbash
rc005:x:6550:668:Rosa Camacho:/home/rc005:/bin/rbash
````

```` shell
$ awk 'n=split($0, a, ":"){print n, a[n]}' passwd
7 /bin/rbash
7 /bin/rbash
7 /bin/bash
7 /bin/rbash
7 /bin/rbash
````
:point_right: **Nota**: Esto se podría hacer de otra forma mucho más simple.

```` shell
$ awk '{print NF,$NF}' FS=':' passwd
7 /bin/rbash
7 /bin/rbash
7 /bin/bash
7 /bin/rbash
7 /bin/rbash
````

<hr>

### Funciones propias

Escribir funciones *custom* es muy simple tal y como se puede apreciar en el siguiente ejemplo:

```` shell
awk 'function test(m)
     {
        printf "This is a test func, parameter: %s\n", m
     }
     BEGIN{test("param")}'
````

Que nos devuelve:

```` shell
This is a test func, parameter: param
````

Análogamente podríamos devolver un valor mediante `return`:

```` shell
awk 'function test(m)
     {
        return sprintf("This is a test func, parameter: %s", m)
     }
     BEGIN{print test("param")}'
````

La única forma de que una variable sea local en una función es en la recogida de parámetros.

Los parámetros escalares se pasan por valor y los arrays por referencia así que cualquier cambio que se haga en el array en la función se reflejara en el cuerpo del programa:

```` shell
 awk 'function test(m)
      {
       m[0] = "new"
      }
      BEGIN{m[0]=1
            test(m)
            exit
      }
      END{print m[0]}'
````

Volcará al `stdout`:

```` shell
new
````

<hr>

# Ahora empecemos con lo divertido :godmode:

Trataremos de resolver una serie de ejercicios que pondrán a prueba los conocimientos adquiridos:

[01](#mostrar-la-penultima-palabra-de-un-fichero). Mostrar la penúltima palabra del fichero `lorem.dat`.

[02](#substituir-un-registro). Substituir una línea o registro.

[03](#colocar-un-punto-y-coma-al-final-de-cada-registro). Añadir un punto y coma al final de cada línea.

[04](#y-una-coma-entre-cada-palabra). ¿Y una coma entre cada palabra?

[05](#todo-junto). ¿Todo junto?

[06](#incluir-los-registros-impares-en-un-archivo-y-los-pares-en-otro). Incluir los registros impares en un archivo y los pares en otro.

[07](#dado-un-fichero-de-password-informa-el-campo-del-home-del-usuario-en-base-al-login). Dado un fichero de `/ect/password` informa el campo del `home` del usuario en base al login.

[08](#cambiar-el-orden-de-los-campos-de-modo-que-el-primero-pase-a-ser-el-final). Cambiar el orden de los campos de modo que el primero pase a ser el final.

[09](#hackeando-traceroute). Hackeando `traceroute`.

[10](#procesos-dependientes-de-un-pid-padre). Procesos dependientes de un `PID` *padre*.

[11](#como-agregar-datos-a-partir-de-una-clave). Como agregar información a partir de una clave.

[12](#mostrar-los-registros-entre-dos-patrones). Mostrar los registros entre dos patrones.

[13](#convertir-un-campo-en-base-a-su-contenido).  Convertir un campo en función de su contenido a un determinado valor numérico.

[14](#agrupando-registros-en-columnas). Agrupando registros en columnas.

[15](#procesando-un-fichero-fasta). Procesando un fichero *FASTA*.

[16](#reporting-complejo). Reporting complejo.

[17](#join-entre-ficheros). *Join* entre ficheros.

[18](#cruzando-passwd-y-group). Cruzando `/etc/passwd` y `/etc/group`.

[19](#conexiones-por-usuario-a-un-servidor). Conexiones por usuario a un servidor.

[20](#obteniendo-la-media-total-proporcionada-por-el-comando-uptime). Obteniendo la media total proporcionada por el comando `uptime`.

<hr>
:astonished: ¡*Markdown* no permite acentos en los *anchors*! :astonished:
<hr>

### 01. Mostrar la penultima palabra de un fichero

Partiendo del siguiente *source* file:

```` shell
$ cat lorem.dat
Lorem ipsum dolor sit amet, consectetur adipiscing elit.
Maecenas pellentesque erat vel tortor consectetur condimentum.
Nunc enim orci, euismod id nisi eget, interdum cursus ex.
Curabitur a dapibus tellus.
Lorem ipsum dolor sit amet, consectetur adipiscing elit.
Aliquam interdum mauris volutpat nisl placerat, et facilisis.
````

```` shell
$ awk '{print $(NF-1)}' lorem.dat
adipiscing
consectetur
cursus
dapibus
adipiscing
neque
````

No hay mucho que explicar `NF` indica el número de campos presente en el registro , luego `(NF-1)` apuntara al campo anterior al último y `$(NF-1)` a su correspondiente valor.

<hr>

### 02. Substituir un registro

Supongamos que necesitamos reemplazar la tercera línea por:

`Esto no es latín`

Nada más simple, basta con jugar con `NR` (*Number of Record*):

```` shell
$ awk 'NR==3{print "Esto no es latín";next}{print}' lorem.dat
Lorem ipsum dolor sit amet, consectetur adipiscing elit.
Maecenas pellentesque erat vel tortor consectetur condimentum.
Esto no es latín
Curabitur a dapibus tellus.
Lorem ipsum dolor sit amet, consectetur adipiscing elit.
Aliquam interdum mauris volutpat nisl placerat, et facilisis.
````

Alternativamente, podemos evitar usar `next` asignando el nuevo texto al registro completo `$0`:

```` shell
$ awk 'NR==3{$0="Esto no es latín"}1' lorem.dat
Lorem ipsum dolor sit amet, consectetur adipiscing elit.
Maecenas pellentesque erat vel tortor consectetur condimentum.
Esto no es latín
Curabitur a dapibus tellus.
Lorem ipsum dolor sit amet, consectetur adipiscing elit.
Aliquam interdum mauris volutpat nisl placerat, et facilisis.
````

<hr>

### 03. Colocar un punto y coma al final de cada registro

```` shell
$ awk '1' ORS=";\n" lorem.dat
Lorem ipsum dolor sit amet, consectetur adipiscing elit.;
Maecenas pellentesque erat vel tortor consectetur condimentum.;
Nunc enim orci, euismod id nisi eget, interdum cursus ex.;
Curabitur a dapibus tellus.;
Lorem ipsum dolor sit amet, consectetur adipiscing elit.;
Aliquam interdum mauris volutpat nisl placerat, et facilisis neque ultrices.;
````

La solución es simple, como conocemos que el `RS` por  defecto es el salto de línea basta con que al de salida `OFS` le  antepongamos el punto y coma `;\n`

:warning: **ATENCIÓN**: ¿Que hace ese `1`?[^4]

<hr>

### 04. ¿Y una coma entre cada palabra?

```` shell
$ awk '{$1=$1}1' OFS=',' lorem.dat
Lorem,ipsum,dolor,sit,amet,,consectetur,adipiscing,elit.
Maecenas,pellentesque,erat,vel,tortor,consectetur,condimentum.
Nunc,enim,orci,,euismod,id,nisi,eget,,interdum,cursus,ex.
Curabitur,a,dapibus,tellus.
Lorem,ipsum,dolor,sit,amet,,consectetur,adipiscing,elit.
Aliquam,interdum,mauris,volutpat,nisl,placerat,,et,facilisis,neque,ultrices.
````

Los más notable de este código es como se fuerza que `awk` reconstruya el *registro* completo, usando los valores actuales para el `FS` y `OFS`.

Para hacerlo usamos esta asignación inocua: `$1 = $1`

<hr>

### 05. ¿Todo junto?

```` shell
$ awk '{$1=$1}1' OFS=',' ORS=';\n' lorem.dat
Lorem,ipsum,dolor,sit,amet,,consectetur,adipiscing,elit.;
Maecenas,pellentesque,erat,vel,tortor,consectetur,condimentum.;
Nunc,enim,orci,,euismod,id,nisi,eget,,interdum,cursus,ex.;
Curabitur,a,dapibus,tellus.;
Lorem,ipsum,dolor,sit,amet,,consectetur,adipiscing,elit.;
Aliquam,interdum,mauris,volutpat,nisl,placerat,,et,facilisis,neque,ultrices.;
````

Tan simple como jugar con las dos variables de salida `OFS` y `ORS`.

<hr>

### 06. Incluir los registros impares en un archivo y los pares en otro

Partimos de la solución:

```` shell
$ awk 'NR%2{print >"impar.dat";next}{print >"par.dat"}' lorem.dat
````

```` shell
$ cat par.dat
Maecenas pellentesque erat vel tortor consectetur condimentum.
Curabitur a dapibus tellus.
Aliquam interdum mauris volutpat nisl placerat, et facilisis.
````

```` shell
$ cat impar.dat
Lorem ipsum dolor sit amet, consectetur adipiscing elit.
Nunc enim orci, euismod id nisi eget, interdum cursus ex.
Lorem ipsum dolor sit amet, consectetur adipiscing elit
````

El símbolo `%` es la función modulo que extrae el resto al dividir el número de registro o línea tratado entre 2:

```` shell
$ awk '{print NR%2}' lorem.dat
1
0
1
0
1
0
````

Como sabemos en `awk` `1` es [verdadero] y `0` *falso*, en base a esta premisa redirigimos la salida según lo requerido.

Hay que prestar **especial atención** al comando **`next`** que fuerza que `awk` deje de forma inmediata de procesar ese registro y pase al siguiente. De esta forma evitamos un doble control de condición que sin **`next`** se hubiera escrito así:

```` shell
awk  'NR % 2{print > "impar.dat"}
     !NR % 2{print > "par.dat"}' lorem.dat
````

<hr>

### 07. Dado un fichero de password informa el campo del home del usuario en base al login

```` shell
$ cat /etc/passwd
jd001:x:1032:666:Javier Diaz::/bin/rbash
ag002:x:8050:668:Alejandro Gonzalez::/bin/rbash
jp003:x:1000:666:Jose Perez::/bin/bash
ms004:x:8051:668:Maria Saenz::/bin/rbash
rc005:x:6550:668:Rosa Camacho::/bin/rbash
````

Supongamos que queremos componer el campo *perdido* anteponiendo la carpeta `/home`:

```` shell
$ awk '$6="/home/"$1' FS=':' OFS=':' /etc/passwd
jd001:x:1032:666:Javier Diaz:/home/jd001:/bin/rbash
ag002:x:8050:668:Alejandro Gonzalez:/home/ag002:/bin/rbash
jp003:x:1000:666:Jose Perez:/home/jp003:/bin/bash
ms004:x:8051:668:Maria Saenz:/home/ms004:/bin/rbash
rc005:x:6550:668:Rosa Camacho:/home/rc005:/bin/rbash
````

Lo primero que tenemos que tener en cuenta es el carácter separador que divide los campos, tanto a la hora de tratar el `stdin` como el `stdout`. Esto se traduce en  `FS=':'` y  `OFS=':'`.

En segundo lugar identificamos la posición en la que se encuentra el campo `void`, en este caso es la número 6 y componemos su nuevo valor en base a la cadena requerida y el primero de los campos que contiene el `login`, el código final: `$6="/home/"$1`.

**IMPORTANTE**: No necesitamos `print` ya que el resultado de la asignación será `true` y el comportamiento por defecto de `awk` cuando algo es *verdadero* es mostrar el registro afectado.

<hr>

### 08. Cambiar el orden de los campos de modo que el primero pase a ser el final

Código final:

```` shell
$ awk -F\: '{last=$1;$1=$NF;$NF=last}1' FS=":" OFS=':' /etc/passwd
/bin/rbash:x:1032:666:Javier Diaz:/home/jd001:jd001
/bin/rbash:x:8050:668:Alejandro Gonzalez:/home/ag002:ag002
/bin/bash:x:1000:666:Jose Perez:/home/jp003:jp003
/bin/rbash:x:8051:668:Maria Saenz:/home/ms004:ms004
/bin/rbash:x:6550:668:Rosa Camacho:/home/rc005:rc005
````

Aquí jugamos con la variable intermedia `last` donde alojamos el valor del primer campo `$1`, después intercambiamos su valor con el del último campo `$1=$NF`,  y finalmente asignamos `last` a `$NF`.

<hr>

### 09. Hackeando traceroute

Partiendo de la salida del *traceador*:

```` shell
$  traceroute -q 1 google.com 2>/dev/null
 1  hitronhub.home (192.168.1.1)  5.578 ms
 2  217.217.0.1.dyn.user.ono.com (217.217.0.1)  9.732 ms
 3  10.127.54.181 (10.127.54.181)  10.198 ms
 4  62.42.228.62.static.user.ono.com (62.42.228.62)  35.519 ms
 5  72.14.235.20 (72.14.235.20)  26.003 ms
 6  216.239.50.133 (216.239.50.133)  25.678 ms
 7  mad01s24-in-f14.1e100.net (216.58.211.238)  25.019 ms
````

Deseamos computar el total de *micro segundos* que tarda un paquete en todo el recorrido *investigado*.

```` shell
$ traceroute -q 1 google.com 2>/dev/null|\
  awk '{total+=$(NF-1)}
       END{print "Total ms: "total}'
Total ms: 153.424
````

La acción se ejecuta para todos los registros ya que no hay ningún tipo de *filtraje*.

`total+=$(NF-1)`: acumulamos en la variable total el valor contenido en el penúltimo campo `$(NF-1)`.

En la regla `END` mostramos el valor final.

<hr>

### 10. Procesos dependientes de un PID padre

En primer lugar obtenemos el `PID` de nuestra *shell*:

```` shell
$ echo $$
51026
````

Lanzamos procesos `sleep` en segundo plano cuyo padre será nuestro `PID` anterior.

```` shell
$ sleep 10 & sleep 15 & sleep 20 &
[1] 86751
[2] 86752
[3] 86753
````

En base al comando `ps` filtramos mediante `awk` por el tercer campo `$3` que corresponde al `PPID`:

```` shell
$ ps -ef|awk '$3==51026'
  501 86751 51026   0  7:57PM ttys001    0:00.00 sleep 10
  501 86752 51026   0  7:57PM ttys001    0:00.00 sleep 15
  501 86753 51026   0  7:57PM ttys001    0:00.00 sleep 20
    0 86754 51026   0  7:57PM ttys001    0:00.00 ps -ef
  501 86755 51026   0  7:57PM ttys001    0:00.00 awk $3==51026
````

Nuestro objetivo final es *volcar* al terminal **solo** los procesos *sleep*:

```` shell
$ ps -ef|awk '$3==51026 && /slee[p]/ {print $2" -> "$5}'
86751 -> 7:57PM
86752 -> 7:57PM
86753 -> 7:57PM
````

Para ello necesitamos añadir al la condición inicial mediante `&&` la búsqueda del patrón `/slee[p]/` en el registro. 

La acción asociada será imprimir el segundo campo que corresponde al `PID` del proceso y el quinto asociado al *timestamp*.

<hr>

### 11. Como agregar datos a partir de una clave

Supongamos el siguiente fichero:

```` shell
$ cat ips.dat
IP            BYTES
81.220.49.127 328
81.220.49.127 328
81.220.49.127 329
81.220.49.127 367
81.220.49.127 5302
81.226.10.238 328
81.227.128.93 84700
````

Nuestro objetivo es determinar cuantos *bytes* han sido tratados por `IP`.

```` shell
$ awk 'NR>1{ips[$1]+=$2}
       END{for (ip in ips){print ip, ips[ip]}}' ips.dat
81.220.49.127 6654
81.227.128.93 84700
81.226.10.238 328
````

Tenemos *bastante* que explicar:

`NR>1{ips[$1]+=$2}`: Cuando el número de registro procesado es superior a 1 se ejecuta acción `ips[$1]+=$2`. EL `NR>1` es necesario para *esquivar* la cabecera del archivo, es decir, la primera línea. `ips` es un array indexado por la ip (el campo `$1`) al que se le asigna la *sumatoria* del campo 2 `$2`.

Es importante tener en cuenta que `awk ` cuando no encuentra una *key* en un *array* añade un nuevo elemento, pero si ocurre lo *lee* y puede manipular su valor como en este caso particular.

En la parte del `END` simplemente recorremos el array mostrando sus índices o keys y el valor contenido.

Este código se podría escribir de esta forma preservando el orden y omitiendo el uso de arrays aprovechando que las `ip`s están ordenadas:

```` shell
awk 'NR == 1{next}
    lip && lip != $1{print lip,sum;sum=0}
    {sum+=$2;lip=$1}
    END{print lip,sum}' ips.dat
81.220.49.127 6654
81.226.10.238 328
81.227.128.93 84700
````

`NR == 1{next}`: Omite la cabecera.

`lip != $1 && lip != ""{print lip,sum;sum=0}`: Usamos `lip` (last ip) como variable auxiliar y solo en el caso de que esta no sea nula `lip` y `&&` sea distinta del la *ip* actual `lip != $1` mostraremos la ip anterior `lip` y la sumatoria `sum` para después inicializarla: `sum=0`.

`{sum+=$2;lip=$1}`: Aquí llevamos a cabo el incremento de la sumatoria de *bytes* `sum+=$2` y asignamos el campo de ip actual a `lip`.

En el bloque `END` mostraremos los resultados para la última `ip` así como la sumatoria final.

Este código preserva el orden pero aumenta la complejidad del programa.

<hr>

### 12. Mostrar los registros entre dos patrones

Necesitamos extraer la líneas comprendidas entre `OUTPUT` y `END`.

```` shell
$ cat pat.dat
test -3
test -2
test -1
OUTPUT
top 2
bottom 1
left 0
right 0
page 66
END
test 1
test 2
test 3
````

Este es un ejemplo clásico a la hora de ilustrar el funcionamiento del macheo de patrones y sus acciones asociadas al que ya le dedique un [post] (en inglés).


```` shell
$ awk '/END/{flag=0}flag;/OUTPUT/{flag=1}' pat.dat
top 2
bottom 1
left 0
right 0
page 66
````

Como podemos ver como su funcionamiento se basa en asociar un valor para `flag` *verdadero* al encontrar el *patrón* de inicio y *falso* al encontrar el que cierra.

Para evitar un paso adicional es importante el orden de las acciones, si lo hacemos siguiendo la secuencia lógica:

```` shell
$ awk '/OUTPUT/{flag=1}flag;/END/{flag=0}' pat.dat
OUTPUT
top 2
bottom 1
left 0
right 0
page 66
END
````

Las *etiquetas* se muestran en la salida ya que después de encontrar el patrón de inicio `OUTPUT` activamos el *flag* y la siguiente acción se realiza sobre el mismo registro, cosa que evitamos si esta activación la realizamos en el último paso de nuestro flujo.

Debemos evitar que `flag` sea `verdadero` cuando no deseemos mostrar los registro.

<hr>

### 13. Convertir un campo en base a su contenido

Supongamos este fichero:

```` shell
$ cat space.dat
10.80 kb
60.08 kb
35.40 kb
2.20 MB
1.10 MB
40.80 kb
3.15 MB
20.50 kb
````

Nuestro objetivo es determinar cuantas megas *pesan* nuestros registros.

```` shell
$ awk '{total+= $1 / ($2=="kb" ? 1024: 1)}
       END{print total}'  space.dat
6.61365
````

Para poder entender su funcionamiento tenemos que tener claro como funciona el operador *ternario* al que ya dedique en su día un [articulo] completo (en inglés).

Sobre la variable `total` acumulamos el resultado de dividir el primer campo `$1` entre el segundo que gracias a la magia de este operador equivaldrá a 1024 en caso de que su valor sea `kb` y a `1` si ya está en MBs.

Finalmente imprimimos el resultado *total* en el bloque `END`.

<hr>

### 14. Agrupando registros en columnas

Nuestro fuente original:

```` shell
$ cat group.dat
string1
string2
string3
string4
string5
string6
string8
````

Pretendemos agrupar los registros en bloques de tres columnas para obtener la siguiente salida:

```` shell
string1 string2 string3
string4 string5 string6
string8
````

Parece complejo , pero resulta mucho más simple si entendemos como usar el *Output Field Separator* `OFS`:

```` shell
$ awk 'ORS=NR%3?" ":RS; END{print "\n"}' group.dat
string1 string2 string3
string4 string5 string6
string8
````

Si establecemos el `ORS` a un carácter en blanco todo la salida se agrupa en una sola línea o registro:

```` shell
$ awk 'ORS=" "; END{print "\n"}' group.dat
string1 string2 string3 string4 string5 string6 string7
````

`ORS=NR%3?" ":RS`: Finalmente usamos el operador *ternario* explicado anteriormente y evaluamos el modulo resultado de dividir el número de registro actual (teniendo en cuenta que el `RS` no se ha tocado por lo que sigue siendo `\n`) entre tres, si el resultado es *true* (distinto a cero) el `ORS` pasara a ser un espacio en blanco, en caso contrario se le asignará el valor del `RS`, es decir el salto de línea *unix*.

<hr>

### 15. Procesando un fichero FASTA

En *bioinformática*, el formato [FASTA] es un formato de fichero informático basado en texto.

Supongamos el siguiente ejemplo:

```` shell
$ cat fasta.dat
>header1
CGCTCTCTCCATCTCTCTACCCTCTCCCTCTCTCTCGGATAGCTAGCTCTTCTTCCTCCT
TCCTCCGTTTGGATCAGACGAGAGGGTATGTAGTGGTGCACCACGAGTTGGTGAAGC
>header2
GGT
>header3
TTATGAT
````

Nuestro objetivo es agregar los resultados de modo que obtengamos la longitud total de las secuencias de cada *header* y una sumarización total de la siguiente forma:

```` shell
>header1
117
>header2
3
>header3
7
3 sequences, total length 127
````

`awk` es una herramienta perfecta para este tipo de *reporting*, en este ejemplo usaremos:

```` shell
awk '/^>/ { if (seqlen) {
              print seqlen
              }
            print

            seqtotal+=seqlen
            seqlen=0
            seq+=1
            next
            }
    {
    seqlen += length($0)
    }
    END{print seqlen
        print seq" sequences, total length " seqtotal+seqlen
    }' fasta.dat

````

La **primera acción** esta asociada a a la detección de la cabecera `/^>/` ya que los *header* son aquellos en los que el registro comienza con el carácter `>`

Cuando la variable `seqlen` tenga valor la mostraremos, en todo caso se volacará por el `stdout` la cabecera, se acumula la longitud total en `seqtotal`, el número de secuencias tratadas en `seq` y se inicializará la longitud de la secuencia actual `seqlen`, finalmente pasamos al siguiente registro con `next`.

Las **segunda acción** es acumular en `seqlen` la longitud de línea *cuando no se trate de una cabecera*.

En el **bloque `END`** mostramos la longitud de secuencia remanente y los totales.

Este ejemplo se basa en mostrar la longitud de secuencia del *header* anterior, al procesar el primer registro evidentemente no tiene valor y se omite, en el `END` se muestra la información disponible después de procesar la última línea.

<hr>

### 16. *Reporting* complejo

Supongamos el siguiente fichero:

```` shell
$ cat report.dat
       snaps1:          Counter:             4966
        Opens:          Counter:           357283

     Instance:     s.1.aps.userDatabase.mount275668.attributes

       snaps1:          Counter:                0
        Opens:          Counter:           357283

     Instance:     s.1.aps.userDatabase.test.attributes

       snaps1:          Counter:             5660
        Opens:          Counter:            37283

     Instance:     s.1.aps.userDatabase.mount275000.attributes
````

Nuestro objetivo es hacer un `report` en el se muestre la línea de *snaps* y su instancia asociada pero unicamente cuando el valor del primer *counter* sea superior a cero, en este caso buscaríamos:

```` shell
snaps1: Counter: 4966
Instance: s.1.aps.userDatabase.mount275668.attributes
snaps1: Counter: 5660
Instance: s.1.aps.userDatabase.mount275000.attributes
````

Para conseguir este resultado nuevamente jugaremos con *patrones* y *flags*:

```` shell
awk '{$1=$1}
     /snaps1/ && $NF>0{print;f=1}
     f &&  /Instance/ {print;f=0}'  report.dat
````

La *primera acción* `{$1=$1}` se ejecuta para todos los registros y como ya hemos visto sirve para reconstruir los registros, en este caso *el truco* nos permite convertir los separadores basados en múltiples espacios en blanco en uno solo ya que es el valor por defecto para el `OFS`.

Lo apreciamos mejor con un ejemplo:

```` shell
$ awk '1' text.dat
one      two
three              four
````

```` shell
$ awk '$1=$1' text.dat
one two
three four
````
La *segunda acción* se dispara cuando encontramos el patrón `/snaps1/` y (`&&`) el último campo es mayor que 0 `$NF>0`. Mostraremos el registro con `print` y daremos un valor *verdadero* al flag `f=1`.

Por último cuando el flag es *verdadero* y encontramos el patrón de *instancia* `f &&  /Instance/` imprimimos la línea y desactivaremos la variable: `print;f=0`.

<hr>

### 17. *JOIN* entre ficheros

Partimos de dos ficheros.

```` shell
$ cat join1.dat
3.5 22
5. 23
4.2 42
4.5 44
````

```` shell
$ cat join2.dat
3.5
3.7
5.
6.5
````

Necesitamos obtener las registros del primero cuyas columnas están presentes en `join2.dat`. Es decir:

```` shell
3.5 22
5. 23
````

Por supuesto que podríamos usar la utilidad *unix* `join` ordenado el primer fichero:

```` shell
$ join <(sort join1.dat) join2.dat
3.5 22
5. 23
````

Veamos como se puede hacer con `awk`:

```` shell
$ awk 'NR==FNR{a[$1];next}
       $1 in a'  join2.dat join1.dat
````

Como siempre descompongamos por acciones.

Cuando el número de registro procesado equivalga al número de registro del fichero actual, es decir `NR==NFR` significará que estamos tratando el **primer** archivo.

La *acción* asociada será incorporar al array `a` un elemento *nulo* indexado el contenido del primer campo `$1` , es decir `a[$1]` inmediatamente después pasamos a procesar el siguiente registro mediante `next`.

La **segunda acción** se disparará de forma implícita cuando el `NR` sea distinto al `FNR`, es decir, cuando estemos procesando el *segundo fichero*, le aplicaremos el *statement* `$1 in a` que será *true* cuando el primer campo del fichero `join1.dat` este  presente en el *array* formado por los campos del `join2.dat`.

Como ya sabemos, cuando algo es *verdadero* en `awk` por defecto se muestra el registro afectado por la condición por lo que obtenemos el resultado esperado.

<hr>

### 18. Cruzando *passwd* y *group*

Supongamos estos dos clásicos *unix*:

```` shell
$ cat /etc/group
dba:x:001:
netadmin:x:002:
````

```` shell
$ cat /etc/passwd
jd001:x:1032:001:Javier Diaz:/home/jd001:/bin/rbash
ag002:x:8050:002:Alejandro Gonzalez:/home/ag002:/bin/rbash
jp003:x:1000:001:Jose Perez:/home/jp003:/bin/bash
ms004:x:8051:002:Maria Saenz:/home/ms004:/bin/rbash
rc005:x:6550:002:Rosa Camacho:/home/rc005:/bin/rbash
````

Nuestro objetivo es obtener un *report* `login:nombre_grupo`:

```` shell
d001:dba
ag002:netadmin
jp003:dba
ms004:netadmin
rc005:netadmin
````

Para ello volveremos a usar las técnicas de trabajo con varios ficheros vistas en el ejemplo anterior:

```` shell
$ awk -F\: 'NR==FNR{g[$3]=$1;next}
            $4 in g{print $1""FS""g[$4]}' /etc/group /etc/passwd
````

Para procesar `/etc/group` volvemos a usar la comparación `NR==FNR` para disparar la primera acción que almacenará en el array `g` bajo el index correspondiente al *ID* del grupo `$3` su nombre `$1`, es decir `g[$3]=$1`. Posteriormente rompemos cualquier procesamiento posterior del registro con `next`.

La siguiente acción afectará a todos los registros del archivo `/etc/passwd`, cuando el cuarto campo `$4` (que corresponde al *ID* del grupo del usuario) este presente en el array `$4 in g` mostraremos el *login* `$1` y el valor asociado al elemento del array referido por la *ID* `g[$4]`. En definitiva: `print $1""FS""g[$4]`

<hr>

### 19. Conexiones por usuario a un servidor

Partimos de la salida del comando `users`, ejemplo:

```` shell
$ users
negan rick bart klashxx klashxx ironman ironman ironman
````

Necesitamos saber cuantos *logeos* tenemos por *usuario*

```` shell
$ users|awk '{a[$1]++}END{for (i in a){print i,a[i]}}' RS=' +'
rick 1
bart 1
ironman 3
negan 1
klashxx 2
````

En este código la acción se ejecuta para todos los registros devueltos por el ejecutable ya que no existe ninguna filtro o patrón a procesar.

`a[$1]++`: Inicializa como 1 o suma un elemento sobre la key de *usuario* `$1` del *array* `a`. En `awk` cuando un valor no esta *iniciado* se considera cero en un contexto numérico.

En el bloque `END` recorremos el array mostrando la *key* y su valor asociado.

<hr>

### 20. Obteniendo la media total proporcionada por el comando *uptime*

Veamos la salida típica de esta utilidad:

```` shell
$ uptime
 11:08:51 up 121 days, 13:09, 10 users,  load average: 9.12, 14.85, 20.84
````

¿Como podemos extraer la media total de las tres mediadas del *load average*?

```` shell
$ uptime |awk '{printf "Media de carga: %0.2f\n", 
                ($(NF-2)+$(NF-1)+$(NF))/3 }' FS='(:|,) +'
Media de carga: 14.94
````

Aquí entra en juego una nueva técnica, usamos como separador de campos `FS` una expresión regular `FS='(:|,) +'`.

Le estamos indicando al programa que el separador pueden ser tanto los *dos puntos* `:`  como las comas `,` seguidos por *cero o más* espacios.

Después simplemente nos quedamos con los tres últimos campos en base al `NF`, realizamos la operación aritmética y mostramos el resultado usando la mascara más idónea para `printf`.

<hr>

## :warning: Disclaimer 2 :warning:

Si has llegado hasta aquí ¡gracias por el interés! 

En mi opinión, `awk` es un lenguaje infravalorado y que merece mucho mas cariño :heart:.

Si además te has quedado con ganas de más házmelo saber en los comentarios y considerare una segunda parte para terminar de matarte de aburrimiento.

Happy coding!

<hr>

[^1]: [A Guide to Unix Shell Quoting][quoting-guide].

[^2]: [Wikipedia sobre pipelines][pipes].

[^3]: En ocasiones es conveniente forzar que `awk` reconstruya el *registro* completo, usando los valores actuales para el `FS` y `OFS`.

      Para  hacerlo usamos esta asignación inocua: `$1 = $1`

[^4]: La respuesta rápida, simplemente es un *atajo* para evitar usar la función `print`.

      En `awk` cuando se cumple una condición la *acción por defecto* es imprimir la línea actual en el *input*.

      `$ echo "test" |awk '1'`

      Lo que es equivalente a:

      `echo "test"|awk '1==1'`

      `echo "test"|awk '{if (1==1){print}}'`

      El motivo es que `1` siempre va a ser [verdadero].

[gnu-awk]: https://www.gnu.org/software/gawk/manual/gawk.html "The GNU Awk User’s Guide"
[so]: http://stackoverflow.com/search?tab=votes&q=user%3a1200821%20%5bawk%5d "Stackoverflow"
[quoting-guide]: http://resources.mpi-inf.mpg.de/departments/rg1/teaching/unixffb-ss98/quoting-guide.html "A Guide to Unix Shell Quoting"
[pipes]: https://en.wikipedia.org/wiki/Pipeline_(Unix) "Pipeline (Unix)"
[verdadero]: https://www.gnu.org/software/gawk/manual/gawk.html#Truth-Values-and-Conditions "True and False in awk"
[FASTA]: https://es.wikipedia.org/wiki/Formato_FASTA "Fichero FASTA"
[post]:{{ site.baseurl }}/awk-between-two-patterns "AWK between patterns"
[articulo]:{{ site.baseurl }}/ternary-operator "Ternary operator"
[english]: {{ site.baseurl }}/awk-power-for-your-cmd "AWK power for your command line"
