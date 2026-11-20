# FileMaker XLSX → JSON

Lee el contenido de un archivo Excel `.xlsx` desde FileMaker Pro y devuelve un JSON con sus hojas, encabezados, filas y celdas.

Utiliza el plugin **BaseElements**. No necesita Microsoft Excel instalado, tablas auxiliares ni campos contenedores. No crea ni modifica registros.

## Características

- Selección de archivo mediante diálogo o ruta recibida como parámetro.
- Lectura de todas las hojas de cálculo, incluidas las vacías.
- Identificación de columnas por letras: `A`, `B`, `C`…
- Conservación de encabezados vacíos o duplicados.
- Valores, tipos OpenXML, índices de estilo y fórmulas disponibles.
- Lectura binaria previa a la descompresión para evitar problemas con acentos en las rutas.
- Gestión de errores y limpieza de archivos temporales.

## Requisitos

- **FileMaker Pro 19.5 o posterior**, en macOS o Windows.
- **BaseElements** instalado y activado en el equipo que ejecuta el guion.
- Para la instalación automatizada, CLI `filemaker` del Agentic Development Toolkit de Claris.

El toolkit no es necesario para ejecutar el lector una vez instalado.

## Archivos del repositorio

| Archivo | Contenido |
|---|---|
| `script.json` | Definición del guion sin identificadores de la base de datos de origen. |
| `install.ndjson` | Operaciones para crear la carpeta y el guion mediante el CLI. |
| `script.txt` | Versión legible de los pasos y cálculos. |
| `example-result.json` | Ejemplo de resultado con datos ficticios. |

## Instalación

Prueba primero en un archivo FileMaker de desarrollo.

Si no existen la carpeta `ADT_scripts` ni el guion:

```sh
filemaker --file="/ruta/Desarrollo.fmp12" \
  --username="TU_CUENTA" --keychain --prompt \
  --dry-run install.ndjson
```

Después de comprobar la validación:

```sh
filemaker --file="/ruta/Desarrollo.fmp12" \
  --username="TU_CUENTA" --keychain --prompt \
  install.ndjson
```

Si la carpeta `ADT_scripts` ya existe, utiliza únicamente la segunda línea de `install.ndjson`.

Estas operaciones crean objetos; no actualizan un guion instalado previamente.

También puedes reproducir los pasos manualmente utilizando `script.txt` como referencia. Este archivo no se puede pegar directamente en el espacio de trabajo de guiones.

## Uso

Ejecuta `UTL_LeerExcelJSON_sp` sin parámetro para abrir el selector:

```text
Ejecutar guion [ “UTL_LeerExcelJSON_sp” ]

Establecer variable [ $_resultado ; Get ( ScriptResult ) ]
Establecer variable [
    $_error ;
    JSONGetElement ( $_resultado ; "errorCode" )
]

Si [ $_error ≠ 0 ]

    Si [ $_error ≠ 1 ]
        Mostrar cuadro de diálogo personalizado [
            "No se pudo leer el Excel" ;
            JSONGetElement ( $_resultado ; "message" )
        ]
    Fin de si

    Salir del guion [ Resultado de texto: $_resultado ]

Fin de si

Establecer variable [
    $_hojas ;
    JSONGetElement ( $_resultado ; "data.hojas" )
]
```

Para indicar un archivo, utiliza este parámetro:

```text
JSONSetElement (
    "{}" ;
    "ruta" ; $_rutaNativa ; JSONString
)
```

La ruta debe ser absoluta y nativa del sistema operativo.

Recoge `Get ( ScriptResult )` inmediatamente después de la llamada. El resultado también se guarda en `$$_EXCEL.RESULTADO_JSON`, que se reemplaza en cada ejecución.

## Ejemplo de resultado

```json
{
  "errorCode": 0,
  "message": "",
  "data": {
    "valoresComoTexto": true,
    "fechasComoSerialExcel": true,
    "hojas": [
      {
        "nombre": "Productos",
        "columnas": {
          "A": "Código",
          "B": "Descripción"
        },
        "filas": [
          {
            "numero": 2,
            "valores": {
              "A": "001",
              "B": "Producto ficticio"
            },
            "tipos": {
              "A": "s",
              "B": "s"
            },
            "estilos": {
              "A": 0,
              "B": 0
            }
          }
        ]
      }
    ]
  }
}
```

La primera fila con contenido se considera cabecera y aparece en `columnas`. Las siguientes aparecen en `filas`.

- `numero`: número original de fila en Excel.
- `valores`: contenido de las celdas como texto.
- `tipos`: tipos originales de OpenXML.
- `estilos`: índices de estilo, sin interpretar su formato visual.
- `formulas`: fórmulas disponibles, cuando existen.

Las celdas ausentes se omiten. Una celda presente sin contenido puede aparecer como texto vacío.

## Errores

| Código | Significado |
|---|---|
| `0` | Lectura correcta. |
| `1` | Selección cancelada por el usuario. |
| Otro | Error: consultar `message` y no utilizar los datos. |

Si falla la lectura o el análisis, no se devuelven hojas parciales. Una lectura correcta puede incluir una advertencia si no se pudo eliminar la carpeta temporal.

## Limitaciones

- Solo admite `.xlsx`, no `.xls` ni libros cifrados con contraseña.
- No recalcula fórmulas: utiliza los resultados guardados en el archivo.
- Las fechas numéricas permanecen como seriales de Excel.
- No aplica formatos visuales: moneda, porcentajes o ceros añadidos mediante formato.
- El análisis XML tiene un límite de 30 segundos; este límite no cubre toda la operación de lectura y descompresión.
- El archivo se carga en memoria.
- Está diseñado para FileMaker Pro; no se ha validado en Server, WebDirect o Go.

## Estado de las pruebas

Probado en FileMaker Pro sobre macOS con varias hojas, hojas vacías, encabezados duplicados, columnas ausentes, textos compartidos y en línea, ceros iniciales, fórmulas y XML inválido.

La lectura binaria se ha comprobado con acentos y caracteres especiales en el archivo y en la carpeta. **La corrección para Windows está incluida, pero su comprobación en un cliente Windows sigue pendiente.**

Todos los ejemplos publicados son ficticios. No se incluyen datos de clientes, credenciales, direcciones de servidores ni bases de datos reales.
