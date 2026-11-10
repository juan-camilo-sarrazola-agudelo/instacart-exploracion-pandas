# Exploración y diagnóstico de calidad de los datos de Instacart con Python y Pandas

Proyecto individual de ciencia de datos – SENA, programa GA1-220501094.  
Aprendiz: **Juan Camilo Sarrazola Agudelo**

| Actividad | Entregable | Versión |
|---|---|---|
| **GA1-220501094-01-AA1-EV04** – Fundamentos en Pandas | `notebooks/01_exploracion_instacart.ipynb` | `v1.1` |
| **GA1-220501094-01-AA1-EV06** – Diagnosticar la calidad de datos | `notebooks/02_diagnostico_calidad.ipynb` + `reports/informe_diagnostico.md` | `v2.0` |

## Descripción y propósito

**Proyecto:** *Explorando Instacart: descubre patrones de compra.*  
Instacart es una plataforma de compras de supermercado. El proyecto busca describir los hábitos de compra de sus clientes
a partir de sus pedidos, para que el equipo de operaciones tome decisiones con información confiable.

**Pregunta analítica:**
> ¿Qué hábitos de compra presentan los clientes de Instacart —cuándo compran, cada cuánto vuelven a comprar y qué productos recompran—?

**Pregunta de la EV06:** ¿qué condiciones de calidad presentan los conjuntos de datos seleccionados y qué acciones de
preparación o limpieza deberían considerarse antes de iniciar su análisis?

## Contenido del repositorio

```
instacart-exploracion-pandas/
├── README.md
├── .gitignore
├── requirements.txt
├── data/                               # CSV de Instacart (no se versionan, ver «Datos»)
│   └── .gitkeep
├── notebooks/
│   ├── 00_entorno.ipynb                # Comprobación del entorno (pandas) y función franja
│   ├── 01_exploracion_instacart.ipynb  # EV04: exploración e indexación con Pandas
│   └── 02_diagnostico_calidad.ipynb    # EV06: carga, inspección y diagnóstico de calidad
└── reports/
    ├── informe_diagnostico.md          # EV06: informe técnico del diagnóstico
    ├── diagnostico_estructural.csv     # Tabla generada por el notebook 02
    └── hallazgos_calidad.csv           # Tabla generada por el notebook 02
```

> La guía propone `data/raw/`. Este proyecto conserva `data/` (usada desde la EV04) para no romper el notebook 01;
> el notebook 02 busca los CSV en `data/raw/` y, si no están, en `data/`, así que funciona con cualquiera de las dos.

## Datos

| Archivo | DataFrame | Filas | Contenido |
|---|---|---|---|
| `instacart_orders.csv` | `orders` | 478.967 | Un registro por pedido: usuario, número de pedido, día, hora y días desde el pedido anterior |
| `products.csv` | `products` | 49.694 | Catálogo de productos con pasillo y departamento |
| `aisles.csv` | `aisles` | 134 | Nombres de los pasillos |
| `departments.csv` | `departments` | 21 | Nombres de los departamentos |
| `order_products.csv` | `order_products` | 4.545.007 | Productos de cada pedido, posición en el carrito y recompra |

Los CSV usan `;` como separador y **no se suben al repositorio** (`data/*.csv` y `data/**/*.csv` están en `.gitignore`)
porque en conjunto pesan más de 100 MB y `order_products.csv` (≈90 MB) supera los 50 MB que GitHub recomienda como máximo por archivo.
Son los archivos del conjunto de datos de Instacart entregados en la formación; para reproducir el trabajo, cópielos en `data/`
(o `data/raw/`) con exactamente estos nombres.

## Reproducir el diagnóstico (EV06)

1. Clonar el repositorio y crear el entorno virtual:

   ```powershell
   git clone https://github.com/juan-camilo-sarrazola-agudelo/instacart-exploracion-pandas.git
   cd instacart-exploracion-pandas
   python -m venv .venv
   .\.venv\Scripts\Activate.ps1          # macOS/Linux: source .venv/bin/activate
   python -m pip install -r requirements.txt
   ```

2. Copiar los cinco CSV en `data/`.
3. Abrir JupyterLab y ejecutar el notebook completo:

   ```powershell
   jupyter lab
   ```

   En `notebooks/02_diagnostico_calidad.ipynb` use *Run → Run All Cells*. También puede ejecutarlo sin abrir Jupyter:

   ```powershell
   jupyter nbconvert --to notebook --execute --inplace notebooks/02_diagnostico_calidad.ipynb
   ```

4. Al terminar, el notebook regenera `reports/diagnostico_estructural.csv` y `reports/hallazgos_calidad.csv`,
   y su última celda imprime una verificación (`OK`) de que se diagnosticaron los cinco datasets.

Dependencias principales: Python 3.x, `pandas` 3.1 y `jupyterlab` (lista completa en `requirements.txt`).

## Resumen del diagnóstico de calidad

| Dataset | Hallazgo principal | Acción propuesta |
|---|---|---|
| `orders` | 28.819 ausentes en `days_since_prior_order` (todos son primeros pedidos), 15 filas duplicadas y tope de 30 días | Mantener NaN como `Int64`, eliminar duplicados, tratar 30 como «30 o más» |
| `products` | 1.258 productos sin nombre (pasillo y departamento `missing`); 141 nombres repetidos con otro formato | Etiquetar «Desconocido», normalizar nombres |
| `aisles` / `departments` | Categorías de relleno `missing` | Excluir o marcar «Sin clasificar» |
| `order_products` | 836 ausentes en `add_to_cart_order` (pedidos de más de 64 productos); 28.906 pedidos sin detalle | `Int64` con NaN; documentar el tipo de unión |

**Conclusión:** los datos son aptos para continuar el proyecto tras una fase de preparación. Detalle completo en
[`reports/informe_diagnostico.md`](reports/informe_diagnostico.md).

## Flujo de ramas y versiones

- `main`: solo versiones estables.
- `develop`: integración.
- `feat/*` y `fix/*`: una rama por fase de trabajo, integrada con `merge --no-ff`.

| Tag | Contenido |
|---|---|
| `v0.1` | Estructura, `.gitignore` y entorno virtual |
| `v0.2` | pandas instalado y `requirements.txt` |
| `v0.3` | JupyterLab en el mismo entorno |
| `v0.4` | Cuaderno `00_entorno` con pandas y `franja` |
| `v0.5` | Cuaderno de la actividad EV04 ejecutado |
| `v1.0` | Entrega EV04: `develop` integrado en `main` |
| `v1.1` | EV04 con salidas guardadas |
| `v2.0` | Entrega EV06: diagnóstico de calidad (rama `feat/ev06-diagnostico-calidad`) |
