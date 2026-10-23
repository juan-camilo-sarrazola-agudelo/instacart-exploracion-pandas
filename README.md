# Exploración de datos de Instacart con Python y Pandas

Actividad **GA1-220501094-01-AA1-EV04 – Fundamentos en Pandas**
*Explorando Instacart: descubre patrones de compra.*

Aprendiz: _escriba aquí su nombre_

## Contenido

| Ruta | Descripción |
|---|---|
| `notebooks/00_entorno.ipynb` | Comprobación del entorno (pandas) y función `franja` |
| `notebooks/01_exploracion_instacart.ipynb` | **Entregable EV04**: los 6 momentos de la actividad |
| `data/` | CSV de Instacart (no se versionan; solo existe `.gitkeep`) |
| `requirements.txt` | Dependencias del entorno (generado con `pip freeze`) |

## Datos

Los CSV usan `;` como separador y **no se suben al repositorio** (`data/*.csv` está en `.gitignore`).
Colócalos en `data/` con estos nombres:

`instacart_orders.csv` · `products.csv` · `aisles.csv` · `departments.csv` · `order_products.csv`

## Reconstruir el entorno

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
jupyter lab
```

En macOS/Linux la activación es `source .venv/bin/activate`.

## Flujo de ramas y versiones

- `main`: solo versiones estables.
- `develop`: integración.
- `feat/*`: una rama por fase de trabajo, integrada con `merge --no-ff`.

| Tag | Contenido |
|---|---|
| `v0.1` | Estructura, `.gitignore` y entorno virtual |
| `v0.2` | pandas instalado y `requirements.txt` |
| `v0.3` | JupyterLab en el mismo entorno |
| `v0.4` | Cuaderno `00_entorno` con pandas y `franja` |
| `v0.5` | Cuaderno de la actividad EV04 ejecutado |
| `v1.0` | Entrega: `develop` integrado en `main` |
