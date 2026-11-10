# Informe técnico – Diagnóstico inicial de calidad de los datos de Instacart

**Actividad:** GA1-220501094-01-AA1-EV06 – Diagnosticar la calidad de datos: preparación, exploración y documentación  
**Aprendiz:** Juan Camilo Sarrazola Agudelo  
**Notebook de soporte:** [`notebooks/02_diagnostico_calidad.ipynb`](../notebooks/02_diagnostico_calidad.ipynb)  
**Tablas generadas por el notebook:** [`diagnostico_estructural.csv`](diagnostico_estructural.csv) · [`hallazgos_calidad.csv`](hallazgos_calidad.csv)

> Todas las cifras de este informe provienen de la ejecución del notebook con pandas 3.1. Si los datos cambian,
> se debe volver a ejecutar el notebook y actualizar este documento con las tablas CSV que genera.

---

## 1. Nombre del proyecto y pregunta analítica

**Proyecto:** *Explorando Instacart: descubre patrones de compra.*

**Propósito:** describir los hábitos de compra de los clientes de Instacart a partir de sus pedidos, para que el equipo
de operaciones tome decisiones con información confiable.

**Pregunta analítica (recuperada de la EV04):**
> ¿Qué hábitos de compra presentan los clientes de Instacart —cuándo compran, cada cuánto vuelven a comprar y qué productos recompran—?

Se concreta en tres preguntas: (1) cada cuántos días vuelven a comprar, (2) qué proporción de los artículos son recompras y
(3) en qué días y horas se concentran los pedidos de los clientes recurrentes.

**Pregunta orientadora de la actividad:** ¿qué condiciones de calidad presentan los conjuntos de datos y qué acciones de
preparación o limpieza deberían considerarse antes de iniciar su análisis?

## 2. Descripción de los datasets utilizados

Los cinco archivos son CSV separados por punto y coma (`;`). Cada uno se cargó en un DataFrame independiente.

| Archivo | DataFrame | Contenido | Relación con la pregunta |
|---|---|---|---|
| `instacart_orders.csv` | `orders` | Un registro por pedido: usuario, número de pedido, día de la semana, hora y días desde el pedido anterior. | *Cuándo* compran y *cada cuánto* vuelven. |
| `products.csv` | `products` | Catálogo: identificador, nombre, pasillo y departamento. | *Qué* productos se compran. |
| `aisles.csv` | `aisles` | Tabla de consulta de pasillos. | Agrupa productos por pasillo. |
| `departments.csv` | `departments` | Tabla de consulta de departamentos. | Agrupa productos por departamento. |
| `order_products.csv` | `order_products` | Un registro por producto dentro de un pedido: posición en el carrito y si es recompra. | *Qué* se recompra; une pedidos con productos. |

Relaciones: `orders` ↔ `order_products` por `order_id`; `products` ↔ `order_products` por `product_id`;
`products` ↔ `aisles` por `aisle_id`; `products` ↔ `departments` por `department_id`.

## 3. Resultados de la inspección estructural

A cada DataFrame se le aplicaron `head()`, `tail()`, `shape`, `columns`, `dtypes`, `info()`, `describe()` y
`describe(include='object')` (este último solo donde hay columnas de texto).

| Dataset | Filas | Columnas | Tipos de datos | Observaciones |
|---|---|---|---|---|
| `orders` | 478.967 | 6 | int64 ×5, float64 ×1 | `days_since_prior_order`: 28.819 ausentes (6,02 %) y enteros guardados como float64; 15 filas duplicadas. |
| `products` | 49.694 | 4 | int64 ×3, str ×1 | `product_name`: 1.258 ausentes (2,53 %). |
| `aisles` | 134 | 2 | int64 ×1, str ×1 | Sin ausentes ni duplicados; incluye el pasillo `missing`. |
| `departments` | 21 | 2 | int64 ×1, str ×1 | Sin ausentes ni duplicados; incluye el departamento `missing`. |
| `order_products` | 4.545.007 | 4 | int64 ×3, float64 ×1 | `add_to_cart_order`: 836 ausentes (0,02 %) y enteros guardados como float64. |

**Características principales**
- `orders`: 478.967 pedidos de **157.437** usuarios distintos (una fila es un pedido, no un cliente). Horas de 0 a 23, días de la semana codificados de 0 a 6, `order_number` de 1 a 100 y `days_since_prior_order` de 0 a 30.
- `products`: los 48.436 nombres disponibles son distintos entre sí; `aisle_id` va de 1 a 134 y `department_id` de 1 a 21.
- `order_products`: el dataset más grande (≈145 MB en memoria). Abarca 450.046 pedidos y 45.573 productos; el 59,05 % de los artículos son recompras (`reordered` = 1).
- **Nota técnica:** con pandas 3 las columnas de texto se cargan con el tipo `str` (antes `object`). `describe(include='object')` las sigue incluyendo por compatibilidad.

## 4. Valores faltantes, tipos de datos y posibles inconsistencias

### 4.1 Valores faltantes

| Columna | Ausentes | Causa encontrada | Efecto en el análisis |
|---|---|---|---|
| `orders.days_since_prior_order` | 28.819 (6,02 %) | Todos tienen `order_number` = 1: es el primer pedido del cliente y no existe pedido anterior. Ausente estructural, no error. | Si se rellena con 0, el promedio de días entre compras baja artificialmente. |
| `products.product_name` | 1.258 (2,53 %) | Todos están en el pasillo 100 y el departamento 21, ambos llamados `missing`. Ese pasillo solo contiene productos sin nombre. | 9.547 artículos comprados (0,21 %) no se pueden identificar ni clasificar. |
| `order_products.add_to_cart_order` | 836 (0,02 %) | Se concentran en 70 pedidos de entre 65 y 127 productos; la posición máxima registrada es 64. Límite del sistema de registro. | Mínimo: solo impide estudiar el orden en carritos muy grandes. |

En total, solo el **0,15 %** de las celdas de los cinco datasets (30.913 de 21.252.916) está vacío.

### 4.2 Tipos de datos

De 18 columnas, 14 tienen un tipo coherente con lo que representan y 4 son parcialmente coherentes:
- `days_since_prior_order` y `add_to_cart_order` representan **enteros**, pero son `float64` porque contienen `NaN` (por eso se ven valores como `30.0`). Conviene `Int64`, que admite nulos.
- `order_dow` es un **código de día** (0–6) almacenado como número: su promedio no tiene interpretación y el archivo no indica qué día corresponde a cada código.
- `reordered` solo contiene 0 y 1: es una variable **binaria** que puede convertirse a `bool`.
- Ninguna columna numérica quedó como texto, lo que confirma que el separador se aplicó correctamente.

### 4.3 Posibles inconsistencias

- **Pedidos duplicados:** 15 filas de `orders` son idénticas en sus 6 columnas. Todas tienen `order_dow` = 3 y `order_hour_of_day` = 2; de los 121 pedidos de esa franja, 15 son copias. La concentración apunta a un fallo puntual del registro. Ninguna otra tabla tiene duplicados ni claves repetidas.
- **Tope en `days_since_prior_order`:** el valor 30 aparece 51.338 veces (11,40 % de los valores disponibles) frente a 2.673 veces el valor 29. El salto indica que 30 significa «30 días o más». Además, `order_number` llega a 100 en 194 pedidos, lo que podría ser otro límite del historial.
- **Nombres de productos con formato inconsistente:** 141 nombres se repiten al ignorar mayúsculas y espacios (por ejemplo, el mismo nombre en MAYÚSCULAS y en tipo título, con `product_id` distintos); 54 nombres están completamente en mayúsculas, 403 tienen espacios dobles y 4 tienen espacios al inicio o al final.
- **Categorías de relleno:** el pasillo `missing` (id 100) y el departamento `missing` (id 21) no describen productos reales.
- **Pedidos sin detalle:** 28.906 pedidos de `orders` (6,04 %) no aparecen en `order_products`. Ninguno es un primer pedido (su `order_number` mínimo es 4), por lo que `order_products` parece ser una muestra parcial del detalle.

**Lo que está bien:** todas las reglas de validez se cumplen (horas 0–23, días 0–6, `reordered` solo 0/1, posiciones ≥ 1, días entre 0 y 30),
la ausencia de días coincide exactamente con los primeros pedidos, no hay recompras en primeros pedidos y todas las claves
foráneas (`order_id`, `product_id`, `aisle_id`, `department_id`) apuntan a registros existentes.

### 4.4 Limitaciones frente a la pregunta analítica

| Limitación | Parte de la pregunta afectada |
|---|---|
| No hay fechas reales, solo día de la semana y hora. | No se pueden estudiar meses, temporadas ni festivos. |
| `order_dow` no tiene diccionario de días. | *Cuándo* compran: no se puede afirmar qué día es el 0. |
| Tope de 30 en `days_since_prior_order`. | *Cada cuánto* vuelven: el promedio subestima a los clientes que tardan más de un mes. |
| `order_products` no cubre el 6,04 % de los pedidos. | *Qué recompran*: el análisis se basa en una muestra. |
| 1.258 productos sin nombre ni categoría. | *Qué recompran*: 9.547 artículos sin identificar. |
| No hay precios, cantidades ni datos de clientes. | No se puede medir gasto ni segmentar clientes. |

## 5. Principales hallazgos de calidad por dataset

| Dataset | Problema identificado | Evidencia encontrada (Pandas) | Posible impacto | Acción propuesta |
|---|---|---|---|---|
| orders | Valores faltantes en `days_since_prior_order` | 28.819 NaN (6,02 %); todos con `order_number` = 1 | Rellenar con 0 distorsiona el promedio de días entre compras | Mantener NaN, convertir a `Int64` y crear la marca `es_primer_pedido` |
| orders | Filas duplicadas | 15 filas idénticas, todas con `order_dow` = 3 y hora = 2 | Conteos inflados; `order_id` deja de ser clave única | `drop_duplicates()` y verificar unicidad de `order_id` |
| orders | Tope en `days_since_prior_order` | Valor 30: 51.338 pedidos frente a 2.673 con 29 | El promedio subestima el tiempo entre compras | Tratar 30 como «30 o más» y usar la mediana |
| orders | Código de día sin diccionario | `order_dow` con valores 0–6 sin etiquetas | No se puede afirmar qué día compran más | Confirmar el diccionario con la fuente; convertir a categoría ordenada |
| orders + order_products | Pedidos sin detalle | 28.906 pedidos (6,04 %) no están en `order_products` | El análisis de productos usa solo una parte de los pedidos | Elegir y documentar el tipo de unión (inner/left) |
| products | Valores faltantes en `product_name` | 1.258 NaN, todos en pasillo 100 y departamento 21 (`missing`) | 9.547 artículos sin identificar | Etiquetar como «Desconocido»; excluir de rankings por nombre y categoría |
| products | Nombres repetidos con distinto formato | 141 repetidos al ignorar mayúsculas/espacios; 403 con espacios dobles | Ventas de un producto repartidas en varias filas | Normalizar texto y revisar repetidos; agrupar por `product_id` |
| aisles / departments | Categorías de relleno | Pasillo `missing` (100) y departamento `missing` (21) | Aparecen como una categoría más en análisis por grupo | Excluir o etiquetar como «Sin clasificar» |
| order_products | Valores faltantes en `add_to_cart_order` | 836 NaN en 70 pedidos de más de 64 productos | Mínimo; impide analizar el orden en carritos muy grandes | `Int64`, conservar NaN y documentar el límite de 64 |
| orders / order_products | Tipos no óptimos | `float64` para enteros; `reordered` 0/1 en `int64` | Valores como `30.0`; más memoria y menor claridad | `Int64`, `bool` y `category` según corresponda |

## 6. Acciones de limpieza y preparación propuestas

Las acciones no se ejecutaron en esta actividad (el propósito era diagnosticar). Se proponen, en orden, para la siguiente fase:

1. Eliminar los 15 pedidos duplicados de `orders` con `drop_duplicates()` y comprobar que `order_id` quede único.
2. Mantener como `NaN` los ausentes de `days_since_prior_order`, convertir la columna a `Int64` y crear `es_primer_pedido`.
3. Conservar los productos sin nombre para conteos, etiquetarlos como «Desconocido» y excluirlos de rankings y análisis por categoría.
4. Convertir `add_to_cart_order` a `Int64`, conservar los `NaN` y documentar el límite de 64 posiciones.
5. Crear una versión normalizada de `product_name` (minúsculas, sin espacios dobles ni extremos), revisar los 141 repetidos y agrupar por `product_id` mientras tanto.
6. Convertir `reordered` a `bool`; `order_dow` a categoría ordenada una vez confirmado el diccionario; `aisle` y `department` a `category`.
7. Interpretar el valor 30 de `days_since_prior_order` como «30 o más», reportar su proporción y preferir la mediana al promedio.
8. Decidir y documentar el tipo de unión entre `orders` y `order_products`.

## 7. Conclusiones sobre las condiciones actuales de los datos

1. **Los datos son aptos para continuar el proyecto.** Los cinco archivos cargan sin errores, las claves entre tablas son íntegras, los rangos son válidos y solo el 0,15 % de las celdas está vacío.
2. **Los problemas son pocos y explicables:** los ausentes de días son primeros pedidos, los de nombre son productos sin clasificar y los del carrito son un límite del registro. Ninguno obliga a descartar los datos.
3. **Se requiere una fase de preparación antes del análisis:** eliminar duplicados, ajustar tipos, normalizar nombres y decidir el tratamiento de los productos `missing` y de los pedidos sin detalle.
4. **La pregunta analítica puede responderse, con matices:** el *cuándo* depende de confirmar el diccionario de días, el *cada cuánto* debe leerse considerando el tope de 30 días y el *qué recompran* se basa en los 450.046 pedidos que tienen detalle.
