# Cheat Sheet PYSPARK 4.1.0
![Texto alternativo](https://github.com/atrigueroshol/PYSPARK/blob/main/pyspark_logo.jpg?raw=true)

## 1. Introducción
PySpark es la interfaz oficial de Apache Spark para el lenguaje de programación Python, que permite desarrollar aplicaciones de procesamiento y análisis de datos a gran escala mediante un modelo de computación distribuida en memoria.

El primer concepto que debemos conocer es SparkSession. Es el punto de entrada principal para trabajar con PySpark. Desde ella se inicializa y se controla el entorno de ejecución de Apache Spark y se accede a todas sus funcionalidades.

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .appName("MiAplicacionPySpark") \
    .getOrCreate()
```

## 2. DataTypes
En PySpark existen los siguientes tipos de datos:

| Columna       | Tipo PySpark | Descripción                               | Rango / Ejemplo                              | Reglas / Notas                                                                 |
|---------------|-------------|-------------------------------------------|---------------------------------------------|-------------------------------------------------------------------------------|
| Numéricos     | ByteType    | Entero de 1 byte                           | -128 a 127                                  | Solo enteros, no admite decimales                                             |
| Numéricos     | ShortType   | Entero de 2 bytes                          | -32,768 a 32,767                            | Solo enteros, no admite decimales                                             |
| Numéricos     | IntegerType | Entero de 4 bytes                          | -2,147,483,648 a 2,147,483,647             | Solo enteros, no admite decimales                                             |
| Numéricos     | LongType    | Entero de 8 bytes                          | -9,223,372,036,854,775,808 a 9,223,372,036,854,775,807 | Solo enteros, no admite decimales                                             |
| Numéricos     | FloatType   | Número decimal de precisión simple         | ±3.4028235e38                               | Precisión limitada, usar para cálculos aproximados                             |
| Numéricos     | DoubleType  | Número decimal de precisión doble          | ±1.7976931348623157e308                     | Más preciso que Float, ideal para cálculos precisos                            |
| Texto         | StringType  | Texto o cadena de caracteres               | "Hola mundo", "123"                          | Puede contener cualquier carácter, longitud ilimitada                          |
| Texto         | BinaryType  | Datos binarios                             | Archivos, imágenes                           | Para datos binarios, no se usa en operaciones de texto                         |
| Booleano      | BooleanType | Valores True o False                        | True / False                                 | No admite 0/1 como booleano; requiere conversión explícita                    |
| Fecha         | DateType    | Fecha sin hora                             | "2026-01-29"                                | Formato `yyyy-MM-dd`; usar funciones de fecha de PySpark                        |
| Fecha         | TimestampType | Fecha y hora                              | "2026-01-29 15:30:00"                        | Formato `yyyy-MM-dd HH:mm:ss[.SSS]`; usar funciones de PySpark                 |
| Estructurado  | ArrayType(elementType, containsNull) | Lista de valores del mismo tipo | [1, 2, 3], ["a", "b"]                         | Todos los elementos deben ser del mismo tipo; puede contener nulls si `containsNull=True` |
| Estructurado  | MapType(keyType, valueType, valueContainsNull) | Diccionario de clave-valor | {"a": 1, "b": 2}                             | Las claves deben ser del mismo tipo; valores pueden ser null si `valueContainsNull=True` |
| Estructurado  | StructType(fields) | Estructura con varios campos (como una fila) | {id:1, name:"Ana"}                            | Cada campo debe tener tipo definido; se usa para definir la estructura de DataFrames |

## 3. Dataframes
Un DataFrame de PySpark es un conjunto distribuido de datos organizados en columnas con nombre y tipo definido, que permite realizar operaciones SQL, transformaciones y análisis a gran escala de manera eficiente mediante procesamiento paralelo. Características clave:
- Distribuido: Los datos pueden estar particionados en múltiples nodos o máquinas.
- Columnas con tipo definido: Cada columna tiene un tipo PySpark (IntegerType, StringType, etc.).
- Inmutable: Las transformaciones crean nuevos DataFrames, no modifican los existentes.
- Optimizado: PySpark usa Catalyst Optimizer para mejorar el rendimiento de las consultas.
- Soporta SQL: Puedes ejecutar consultas SQL directamente sobre DataFrames.

Existen varias formas de crear un dataframe pero la más común es la siguiente:
```python
df = (
    spark.read.format("csv/json/...")
        .option(...)
        .option(...)
        .load(path="/Volumes/.../.../file.csv")
)
```
En la instrucción read.format() se idica el formato de tabla que queremos leer. Existen numerosas opciones. Dependiedo del formato elegido existirán diferentes posibilidades en la instrucción .option(), por ejemplo, epara los ficheros de tipo csv se suele utilizar .option("header", "true") y .option("inferSchema", "true"). Las diferentes posibilidades se pueden consultar en la docuentación oficial de pyspark: https://spark.apache.org/docs/latest/sql-data-sources.html. La instrucción load es la encargada de crear el dataframe y dy en la que se indica la ruta del fichero base.

Otra de las formas más comunes crear una dataframe es desde una tabla de pyspark:
```python
df = spark.table("table_path")
```
Siempre que creamos un dataframe se recomienda definir un schema para evitar errores y acelerar la carga de datos. La definición de los Schemas se hace de la siguiente forma:
```python
from pyspark.sql.types import StringType, LongType, IntegerType, DateType, StructType, StructField

df_schema = StructType([
    StructField("column_name", DateType()),
    StructField("column_name", StringType()),
    StructField("column_name", LongType()),
    StructField("column_name", IntegerType()),
    StructField("column_name", IntegerType())
])
```
Como podemos ver en el ejemplo primero es importar los tipos qu necesitamos para nuestro schema y a continuación defiimos nuestro schema. Debemos incluir el schema en la creación de nuestro dataframe:
```python
df = (
    spark.read.format("csv/json/...")
        .option(...)
        .schema(df_schema)
        .load(path="/Volumes/.../.../file.csv")
)
```
## 4 Conceptos Básicos
## 5 Transformaciones




