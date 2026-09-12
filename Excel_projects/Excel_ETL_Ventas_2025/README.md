# Proyecto ETL en Excel

## Fase 1: Descubrimiento y perfilado de datos

### Conversión inicial

El archivo original `reto_excel_ventas_2025.csv` se convirtió al formato de libro de Microsoft Excel (`ventas_raw_data.xlsx`) para:

- Trabajar con tablas estructuradas.
- Aplicar Power Query y funciones de transformación.
- Facilitar la creación de dashboards y KPI.

### Problemas detectados en los datos en crudo

- La columna `fecha` contenía información de fecha y hora. Se validó el tipo de dato mediante la función `TIPO()` y se confirmó que los valores correspondían a fechas válidas.
- Se creó la columna `BusinessKey`, compuesta por los atributos `fecha`, `producto`, `categoria`, `ciudad`, `cantidad` y `precio_unitario`, para identificar posibles registros duplicados.
- Se confirmó la validez del campo `fecha` con la fórmula `=SI(TIPO(A2)=1,"Número/Fecha","Texto/Inválido")`, la cual indicó que los datos tenían el formato esperado.
- Se identificaron inconsistencias visuales en los nombres: algunos estaban completamente en mayúsculas, otros completamente en minúsculas y no existía una convención de formato uniforme.
- Se agregó la columna auxiliar `producto_valido` para verificar la presencia de espacios normales y del carácter especial 160 en los nombres de productos.
- Se agregó la columna auxiliar `ciudad_valida` para verificar la presencia de espacios normales y del carácter especial 160 en los nombres de ciudades.
- Se verificaron las columnas `cantidad` y `precio_unitario`, evaluando valores cero, negativos, no numéricos y valores potencialmente sospechosos. No se identificaron valores que incumplieran los criterios establecidos para esta revisión.
- Se confirmó mediante filtros que la columna `categoria` contiene únicamente las opciones `Tecnología` y `Hogar`.
- Durante la etapa de exploración y perfilado de los datos en crudo, se identificaron cuatro registros duplicados, correspondientes a dos pares de registros.

## Fase 2: Limpieza y transformación de datos

### Pasos realizados en Power Query

1. **Eliminación de duplicados**

   Se identificaron cuatro registros involucrados en duplicaciones, correspondientes a dos pares de registros duplicados. Se utilizó la columna `ID` como referencia para determinar qué registro conservar: se mantuvo el ID más bajo y se eliminó el ID más alto mediante la función de eliminación de duplicados de Power Query.

2. **Eliminación de columnas auxiliares**

   Se eliminaron las columnas auxiliares de validación y confirmación de inconsistencias de nombres y duplicidad, creadas en Excel: `fecha_valida`, `producto_valido`, `producto_correcto`, `precio_valido` y `cantidad_valida`.

3. **Limpieza y estandarización de productos**

   Se estandarizó la columna `producto`, aplicando mayúscula inicial a cada palabra para mantener una convención de formato uniforme en todos los registros.

4. **Estandarización de ciudades**

   Se estandarizó la capitalización de los nombres de las ciudades y se corrigieron abreviaturas o nombres inconsistentes para mantener una misma convención de formato.

   - `SD` → `Santo Domingo`
   - `la vega` → `La Vega`
   - `Sto Domingo` → `Santo Domingo`
   - `Stgo` → `Santiago`
   - `San Cristobal` → `San Cristóbal`
   - `San Pedro` → `San Pedro de Macorís`

5. **Creación de `nombre_completo`**

   Se creó el campo `nombre_completo` a partir de la concatenación de los campos `nombre` y `apellido`.

6. **Estandarización de `nombre_completo`**

   Se eliminaron espacios innecesarios y caracteres no deseados, y se estandarizó la capitalización del campo `nombre_completo`.

7. **Transformación de fecha**

   Se transformó la columna `fecha` para conservar únicamente el componente de fecha y eliminar la información correspondiente a la hora.

8. **Validación final**

   Se verificó que las transformaciones realizadas produjeran los resultados esperados.

## Fase 3: Modelo de datos

### Estructuración y modelado

#### Estructura del modelo

El modelo se estructuró utilizando un esquema estrella compuesto por una tabla de hechos y cuatro tablas de dimensiones.

#### Tabla de hechos

`Fact_Ventas` contiene una fila por cada transacción de venta y conserva el `ID` original de la fuente para mantener la trazabilidad de las transacciones.

#### Tablas dimensionales

- `Dim_Producto`: identifica los productos y su categoría.
- `Dim_Ciudad`: contiene el catálogo estandarizado de ciudades.
- `Dim_Vendedor`: contiene el catálogo de vendedores.
- `Dim_Fecha`: contiene las fechas únicas y los atributos utilizados para el análisis temporal.

#### Relaciones

Las claves generadas en las tablas dimensionales se utilizan como claves foráneas en `Fact_Ventas`, permitiendo relacionar cada transacción con su producto, ciudad, vendedor y fecha correspondiente.

## Fase 4: Análisis de datos

### Preguntas de negocio

1. ¿Cuánto vendió la empresa en total?
2. ¿Cuántas unidades vendió?
3. ¿Cuál fue el precio promedio de venta?
4. ¿Qué productos generan más ventas?
5. ¿Qué categoría genera más ventas?
6. ¿Qué ciudades generan más ventas?
7. ¿Qué vendedores generan más ventas?
8. ¿Cómo evolucionaron las ventas a través del tiempo?
9. ¿Cuáles fueron los meses con mayores y menores ventas?
10. ¿Qué productos tienen el mayor volumen de unidades vendidas?

### KPI

#### 1. Ventas totales

**Pregunta de negocio:** ¿Cuánto vendió la empresa en total?

**Definición:** Valor total generado por las ventas realizadas durante el período analizado.

**Cálculo:** `Cantidad × Precio_Unitario`, sumado para todas las transacciones.

**Resultado:** RD$572,256,050.00

#### 2. Unidades vendidas

**Pregunta de negocio:** ¿Cuántas unidades vendió la empresa durante el período analizado?

**Definición:** Cantidad total de unidades vendidas durante el período analizado.

**Cálculo:** Suma de la columna `Cantidad` de todas las transacciones de venta.

**Resultado:** 42,714 unidades vendidas.

#### 3. Precio promedio de venta

**Pregunta de negocio:** ¿Cuál fue el precio promedio de venta de los productos durante el período analizado?

**Definición:** Valor promedio del precio unitario registrado en las transacciones de venta durante el período analizado.

**Cálculo:** Promedio de la columna `Precio_Unitario` de todas las transacciones de venta.

**Resultado:** RD$13,390.02

#### 4. Número de transacciones

**Pregunta de negocio:** ¿Cuántas transacciones de venta se realizaron durante el período analizado?

**Definición:** Cantidad total de transacciones de venta registradas durante el período analizado.

**Cálculo:** Recuento de los registros de ventas de la tabla `Fact_Ventas`, utilizando el identificador `ID` de cada transacción.

**Resultado:** 2,000 transacciones.

#### 5. Ticket promedio

**Pregunta de negocio:** ¿Cuál fue el valor promedio de cada transacción durante el período analizado?

**Definición:** Valor promedio generado por cada transacción durante el período evaluado.

**Cálculo:** `Ventas totales ÷ número de transacciones`.

**Resultado:** RD$28,612.80 por transacción.

#### 6. Unidades promedio vendidas por transacción

**Pregunta de negocio:** ¿Cuántas unidades se venden, en promedio, en cada transacción?

**Definición:** Cantidad promedio de unidades incluidas en cada transacción de venta durante el período analizado.

**Cálculo:** `Unidades vendidas ÷ número de transacciones`.

**Resultado:** 2.1385 unidades por transacción.
