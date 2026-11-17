# Modelos

Este archivo documenta las estructuras, clases y sus métodos del proyecto.

## `CLI`

Se encarga de recibir e interpretar los argumentos proporcionados por el usuario al ejecutar el programa. Permite indicar el archivo JSON que se quiere consultar, la consulta que se desea realizar y, opcionalmente, dónde guardar el resultado.

### `GetPathJson() -> String`

Obtiene la ruta del archivo JSON que se quiere procesar. Corresponde al argumento `-p`.

### `GetQuery() -> String`

Obtiene la consulta que se quiere realizar sobre el contenido del archivo JSON. Corresponde al argumento `-q`.

### `GetPathSalida() -> Option<String>`

Obtiene la ruta donde se guardará el resultado. Corresponde al argumento opcional `-s`. Si no se proporciona, devuelve `None` y el resultado se muestra únicamente por consola.

## `ControladorArchivos`

Se encarga de gestionar la lectura y escritura de archivos. Centraliza estas operaciones para evitar que otras partes del programa tengan que encargarse directamente del acceso al sistema de archivos.

### `Leer(path: String) -> String`

Lee el contenido completo del archivo indicado por `path` y lo devuelve como una cadena de texto.

### `Escribir(path: String, texto: String) -> ()`

Escribe el contenido de `texto` en el archivo indicado por `path`. Si el archivo ya existe, solicita confirmación antes de sobrescribirlo. Si no existe, solicita confirmación antes de crearlo.

## `Serializador`

Se encarga de convertir las estructuras `ElementoJson` a su representación textual en formato JSON. Recorre recursivamente los elementos de las estructuras complejas para construir el resultado.

### `Convertir(json: ElementoJson) -> String`

Recibe un elemento JSON y devuelve su representación textual. Si recibe un array o un objeto, serializa recursivamente sus elementos internos y combina los resultados para construir el JSON completo.

## `Deserializador`

Se encarga de analizar una cadena de texto en formato JSON y convertirla en una estructura de datos `ElementoJson`. Su implementación realiza el análisis del texto desde cero, identificando los distintos tipos de valores y sus estructuras.

### `Convertir(json: String) -> ElementoJson`

Recibe una cadena de texto en formato JSON y devuelve su representación mediante `ElementoJson`. Reconoce valores nulos, booleanos, numéricos, cadenas, arrays y objetos, incluyendo estructuras anidadas.

## `ElementoJson` y `Numero`

`ElementoJson` es un enum que representa los posibles tipos de valores que puede contener un JSON. Al ser un enum recursivo, permite representar estructuras anidadas, como arrays que contienen otros arrays u objetos que contienen otros objetos.

Además, permite que distintos tipos de datos compartan una misma interfaz de representación y serialización.

`Numero` es un enum auxiliar que permite representar los distintos tipos numéricos de JSON sin perder precisión. Se divide en tres variantes:

- `Entero(i128)`: almacena números enteros con signo.
- `EnteroPositivo(u128)`: almacena números enteros sin signo.
- `Flotante(f64)`: almacena números decimales.

Esta separación es necesaria porque `f64` no puede representar exactamente todos los enteros de 128 bits. Al conservar los enteros en sus tipos originales, se evitan pérdidas de precisión durante la deserialización y serialización.

Las estructuras se definen de la siguiente manera:

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
    Objeto(Vec<(ElementoJson, ElementoJson)>),
}
```

### `ElementoJson::Nulo`

Representa el valor `null` de JSON. No almacena ningún dato adicional.

### `ElementoJson::Booleano(bool)`

Representa un valor booleano. Almacena `true` o `false`.

### `ElementoJson::Numero(Numero)`

Representa un valor numérico. Utiliza el enum `Numero` para distinguir entre enteros con signo, enteros sin signo y números de punto flotante.

### `ElementoJson::Texto(String)`

Representa una cadena de texto. Almacena su contenido como un `String` de Rust.

### `ElementoJson::Array(Vec<ElementoJson>)`

Representa un array JSON. Almacena una colección ordenada de elementos `ElementoJson`, permitiendo contener cualquier tipo de valor JSON, incluso otros arrays y objetos.

### `ElementoJson::Objeto(Vec<(ElementoJson, ElementoJson)>)`

Representa un objeto JSON. Almacena sus propiedades como un vector de tuplas, donde cada tupla contiene una clave y su correspondiente valor.

Las claves se representan mediante `ElementoJson::Texto` y los valores pueden ser cualquier variante de `ElementoJson`.

Se utiliza `Vec` en lugar de `HashMap` para conservar el orden de las propiedades y mantener una estructura uniforme para las claves y los valores.

### `to_string() -> String`

Convierte un valor `ElementoJson` en su representación textual JSON.

Para los valores simples, devuelve directamente su representación correspondiente. En el caso de las estructuras complejas, como arrays y objetos, serializa recursivamente sus elementos internos y combina los resultados.

Las cadenas de texto deben escaparse correctamente para respetar el formato JSON, incluyendo comillas, barras invertidas y caracteres de control.

En el caso de los objetos, también convierte las claves a su representación textual antes de combinarlas con sus respectivos valores.