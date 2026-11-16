# Modelos

Este archivo documentara todas las clases y sus metodos de este proyecto.

## `CLI`

### `GetPathJson() -> $str`

### `GetQuery() -> $str`

## `Serializador`

## `ElementoJson` y `NumeroJson`

`ElementoJson` es un emum donde se dicen se hayan los posibles elementos de un json (un array o un objeto). La utilidad de ser un emun es que se pueden contener a si mismo mas facilmente y ademas permite que distiontos struc tengan el mismo metodo y funcione facilmete.

```rust
enum NumeroJson {
    Entero(i128),
    EnteroPositivo(u128),
    Flotante(f64),
}

enum ElementoJson {
    Nulo,
    Booleano(bool),
    Numero(NumeroJson),
    Texto(String),
    Array(Vec<ElementoJson>),
    Objeto(HashMap<String, ElementoJson>),
}
```

### `ElementoJson::Number()`
