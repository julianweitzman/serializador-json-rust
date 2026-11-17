# Modelos

Este archivo documenta las estructuras, clases, enums y sus métodos del proyecto.

El proyecto utiliza un enum `Error` compartido para representar los posibles errores. Cada variante contiene un campo `mensaje: String` que describe el problema ocurrido. Los métodos que pueden fallar utilizan `Result<T, Error>` para devolver el resultado o el error correspondiente.

```rust
pub enum Error {
    // CLI
    ArgumentoDesconocido { mensaje: String },
    ArgumentoDuplicado { mensaje: String },
    ValorArgumentoFaltante { mensaje: String },
    ArgumentoObligatorioFaltante { mensaje: String },
    ArgumentoVacio { mensaje: String },

    // Archivos
    ArchivoNoEncontrado { mensaje: String },
    PermisoDenegado { mensaje: String },
    RutaInvalida { mensaje: String },
    ErrorLectura { mensaje: String },
    ErrorEscritura { mensaje: String },
    CodificacionInvalida { mensaje: String },
    OperacionCancelada { mensaje: String },

    // JSON
    JsonInvalido { mensaje: String },
    NumeroFueraDeRango { mensaje: String },
    NumeroNoFinito { mensaje: String },
    ClaveObjetoInvalida { mensaje: String },
}
```

## `CLI`

Se encarga de recibir e interpretar los argumentos proporcionados por el usuario al ejecutar el programa. Permite indicar el archivo JSON que se quiere consultar, la consulta que se desea realizar y, opcionalmente, dónde guardar el resultado.

### `get_args() -> Vec<String>`

Obtiene los argumentos proporcionados al ejecutar el programa y los devuelve en un vector de cadenas de texto.

### `process_args(args: Vec<String>) -> Result<Args, Error>`

Interpreta los argumentos recibidos y construye un objeto `Args` con los valores correspondientes. Identifica los argumentos `-p`, `-q` y `-s`, y asigna sus respectivos valores.

Devuelve un error si los argumentos son inválidos, faltan parámetros obligatorios, se repiten o existe algún problema en su estructura.

## `Args`

Estructura que almacena los argumentos ya procesados de la CLI.

Sus campos son:

- `path: String`: ruta del archivo JSON que se quiere procesar.
- `query: String`: consulta que se quiere realizar sobre el contenido del archivo.
- `save: Option<String>`: ruta donde se guardará el resultado. Si no se proporciona el argumento `-s`, su valor será `None`.

## `ControladorArchivos`

Se encarga de gestionar la lectura y escritura de archivos. Centraliza estas operaciones para evitar que otras partes del programa tengan que encargarse directamente del acceso al sistema de archivos.

### `leer(path: String) -> Result<String, Error>`

Lee el contenido completo del archivo indicado por `path` y lo devuelve como una cadena de texto.

Devuelve un error si el archivo no existe, no se tienen los permisos necesarios, la ruta es inválida o ocurre algún problema durante su lectura o decodificación.

### `escribir(path: String, texto: String) -> Result<(), Error>`

Escribe el contenido de `texto` en el archivo indicado por `path`.

Si el archivo ya existe, solicita confirmación antes de sobrescribirlo. Si no existe, solicita confirmación antes de crearlo.

Devuelve un error si la ruta es inválida, no se tienen los permisos necesarios, ocurre algún problema durante la escritura o el usuario cancela la operación.

## `Serializador`

Se encarga de convertir las estructuras `ElementoJson` a su representación textual en formato JSON. Recorre recursivamente los elementos de las estructuras complejas para construir el resultado.

### `convertir(json: ElementoJson) -> Result<String, Error>`

Recibe un elemento JSON y devuelve su representación textual. Si recibe un array o un objeto, serializa recursivamente sus elementos internos y combina los resultados para construir el JSON completo.

Devuelve un error si encuentra un valor que no puede representar correctamente en formato JSON, como un número no finito o una clave de objeto que no sea de tipo `ElementoJson::Texto`.

## `Deserializador`

Se encarga de analizar una cadena de texto en formato JSON y convertirla en una estructura de datos `ElementoJson`. Su implementación realiza el análisis del texto desde cero, identificando los distintos tipos de valores y sus estructuras.

### `convertir(json: String) -> Result<ElementoJson, Error>`

Recibe una cadena de texto en formato JSON y devuelve su representación mediante `ElementoJson`. Reconoce valores nulos, booleanos, numéricos, cadenas, arrays y objetos, incluyendo estructuras anidadas.

Devuelve un error si el texto no cumple con la sintaxis JSON, contiene números fuera de los rangos representables o presenta algún otro problema durante el análisis.

## `ElementoJson` y `Numero`

`ElementoJson` es un enum que representa los posibles tipos de valores que puede contener un JSON. Al ser un enum recursivo, permite representar estructuras anidadas, como arrays que contienen otros arrays u objetos que contienen otros objetos.

Además, permite que distintos tipos de datos compartan una misma interfaz de representación y serialización.

`Numero` es un enum auxiliar que permite representar los distintos tipos numéricos de JSON sin perder precisión. Se divide en tres variantes:

- `Entero(i128)`: almacena números enteros con signo.
- `EnteroPositivo(u128)`: almacena números enteros sin signo.
- `Flotante(f64)`: almacena números de punto flotante.

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
