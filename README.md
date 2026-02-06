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
En la instrucción `read.format()` se especifica el formato del archivo que queremos leer. Dependiendo del formato elegido, se pueden utilizar distintas opciones mediante `.option()`. Por ejemplo, en archivos de tipo CSV es común emplear:
```python
.option("header", "true")
.option("inferSchema", "true")
```
Para conocer todas las opciones disponibles según el formato de archivo, puedes consultar la documentación oficial de PySpark: [Data Sources](https://spark.apache.org/docs/latest/sql-data-sources.html).

La instrucción `load()` se encarga de crear el DataFrame y recibe como argumento la ruta del archivo base.

Otra de las formas más comunes crear una dataframe es desde una tabla de pyspark:
```python
df = spark.table("table_path")
```
También existe la posibilidad de crear un dataframe a partir de una lista creada por nosotros mismos. Esta forma de crear dataframes es muy útil para hacer pruebas en nuestro código.
```python
schema = "id int, name string, age short, salary double"

data_list = [(100, "Alberto", 45, 45000),
             (101, "Sergio", 36, 33000),
             (102, "Juan", 48, 28000)]

sample_df = spark.createDataFrame(data=data_list, schema=schema)
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
Como podemos ver en el ejemplo, primero se importan los tipos necesarios para definir nuestro schema y, a continuación, se procede a crearlo. Es importante incluir este schema al momento de crear el DataFrame.
```python
df = (
    spark.read.format("csv/json/...")
        .option(...)
        .schema(df_schema)
        .load(path="/Volumes/.../.../file.csv")
)
```
## 4 Conceptos Básicos
En PySpark existen dos tipos de operaciones:
- Transformaciones: operación que define un nuevo DataFrame o RDD a partir de otro, pero no ejecuta nada todavía. Spark solo construye un plan de ejecución.
- Acciones: operación que ejecuta realmente el plan de Spark y devuelve un resultado concreto (como contar filas, mostrar datos, guardar archivos, etc.).

Uno de los aspectos más importantes de Spark es su evaluación perezosa (_lazy evaluation_). Cuando se realizan transformaciones, Spark no procesa los datos de inmediato, sino que solo las registra en un grafo de ejecución. Es únicamente al ejecutar una acción cuando Spark procesa todas las transformaciones necesarias para producir el resultado

## 5 Transformaciones
En PySpark existen diferentes tipos de transformaciones. Recordemos que, al aplicar una transformación a un DataFrame, se crea un nuevo DataFrame sin modificar el original.

### Creación y modificación de columnas de un DataFrame

Para crear o modificar una columna de un dataframe utilizaremos la función **withColumn()**.
``` python
new_df = old_df.withColumn("newcolumn", expr("SQL expression"))
```
Para crear o modificar varias columnas de un dataframe utilizaremos **withColumns()**
```python
from pyspark.sql.functions import expr

new_df = (
    old_df.withColumns({
        "newcolumn": expr("SQL expression") or PySpark condition,
        "modifycolumn": expr("SQL expression") or PySpark condition
    })
)
```
Otra de las formas de crear una nueva columna o modificar las ya existentes es utilizando **selectExpr()**.  Esta función solo mantiene en el DataFrame las columnas especificadas en ella.
```python
new_df = (
	old_df.selectExpr(
		"old_column as new_column_1",
		"SQL expresion old_column as new_column_2"
	)
)
```

*En las expresiones solo se puede hacer referencia a una columna nueva si se esta creando una columna nueva, si se utiliza para modificar una columna ya existente dará error.*

### Renombrar columnas de un Dataframe
Para renombrar una o varias columnas utilizaremos **withColumnsRenamed()**
``` python
new_df = (
    old_df.withColumnsRenamed({
        "oldcolumn_1": "newcolumn_1",
        "oldcolumn_2": "newcolumn_2"
    })
)
```
### Eliminar columnas de un Dataframe
Para eliminar un columna utilizaremos la funcion **drop()**
```python
new_df = old_df.drop("column")
```
### Filtrar columnas de un Dataframe
Para filtrar columnas de un dataframe se utilizan las funciones **filter()** o **where()**. Ambas funciones hacen lo mismo.
```python
new_df = old_df.filter(column condition)
```
### Eliminar Duplicados
Si se quieren eliminar todos los duplicados del DataFrame, se debe utilizar la función **distinct**().
```python
new_df = old_df.distinct()
```
Si se quieren eliminar duplicados especificando las columnas, se debe utilizar la función **dropDuplicates()**.
```python
new_df = old_df.dropDuplicates(["column_1", "column_2"])
```
En este caso, si en new_df hay dos registros con los mismos valores en column_1 y column_2, pero con valores distintos en column_3, la función conservará uno de ellos, sin garantizar cuál, a menos que el DataFrame esté previamente ordenado (se queda con el primero que encuentra).

### Ordenar los registros
Para ordenar los registros se utiliza la función orderBy(). Por defecto ordena de forma scendiente.
```python
new_df = old_df.orderBy("column_1", ascending=True/False)
```
### Limitar el número de registros
Para restringir el número de registros del dataframe se utilizá la función **limit()**.
```python
new_df = old_df.limit(n)
```
### Tipos de Joins
Para hacer joins en pyspark debemos cargar los dataframes que queremos unir y asignarles un alias. Además debemos definir mediante una expresión los campos por los que realizar la unión. La expresión se puede escribir utilizando col() o expr().
```python
from pyspark.sql.functions import expr, col

df_1= spark.table("path_1").alias("m")
df_2= spark.table("path_1").alias("b")

join_expr =  col("m.column ") ==  col("b.column")
```
#### INNER JOIN
```python
new_df = df_1.join(df_2, join_expr, "inner")
```
#### OUTER JOINS
```python
new_df = df_1.join(df_2, join_expr, "left")
```
```python
new_df = df_1.join(df_2, join_expr, "right")
```
```python
new_df = df_1.join(df_2, join_expr, "full")
```
### Agregaciones
PySpark tiene muchas funciones de agregación y las podemos encontrar en la documentación oficial [funciones de agregación](https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/grouping.html).

Ejemplo de funciones de agregación simple:

``` python
from pyspark.sql import functions as F

new_df = old_df.select(
    F.sum("column_1").alias("total_column_1"),
    F.avg("column_2").alias("media_column_2"),
    F.max("column_3").alias("max_column_3"),
    F.min("column_4").alias("min_column_4"),
    F.count("column_5").alias("num_column_5")
)
```
Lo más común es combinar funciones de agregación con groupBy(). 
```python
new_df = old_df.groupBy("column_1").agg(
    F.sum("column_2").alias("total_column_2")
)
```
## 5 Acciones

Las acciones son operaciones disparan la ejecución del DAG (las transformaciones). Las acciones más utlizadas son las siguientes:

#### Display()
Muestra los datos de un DataFrame de manera visual y tabular. Sirve para inspeccionar los datos rápidamente en un notebook.
```python
df.display()
```
#### Collect()
Devuelve todos los registros de un RDD o DataFrame al driver. Sirve para la inspección completa de los datos. **Se debe tener cuidado con los datos de gran tamaño**.
```python
df.collect()
```
#### Take()
Devuelve los n primeros elementos al driver. Sirve para explorar parte del dataset
```python
df.take(n)
```
#### First()
Devuelve el primer elemento del dataframe.
```python
df.first()
```
#### Count()
Devuelve el número de resgistros del dataframe.
```python
df.count()
```

