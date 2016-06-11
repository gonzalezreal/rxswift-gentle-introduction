footer: “RxSwift, A gentle introduction” - Guille González **@gonzalezreal**
slidenumbers: false
autoscale: true
build-lists: true

# _**RxSwift**_
## A gentle introduction

---

### Guille González
### _**@gonzalezreal**_

^ Mi nombre es Guille González, podéis encontrarme en GitHub, Twitter y Medium como @gonzalezreal.

^ Soy Tech Lead de iOS en Telefonica y también trabajo como formador en KeepCoding.

^ Hoy vengo a hablaros de Programación Reactiva en Swift.

^ Swift ha impulsado la adopción de estilos y patrones de programación funcional entre los desarrolladores de aplicaciones. Si has estado huyendo de estos estilos de programación, es el momento de parar y empezar a adoptarlos.

---

![inline](https://avatars3.githubusercontent.com/u/6407041?v=3&s=400)

## http://reactivex.io

^ La programación funcional reactiva es uno de esos estilos, y se está haciendo popular gracias a ReactiveX y a su encarnación
en Swift.

---

## _**Observer**_
## on steroids

![right](http://www.fringebloggers.com/wp-content/uploads/2015/10/fringe-september.jpg)

^ Rx extiende el patrón observer para soportar secuencias de eventos y añade operadores que permiten componer estas secuencias de manera declarativa, abstrayendo detalles como threading, sincronización, etc.

---

Taps, keyboard events, timers, GPS events, web service responses
# ↓
UI update, data written to disk, API request, etc.

^ Una forma de pensar acerca del software en términos de transformar entradas para producir salidas de manera continua a lo largo del tiempo.

---

# **Asynchronous Events in Cocoa**
- Target-Action
- `NSNotificationCenter`
- Key-Value Observing
- Delegates
- Callback Closures

^ Estos son los mecanismos estándar que proporciona Cocoa para tratar con eventos asíncronos.

^ Ahora imagina tratar con eventos procedentes de estas fuentes con un único tipo.

---

## **`Observable<Element>`**

^ Rx gira en torno al concepto de *secuencias de eventos observables*, o simplemente **Observables**.

^ Con *Observables*, puedes tratar eventos asíncronos de manera uniforme, usando el mismo tipo de operaciones que usas con las colecciones de Swift o los opcionales.

^ Puede contener un número indeterminado de eventos.

^ Captura valores presentes y futuros.

^ Podemos usarlo para representar operaciones asíncronas.

---

### **`--1--2--3--4--5-|--->`**
### **`--a--b--b--a--✕----->`**
### **`---tap-tap----tap--->`**
### **`---JSON-|----------->`**
### **`-|------------------>`**
### **`-✕------------------>`**

^ Next, Error, Completed

^ Las secuencias pueden contener 0 ó más eventos.

^ Una vez recibido un evento `Error` ó `Completed`, la secuencia no puede producir más eventos.

^ Algunas secuencias son finitas y otras infinitas, como una secuencia de "toques" en un botón.

^ Una secuencia también puede representar una operación asíncrona.

---

```swift
enum Event<Element>  {
    case Next(Element)
    case Error(ErrorType)
    case Completed
}
```

^ Rx utiliza un tipo enumerado para modelar los eventos de una secuencia, aprovechando los valores asociados para proporcionar tanto el dato de un evento `Next`, como el error de un evento `Error`.

^ Los diagramas que hemos visto antes, en realidad se parecen más a esto...

---

### **`--Next("a")--Next("b")--Error(diskError)->`**
### **`-----Next---Next-------Next-------------->`**
### **`---Next(json)-Completed------------------>`**

^ 2ª secuencia: Un tap en un botón no lleva ningún dato asociado, es perfectamente válido pasar `Void` como evento Next.

---

## Creating Observables

- ```Observable.empty()```
- ```Observable.just("🏀")```
- ```Observable.of("🏀", "🎾", "⚽️")```
- ```Observable.error(Error.CouldNotDecodeJSON)```

^ Hay muchas formas de crear Observables.

^ La 1ª crea un observable que envía un evento completed.

^ La 2ª crea un observable que envía un evento next y un evento completed.

^ La 3ª crea un observable que envía 3 eventos next y un evento completed.

^ Y la última crea un observable que envía un evento error.

---

## Creating Observables

```swift
let o = Observable.create { observer in
    observer.on(.Next("👋 world!"))
    observer.on(.Completed)
    return NopDisposable.instance
}
```

^ Este es el método más potente para crear observables, ya q

^ El closure que pasamos como parámetro al método `create` recibe un subscriptor al que se le envían ambos eventos.

^ ¿Pensáis que este código hace algo?

---

> If a tree falls in a forest and no one is around to hear it, does it make a sound?
-- George Berkeley

^ Por defecto, un objeto observable no va a ejecutar el closure de subscripción a menos que haya un subscriptor. De hecho, el closure se ejecuta una vez por subscripción. A estos observables se les denomina `fríos`.

^ Si el árbol hace ruido al caer aunque no haya nadie para escucharlo, entonces el observable es `caliente`.

---
```
┌─────────────────────────────────────┬─────────────────────────────────────┐
│           Hot Observables           │          Cold observables           │
├─────────────────────────────────────┼─────────────────────────────────────┤
│Use resources even when there are no │Don't use resources until there is a │
│subscribers.                         │subscriber.                          │
├─────────────────────────────────────┼─────────────────────────────────────┤
│Resources usually shared between all │Resources usually allocated per      │
│the subscribers.                     │subscriber.                          │
├─────────────────────────────────────┼─────────────────────────────────────┤
│Usually stateful.                    │Usually stateless.                   │
├─────────────────────────────────────┼─────────────────────────────────────┤
│UI controls, taps, sensors, etc.     │HTTP request, async operations, etc. │
└─────────────────────────────────────┴─────────────────────────────────────┘
```

---

## Observers

```swift
protocol ObservableType {
    func subscribe(on: (event: Event) -> Void) -> Disposable
}
```

^ Este es el método que usamos para suscribirnos a los eventos de un observable.

^ `subscribe` recibe un closure que se va a ejecutar cada vez que el observable envíe un evento.

^ El objeto `Disposable` que devuelve el método se usa para cancelar la suscripción, liberando cualquier recurso asociado a esta.

---

## Observers

```swift
Observable.create { observer in
    observer.onNext("👋 world!")
    observer.onCompleted()
    return NopDisposable.instance
}.subscribe { event in
    print(event)
}

// outputs:
//   Next(👋 world!)
//   Completed
```

^ En realidad, el closure que estamos pasando en el método `create`, es la implementación de `subscribe`.

^ Presta atención a como estamos enviando los eventos con `onNext` y `onCompleted`.

^ También puedes llamar a métodos como `subscribeNext`, `subscribeError` ó `subscribeCompleted`.

---

## Observers

```swift
Observable.create { observer in
    observer.onNext("👋 world!")
    observer.onCompleted()
    return NopDisposable.instance
}.subscribeNext { text in
    print(text)
}

// outputs:
//   👋 world!
```

^ Si sólo estás interesado en un tipo de evento, puedes llamar a versiones de `subscribe` más específicas.

^ ¿De que sirve devolver un objeto de tipo disposable en el closure de creación?

---

## Disposables

```swift
let appleWeb = Observable.create { observer in
    let task = session.dataTaskWithURL(appleURL) { data, response, error in
        if let data = data {
            observer.onNext(data)
            observer.onCompleted()
        } else {
            observer.onError(error ?? Error.UnknownError)
        }
    }
    
    task.resume()
    
    return AnonymousDisposable {
        task.cancel()
    }
}
```

^ Este es un buen ejemplo de como podemos transformar una operación asíncrona en un observable.

^ Usamos la API de NSURLSession para obtener la web de Apple, enviando los eventos correspondientes a nuestro suscriptor en el bloque de finalización de `dataTaskWithURL`.

^ Devolvemos un objeto disposable que cancelará la petición si el algún momento se llama a `dispose()` mientras esta se está ejecutando.

---

## Dispose Bags

```swift
self.disposeBag = DisposeBag()

...

appleWeb.subscribeNext { data in
    print(data)
}.addDisposableTo(disposeBag)
```

^ Un objeto DisposeBag es un contenedor thread-safe que durante su destrucción, va a liberar todos los disposables añadidos al mismo.

---

## Operators

- `map`
- `flatMap`
- `filter`
- `throttle`
- `merge`
- `combineLatest`
- and many more...

^ Aquí es donde ReactiveX brilla, ya que proporciona una serie de operadores para transformar y combinar secuencias observables.

^ Algunos de ellos os resultarán familiares, como `map` y `flatMap`, ya que son los mismos que implementan las colecciones y los opcionales en la librería estándar de Swift.

^ La mayoría de estos operadores devuelven un `Observable`. Esto permite aplicarlos uno detrás del otro, en cadena. Cada operador en la cadena modifica el `Observable` resultante de la operación anterior.

^ Vamos a ver algunos ejemplos.

---

## `map & flatMap` 

```swift
struct Country {
    let name: String
    let borders: [String]
}

protocol CountriesAPI {
    func countryWithName(name: String) -> Observable<Country>
    func countriesWithCodes(codes: [String]) -> Observable<[Country]>
}
```

^ Suponed que tenemos un servicio web que nos devuelve países tanto por nombre como por código de país.

^ Teniendo en cuenta que la propiedad `borders` es un array con los códigos de país con los que el país tiene frontera, ¿cómo podríamos obtener las fronteras de un país por su nombre?

---

## `map & flatMap`

```swift
myAPI.countryWithName("spain")
    .flatMap { country in
        myAPI.countriesWithCodes(country.borders)
    }
    .map { countries in
        countries.map { $0.name }
    }
    .subscribeNext { countryNames in
        print(countryNames)
    }
```

^ `flatMap` transforma los elementos emitidos por un observable en observables, para luego "aplanar" los eventos emitidos por esos observables en un único observable.

^ `map` transforma los elementos emitidos por un observable, aplicando una función a cada elemento.

---

`Observable` chaining is similar to `Optional` chaining:

```swift
let cell = UITableViewCell(style: .Default, reuseIdentifier: nil)

let maybeSize  = cell.imageView?.image?.size
let maybeSize2 = cell.imageView.flatMap { $0.image }.flatMap { $0.size }
```

^ Encadenar observables de un sólo elemento es similar a encadenar opcionales, de hecho, también puedes usar `flatMap` en vez de el operador `?`.

^ Ten en cuenta que para Observables de más de un elemento, se van a "mapear" todos los elementos. Puedes evitar esto usando `flatMapLatest`.

---

## `observeOn`

```swift
myAPI.countryWithName("spain")
    .flatMap { country in
        myAPI.countriesWithCodes(country.borders)
    }
    .map { countries in
        countries.map { $0.name }
    }
    .observeOn(MainScheduler.instance)
    .subscribeNext { countryNames in
        // Main thread, all good
    }
```

^ Los schedulers abstraen dónde se ejecuta tanto la subscripción, como la observación (dónde llegan los eventos)

^ Existen dos operaciones principales que trabajan con schedulers: `observeOn` y `subscribeOn`.

^ Si quieres recibir los eventos en un scheduler diferente, puedes utilizar `observeOn`.

^ Si quieres que el bloque de subscripción se ejecute en un scheduler diferente, entonces debes utilizar `subscribeOn`.

^ Normalmente, vas a usar más `observeOn`

---

![fit, mute, autoplay, loop](throttle.mov)

^ Un caso de uso muy típico de ReactiveX es el auto-completado.

^ Queremos mostrar resultados a medida que el usuario va escribiendo en el campo de búsqueda, y al mismo tiempo hacer el menor número de peticiones posibles.

---

## `throttle`

```swift
let results = searchBar.rx_text
    .throttle(0.3, scheduler: MainScheduler.instance)
    .flatMapLatest { query in
        if query.isEmpty {
            return Observable.just([])
        }
        
        return searchShows(query)
    }
    .observeOn(MainScheduler.instance)
    .shareReplay(1)
```

^ `throttle`, también llamado `debounce`, sólo va a emitir un elemento de un Observable después de que haya transcurrido un intervalo de tiempo sin que este haya emitido un nuevo elemento.

^ Este código observa un campo de búsqueda, mapeando el último texto escrito a una llamada a la API de búsqueda.

^ Con `shareReplay` nos aseguramos de compartir el side effect entre varios subscriptores.

---

## `combineLatest`

```swift
Observable.combineLatest(emailField.rx_text, passwordField.rx_text)
{ email, password in
    return email.characters.count > 0 &&
    password.characters.count > 0
}
.bindTo(sendButton.rx_enabled)
.addDisposableTo(disposeBag)
```

^ Este código está conmutando el estado `enabled` de un botón, basándose en los contenidos de dos campos: si ambos contienen texto, el botón está habilitado. En caso contrario se deshabita.

^ Esto se consigue combinando las dos secuencias de valores `String` en una única secuencia de valores `Bool`, que se vincula con la propiedad `enabled` del botón.

---

## Follow-up Resources

- reactivex.io
- rxmarbles.com
- github.com/ReactiveX/RxSwift
- tinyurl.com/consuming-web-services

---

## Questions?
## Comments?