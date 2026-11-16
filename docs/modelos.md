# Modelos

Este archivo documentara todas las clases y sus metodos de este proyecto.

## `CLI`

### `GetPathJson() -> String`

### `GetQuery() -> String`

## `ControladorArchivos`

### `Leer(path: $String) -> String`

### `Escribir(path: $String, texto: $String) -> ()`

## `Serializador`

### `Convertir(json: ElementoJson) -> String`

## `Deserializador`

### `Convertir(json: $String) -> ElementoJson`

## `ElementoJson` y `Numero`

`ElementoJson` es un emum donde se dicen se hayan los posibles elementos de un json (un array o un objeto). La utilidad de ser un emun es que se pueden contener a si mismo mas facilmente y ademas permite que distiontos struc tengan el mismo metodo y funcione facilmete.

`Numero` esta solo para poder almacenar todos los tipos numericos ya que `f64` se queda corto para `i128` y `u128`.

```rust
enum Numero {
    Entero(i128),
    EnteroPositivo(u128),
    Flotante(f64),
}

enum ElementoJson {
    Nulo,
    Booleano(bool),
    Numero(Numero),
    Texto(String),
    Array(Vec<ElementoJson>),
    Objeto(HashMap<String, ElementoJson>),
}
```

### `ElementoJson::Nulo`

### `ElementoJson::Booleano(bool)`

### `ElementoJson::Numero(Numero)`

### `ElementoJson::Texto(String)`

### `ElementoJson::Array(Vec<ElementoJson>)`

### `ElementoJson::Objeto(HashMap<String, ElementoJson>)`
