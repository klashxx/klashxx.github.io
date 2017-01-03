---
layout: post
title: Interfaces en Go
permalink: interfaces-en-go
comments: true
---

Un *interface* en Go **es un tipo**, *tan simple como eso*.

Parce una tontería pero una vez que tenemos claro esto lo demás viene rodado.

Eso si **es un tipo especial**, *especial* porque **solo puede contener métodos**, es decir, las *acciones* que se podrán ejecutar sobre nuestros tipos, generalmente  `structs`.

Para empezar supongamos un objeto `Bici` sobre el que definimos un método para establecer la marcha que vamos a usar:

```go
package main

type Bici struct {
	marcha int
}

func (b *Bici) PonMarcha(marcha int) {
	b.marcha = marcha
}

func main() {
	vehiculo := &Bici{}
	vehiculo.PonMarcha(14)
}
```
:warning: **NOTA**: `PonMarcha` debe implementarse con un **pointer receiver** para permitir la modificación de los valores / propiedades de la estructura.[^1]

¡Vale fácil! Ahora añadimos una `Moto`:

```go
package main

type Vehiculo interface {
	PonMarcha(marcha int)
}

type Bici struct {
	marcha int
}

func (b *Bici) PonMarcha(marcha int) {
	b.marcha = marcha
}

type Moto struct {
	marcha int
}

func (m *Moto) PonMarcha(marcha int) {
	m.marcha = marcha
}

func main() {
	vehiculo := &Bici{}
	vehiculo.PonMarcha(14)
	vehiculo = &Moto{}
	vehiculo.PonMarcha(2)
}
```

Parece directo… pero este código **no es compilable**:

```bash
./main.go:29: cannot use Moto literal (type *Moto) as type *Bici in assignment
```

`Moto` y `Bici` no tienen nada en común … ¿o sí?

Parece evidente que estos dos objetos comparten ciertas *acciones* / *métodos*, en nuestro caso `PonMarcha`, por lo que podríamos definir un tipo `interface` que los agrupe:

```go
type Vehiculo interface {
	PonMarcha(marcha int)
}
```

Veamos el código completo:

```go
package main

type Bici struct {
	marcha int
}

func (b *Bici) PonMarcha(marcha int) {
	b.marcha = marcha
}

type Moto struct {
	marcha int
}

func (m *Moto) PonMarcha(marcha int) {
	m.marcha = marcha
}

type Vehiculo interface {
	PonMarcha(marcha int)
}

func main() {
        var vehiculo Vehiculo

	vehiculo = &Bici{}
	vehiculo.PonMarcha(14)
	vehiculo = &Moto{}
	vehiculo.PonMarcha(2)
}
```

¡Nuestro código ya compila!  Podemos usar una variable tipo `Vehiculo` porque sabemos que tanto el tipo `Bici` como el tipo `Moto` satisfacen su  *interface*.

Esa es la _**condición sin ecua non**_: todos los tipos deben implementar todos los métodos del *interface*, pero **no** quedan limitados por el, por ejemplo a `Moto` se le podría añadir la acción para echar gasolina:

```go
func (m *Moto) EchaGasolina(nivel int) {
    m.nivel = nivel
}
```
Seguiría satisfaciendo el *interface* `Vehiculo` pero , **OJO**, como es lógico, no se podría acceder al nuevo método desde el:

```bash
./main.go:36: vehiculo.EchaGasolina undefined (type Vehiculo has no field or method EchaGasolina)
```

### Stringer

Este es el ejemplo clásico del *interface*  [**`Stringer`**][stringer]  que nos acercará mucho más a su uso más práctico.

Está definido en el paquete [**`fmt`**][fmt]  y su código no puede ser más simple:

```go
type Stringer interface {
        String() string
}
```

**Todo lo que nos importa es**: si definimos un objeto que satisfaga este *interface* podremos aprovecharnos la funcionalidad implementada en [**`print`**][print] definida como:

> The String method is used to print values passed as an operand to any format that accepts a string or to an unformatted printer such as Print.

Un tipo que implemente el *interface* *Stringer* podrá pasarse a cualquier función de *impresión* de [fmt][fmt] para obtener su representación.

Veámoslo en nuestro ejemplo:

```go
package main

import (
  "fmt"
  "strconv"
  )

type Bici struct {
    marcha int
}

func (b *Bici) PonMarcha(marcha int) {
    b.marcha = marcha
}

func (b Bici) String() string {
    return "Bici en marcha número: "+strconv.Itoa(b.marcha)
}

type Moto struct {
    marcha int
    nivel int
}

func (m *Moto) PonMarcha(marcha int) {
    m.marcha = marcha
}

func (m Moto) String() string {
    return "Moto en marcha número: "+strconv.Itoa(m.marcha)
}

type Vehiculo interface {
    PonMarcha(marcha int)
    String() string
}

func main() {
    var vehiculo Vehiculo

    vehiculo = &Bici{}
    vehiculo.PonMarcha(14)
    fmt.Println("Vehículo: ", vehiculo)

    vehiculo = &Moto{}
    vehiculo.PonMarcha(2)
    fmt.Println("Vehículo: ", vehiculo)
}
```

Nos [devuelve][play-stringer]:

```bash
Vehículo:  Bici en marcha número: 14
Vehículo:  Moto en marcha número: 2
```

### Interface{}

Es el **interface vacio**, **no implementa ningún método** y por lo tanto **es satisfecho por cualquier tipo**.

En base a este tipo es posible implementar una función que reciba cualquier tipo de valor, por ejemplo [**`fmt.Print`**][print] que tiene la siguiente [*signature*][declaration]:

```go
func Print(a ...interface{}) (n int, err error)
```

:warning: **Nota**: el tipo precedido por `...` indica que esta función es [variadic][variadic] por lo que puede ser invocada con cero o mas argumentos para es parámetro.

#### ¿Cuál sería el tipo de `a`?

El tipo de `a` **no es** *cualquier tipo*  sino  `interface{}`, al pasar un valor como argumento *go* lo convertirá (si es necesario) al tipo `interface{}`.

Un *interface^* se construye con dos *palabras* de dirección de memoria:

- Una se usa para apuntar la tabla de metodos asociada al tipo "original".
- La otra para apuntar a los datos.

Entender representación evita bastantes confusiones.

Ahora supongamos que queremos construir un [slice][slices] de tipos `Vehiculo`:

```go
vehiculos := []Vehiculo{&Bici{}, &Moto{}}
```

Podemos añadirle directamente los tipos que lo satisfacen porque la conversión al tipo `Vehiculo` se lleva a cabo *directamente*.

En el *slice*  `vehiculos` todos sus elementos son tipo `Vehiculo` pero, **cada uno de sus valores tienen diferentes tipos de origen**.

Si definimos `Vehiculo` como un *interface* vacio:

```go
type Vehiculo interface{}
```

Podremos realizar la asignación, porque tanto `Bici`como `Moto` sadisfacen el *interface vacio*:

```go
vehiculos := []Vehiculo{&Bici{}, &Moto{}}
```
Pero no tendremos acceso a ninguno de sus métodos.


[^1]: ¿Cuándo usar un *pointer receiver* y otro estándar?
      Un *receiver* puede considerarse como un argumente que se pasa a un método.

      - Si queremos modificar el *receiver* usaremos un *pointer* y le pasamos la referencia.
      - Si la estructura *atachada* es muy grande por lo que pasarla por valor es caro.
      - Por consistencia, si la mayor parte de los *receiver* ya son tipo *pointer*.


[fmt]: https://golang.org/pkg/fmt/ "Package fmt"
[stringer]: https://golang.org/pkg/fmt/#Stringer "Stringer"
[slices]: https://blog.golang.org/go-slices-usage-and-internals "Go Slices"
[print]:  https://golang.org/src/fmt/print.go "Print"
[play-stringer]: https://play.golang.org/p/hoOgG9H476 "Playground Stringer"
[declaration]: https://blog.golang.org/gos-declaration-syntax "Declaration syntax"
[variadic]: https://golang.org/ref/spec#Passing_arguments_to_..._parameters "Variadic functions"
