# Pipelines en Go

*Traducción libre de [Go Concurrency Pipelines](https://blog.golang.org/pipelines)*


Un `pipeline` se podría definir como una serie de etapas de proceso conectadas por `channels` (*canales*).

Cada una de estas fases está conformada por un grupo de **goroutines** ejecutando una misma función:
-   **Reciben** valores a partir de los canales *inbound* o de entrada
-   **Ejecutan** algún tipo de *manipulación* sobre esos datos
-   **Devuelven** los valores a través de los canales de salida o *outbound*.

Las etapas están conectadas a través de un número arbitrario de canales de *entrada* y *salida*, excepto la primera y última que solo tendrán de salida y entrada respectivamente.

## Ejemplo, números cuadrados

En una primera fase `gen` sería una función que emitiría los enteros recibidos en *array*.

Se ocupa de arrancar una `goroutine`, enviar los valores por el canal de salida (`return`) y **cerrarlo** una vez se haya completado el envío.

```go
func gen(nums ...int) <-chan int {
    out := make(chan int)

    go func() {
        for _, n := range nums {
            out <- n
        }
        close(out)
    }()
    return out
}
```

La segunda *fase* `sq` se encarga de recibir los enteros emitidos por la función `gen` y devolverlos por otro canal *elevados* al cuadrado, sin olvidar cerrar el *channel* al concluir el procesamiento.


```go
func sq(in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        for n := range in {
            out <- n * n
        }
        close(out)
    }()
    return out
}
```


En el `main` configuramos el *pipeline* y se ejecuta la fase final. **No es necesario cerrar** ya que solo controla el *inbound*.

```go
func main() {
    // Set up the pipeline.
    c := gen(2, 3)
    out := sq(c)

    // Consume the output.
    fmt.Println(<-out) // 4
    fmt.Println(<-out) // 9
}
```

Como `sq` usa los mismos tipos para los canales de entrada y salida, podemos reusarlos y consumirlos mediante `range` al igual que se hizo en anteriores etapas.

```go
func main() {
    // Set up the pipeline and consume the output.
    for n := range sq(sq(gen(2, 3))) {
        fmt.Println(n) // 16 then 81
    }
}
```

## Fan-Out, Fan-in

Se pueden levantar múltiples funciones que se alimenten del mismo canal `inbound`.

A esto se conoce como **Fan-out** y permite paralelizar de forma simple el trabajo de los *workers* o consumidores.

También se puede usar una **única** función que lea **múltiples** canales *inboud* para *agregar* los resultados.

Para esta labor se usa la técnica del multiplexado de canales en uno solo que se cierra cuando todos los input lo hacen. A esto se conoce como **Fan-in**.

Podemos cambiar el *pipeline* para ejecutar dos instancias de `sq`, leerán del mismo canal de entrada necesitando una nueva función `merge` que se encargará del `Fan-in` de los resultados.

```go
func main() {
    in := gen(2, 3)

    // Distribute the sq work across two goroutines that both read from in.
    c1 := sq(in)
    c2 := sq(in)

    // Consume the merged output from c1 and c2.
    for n := range merge(c1, c2) {
        fmt.Println(n) // 4 then 9, or 9 then 4
    }
}
```

`merge` convierte un array de canales en único, arrancando una *goroutine* para cada uno de los canales de entrada que *copia* los valores un único canal de salida.

Una vez arrancados todos los canales de salida `merge` lanza una ruina más para cerrar el canal `outbound` una vez completados todos los envíos.

**ATENCIÓN**: enviar sobre un canal cerrado produce **panic** afortunadamente [sync.WaitGroup](https://golang.org/pkg/sync/#WaitGroup) nos facilita la tarea de sincronización.

```go
func merge(cs ...<-chan int) <-chan int {
    var wg sync.WaitGroup
    out := make(chan int)

    // Start an output goroutine for each input channel in cs.  output
    // copies values from c to out until c is closed, then calls wg.Done.
    output := func(c <-chan int) {
        for n := range c {
            out <- n
        }
        wg.Done()
    }
    wg.Add(len(cs))
    for _, c := range cs {
        go output(c)
    }

    // Start a goroutine to close out once all the output goroutines are
    // done.  This must start after the wg.Add call.
    go func() {
        wg.Wait()
        close(out)
    }()
    return out
}
```


##### *Resumiendo*

-   Cada *stage* cierra sus canales de salida al terminar los envíos.
-   Cada *stage* se mantiene escuchando, recibiendo valores para los canales de entrada hasta que estos son cerrados.

Este patrón permite que cada *recibi*dor pueda componerse como un bucle `range` y asegura que todas las *goroutines* terminen una vez procesados **todos** los valores.


Pero ... *en la vida real* los recibidores no siempre obtienen todos los valores de entrada.

A veces, por diseño, el recibidor puede que solo necesite un grupo de valores para continuar progresando.

Más frecuentemente, terminan antes de tiempo al recibir errores en una fase temprana.

En cualquiera de estos casos el recibidor no debe *esperar* a que lleguen **todos** los valores restantes, necesitaremos que los *productores* dejen de generar valores que ya no son necesarios en fases posteriores.

En nuestro ejemplo si una de las fases falla al *consumir* los canales de entrada, las *goroutines* que estén intentando enviar a ese *stage* se bloquearan indefinidamente.

```go
// Consume the first value from output.
out := merge(c1, c2)
fmt.Println(<-out) // 4 or 9
return
// Since we didn't receive the second value from out,
// one of the output goroutines is hung attempting to send it.
```

Una de las formas de evitar este problema es establecer un `buffer` para los canales *inbound*.

Por ejemplo`, podemos usarlo en la función `gen y **evitar** crear una nueva `goroutine.

```go
func gen(nums ...int) <-chan int {
    out := make(chan int, len(nums))
    for _, n := range nums {
        out <- n
    }
    close(out)
    return out
}
```

Del mismo modo podemos tratar el `merge`.

```go
func merge(cs ...<-chan int) <-chan int {
    var wg sync.WaitGroup
    out := make(chan int, 1) // enough space for the unread inputs
    // ... the rest is unchanged ...
```

Aunque **esto** evita el problema de los canales bloqueados **se puede considerar una mala praxis**.

La elección del tamaño de buffer depende de conocer de antemano el número de valores que va a recibir `merge` y el que *consumirán* las fases anteriores.

Si pasamos un valor adicional a `gen` o si los *recibidores* leen un menor número de valores tendrá como consecuencia un nuevo bloqueo.


## Cancelación explicita

Cuando `main` *decide* que debe terminar sin recibir todos los valores de salida, **debe comunicarlo** a todos las rutinas *upstream*, para esta labor usa el canal `done`.

```go
func main() {
    in := gen(2, 3)

    // Distribute the sq work across two goroutines that both read from in.
    c1 := sq(in)
    c2 := sq(in)

    // Consume the first value from output.
    done := make(chan struct{}, 2)
    out := merge(done, c1, c2)
    fmt.Println(<-out) // 4 or 9

    // Tell the remaining senders we're leaving.
    done <- struct{}{}
    done <- struct{}{}
}
```

Las rutinas de envío cambian su operación `send` por una sentencia `select` que procederá en función de que procese un mensaje de salida o que reciba un `done`.

`done` es una estructura vacía porque realmente no importa su contenido, simplemente es un evento al recibir que indica que el `send` en el productor debe ser *abandonado*.

Las rutinas productoras continúan escaneando su canal de entrada `c` por lo que no se bloquean los `streams` de envío.

```go
func merge(done <-chan struct{}, cs ...<-chan int) <-chan int {
    var wg sync.WaitGroup
    out := make(chan int)

    // Start an output goroutine for each input channel in cs.  output
    // copies values from c to out until c is closed or it receives a value
    // from done, then output calls wg.Done.
    output := func(c <-chan int) {
        for n := range c {
            select {
            case out <- n:
            case <-done:
            }
        }
        wg.Done()
    }
    // ... the rest is unchanged ...
```

Este planteamiento deriva en otro problema, cada *recibidor*  necesita conocer el número de potenciales *bloqueadores* *productores* y coordinar el envío a los *senders* de un retorno temprano.

Nuestro objetivo será: **ordenar a un número indeterminado e ilimitado de rutinas que paren la producción de mensajes**.

Podemos conseguirlo cerrando un canal, porque *una operación de recepción sobre un canal cerrado se ejecutará de forma inmediata* devolviendo el valor cero del tipo.

Esto significa que el `main` puede desbloquear todos los `senders` simplemente cerrando el canal `done`.

Esta actuación se traduce efectivamente en una señal *broadcast* sobre todos los `senders`. Todos los *stages* en nuestro *pipeline* deben estar preparadas para aceptar `done` como *parámetro* y lanzar el close vía `defer`, de tal forma que cualquier `return` desde el `main de la orden de salida a todas las *fases* del *pipeline*.


```go
func main() {
    // Set up a done channel that's shared by the whole pipeline,
    // and close that channel when this pipeline exits, as a signal
    // for all the goroutines we started to exit.
    done := make(chan struct{})
    defer close(done)

    in := gen(done, 2, 3)

    // Distribute the sq work across two goroutines that both read from in.
    c1 := sq(done, in)
    c2 := sq(done, in)

    // Consume the first value from output.
    out := merge(done, c1, c2)
    fmt.Println(<-out) // 4 or 9

    // done will be closed by the deferred call.
}
```

Ahora la rutina de salida del `merge` puede *escapar* sin consumir todo el canal de entrada ya que sabe que el productor `sq` dejará de enviar cuando cierre el canal `done`.

La llamada a `wg.Done` **es asegurada** por `defer`.

```go
func merge(done <-chan struct{}, cs ...<-chan int) <-chan int {
    var wg sync.WaitGroup
    out := make(chan int)

    // Start an output goroutine for each input channel in cs.  output
    // copies values from c to out until c or done is closed, then calls
    // wg.Done.
    output := func(c <-chan int) {
        defer wg.Done()
        for n := range c {
            select {
            case out <- n:
            case <-done:
                return
            }
        }
    }
    // ... the rest is unchanged ...
```

Del mismo modo `sq` puede *retornar* tan pronto como se cierre `done.

```go
func sq(done <-chan struct{}, in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for n := range in {
            select {
            case out <- n * n:
            case <-done:
                return
            }
        }
    }()
    return out
}
```


##### *Resumiendo*

-   Los canales de salida deben cerrase tan pronto se finalicen las operaciones de envío.
-   Los canales de entrada continua recibiendo hasta que son cerrados o desbloqueados.

Los *pipelines* desbloquean los productores asegurándose de que existe espacio suficiente en el buffer u ordenándolo directamente al abandonar un canal.


## Ejemplo MD5


`MD5` es un algoritmo de mensajes  útil para *checksum* de ficheros.

Ejemplo de salida de [`md5sum`](https://es.wikipedia.org/wiki/Md5sum).

```bash
% md5sum *.go
d47c2bbc28298ca9befdfbc5d3aa4e65  bounded.go
ee869afd31f83cbb2d10ee81b2b831dc  parallel.go
b88175e65fdcbc01ac08aaf1fd9b5e96  serial.go
```

Lo *imitaremos* usando un directorio como argumento para mostrar por `stdout` los valores de *checksum* para cada uno de los ficheros que contiene.


El `main` contendrá una función helper `MD5All` que retornará un `map` que asocie el `path``  y el valor `md5, finalmente ordena y muestra los resultados.

```go
func main() {
    // Calculate the MD5 sum of all files under the specified directory,
    // then print the results sorted by path name.
    m, err := MD5All(os.Args[1])
    if err != nil {
        fmt.Println(err)
        return
    }
    var paths []string
    for path := range m {
        paths = append(paths, path)
    }
    sort.Strings(paths)
    for _, path := range paths {
        fmt.Printf("%x  %s\n", m[path], path)
    }
}
```


El `MD5All` es el centro de nuestro análisis. En [`serial.go`](https://blog.golang.org/pipelines/serial.go) la implementación no usa concurrencia se limita a leer y calcular el *checksum* de cada fichero al recorrer el *árbol* de ficheros.

```go
// MD5All reads all the files in the file tree rooted at root and returns a map
// from file path to the MD5 sum of the file's contents.  If the directory walk
// fails or any read operation fails, MD5All returns an error.
func MD5All(root string) (map[string][md5.Size]byte, error) {
    m := make(map[string][md5.Size]byte)
    err := filepath.Walk(root, func(path string, info os.FileInfo, err error) error {
        if err != nil {
            return err
        }
        if !info.Mode().IsRegular() {
            return nil
        }
        data, err := ioutil.ReadFile(path)
        if err != nil {
            return err
        }
        m[path] = md5.Sum(data)
        return nil
    })
    if err != nil {
        return nil, err
    }
    return m, nil
}
```


## Procesamiento paralelo

En [`parallel.go`](https://blog.golang.org/pipelines/parallel.go) dividimos `MD5All` en un *pipeline* en **dos fases**.

En la primera, `sumFiles` se encargará de recorrer el *árbol* y leer cada fichero *regular* mediante una `goroutine`, finalmente envía lo producido a un canal tipo `result`.

```go
type result struct {
    path string
    sum  [md5.Size]byte
    err  error
}
```


`sumFiles` devuelve dos canales, uno para `results` y otro para los posibles errores devueltos por [`filepah.Walk`](https://golang.org/pkg/path/filepath/#Walk).

`walk` arranca una nueva *goroutine* para cada uno de los archivos, y comprueba el canal `done`, si este se cierra `sumFiles` termina **inmediatamente**.

```go
func sumFiles(done <-chan struct{}, root string) (<-chan result, <-chan error) {
    // For each regular file, start a goroutine that sums the file and sends
    // the result on c.  Send the result of the walk on errc.
    c := make(chan result)
    errc := make(chan error, 1)
    go func() {
        var wg sync.WaitGroup
        err := filepath.Walk(root, func(path string, info os.FileInfo, err error) error {
            if err != nil {
                return err
            }
            if !info.Mode().IsRegular() {
                return nil
            }
            wg.Add(1)
            go func() {
                data, err := ioutil.ReadFile(path)
                select {
                case c <- result{path, md5.Sum(data), err}:
                case <-done:
                }
                wg.Done()
            }()
            // Abort the walk if done is closed.
            select {
            case <-done:
                return errors.New("walk canceled")
            default:
                return nil
            }
        })
        // Walk has returned, so all calls to wg.Add are done.  Start a
        // goroutine to close c once all the sends are done.
        go func() {
            wg.Wait()
            close(c)
        }()
        // No select needed here, since errc is buffered.
        errc <- err
    }()
    return c, errc
}
```

`MD5All` recibe los valores, en caso de error finaliza y realiza el close vía `defer`.

```go
func MD5All(root string) (map[string][md5.Size]byte, error) {
    // MD5All closes the done channel when it returns; it may do so before
    // receiving all the values from c and errc.
    done := make(chan struct{})
    defer close(done)

    c, errc := sumFiles(done, root)

    m := make(map[string][md5.Size]byte)
    for r := range c {
        if r.err != nil {
            return nil, r.err
        }
        m[r.path] = r.sum
    }
    if err := <-errc; err != nil {
        return nil, err
    }
    return m, nil
}
```


## Paralelismo limitado.

El ejemplo anterior arrancábamos una rutina por cada fichero. **Esto puede provocar el agotamiento de los recursos del sistema**.

**Podemos limitar este uso** estableciendo un número máximo de ficheros a leer en paralelo.

En  [`bounded.go`](https://blog.golang.org/pipelines/bounded.go) se establecen un **número fijo** de *goroutines* para leer los ficheros.

Nuestro *pipeline* se compondrá de tres etapas.

-   Recorrer el árbol.
-   Leer los ficheros y obtener su *checksum*.
-   Recoger los resultados.


En la **primera etapa** `walkFiles` retorna los `path` a los archivos.

```go
func walkFiles(done <-chan struct{}, root string) (<-chan string, <-chan error) {
    paths := make(chan string)
    errc := make(chan error, 1)
    go func() {
        // Close the paths channel after Walk returns.
        defer close(paths)
        // No select needed for this send, since errc is buffered.
        errc <- filepath.Walk(root, func(path string, info os.FileInfo, err error) error {
            if err != nil {
                return err
            }
            if !info.Mode().IsRegular() {
                return nil
            }
            select {
            case paths <- path:
            case <-done:
                return errors.New("walk canceled")
            }
            return nil
        })
    }()
    return paths, errc
}
```

En la **fase intermedia** se arranca un *número fijo de rutinas* que reciben los `paths` a los ficheros y envían los resultados al canal `c`.

```go
func digester(done <-chan struct{}, paths <-chan string, c chan<- result) {
    for path := range paths {
        data, err := ioutil.ReadFile(path)
        select {
        case c <- result{path, md5.Sum(data), err}:
        case <-done:
            return
        }
    }
}
```

Al contrario que en ejemplos anteriores, `digester` no cierra su canal de entrada, ya que puede ser usado por múltiples *goroutines* en un canal común.  `MD5All` coordina todos canales para cerrarlos cuando finalicen todos los `digesters`.

```go
 // Start a fixed number of goroutines to read and digest files.
c := make(chan result)
var wg sync.WaitGroup
const numDigesters = 20
wg.Add(numDigesters)
for i := 0; i < numDigesters; i++ {
    go func() {
        digester(done, paths, c)
        wg.Done()
    }()
}
go func() {
    wg.Wait()
    close(c)
}()
```

Podríamos optar por que cada uno de los `digester` crearan y retornaran su propio canal pero esto obligaría a crear `goroutines` adicionales para poder realizar el *Fan-in* de los resultados.

La **última fase** recibe todos los `result` de `c` y chequea el canal de error. Esta comprobación no puede realizarse previamente, antes de este punto `walkFiles` podría bloquear enviando valores al *downstream*.


```go
m := make(map[string][md5.Size]byte)
for r := range c {
    if r.err != nil {
        return nil, r.err
    }
    m[r.path] = r.sum
}
// Check whether the Walk failed.
if err := <-errc; err != nil {
    return nil, err
}
return m, nil
```
