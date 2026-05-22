# Desit SPA — Contexto para Claude Code

## Qué es este proyecto
Sistema de gestión interno para **Desit Gamer Store** (RUT 77.839.066-3).
SPA en vanilla JS — un solo archivo `index.html` (~2100 líneas). Sin framework, sin build step.
Deploy en GitHub Pages: https://darsito92.github.io/desit-spa/
Repositorio: https://github.com/darsito92/desit-spa (rama `main`)

## Stack
- HTML/CSS/JS puro en `index.html`
- Firebase Realtime Database v10.7.1 (compat SDK vía CDN)
  - URL: `https://desit-spa-default-rtdb.firebaseio.com/`
  - Path de datos: `desit_spa_data`
  - Reglas: `read/write: true` (acceso público)
- GitHub Pages auto-deploy desde `main`

## Arquitectura clave
- Estado global: objeto `state` sincronizado con Firebase vía `fbRef.set(state)`
- Función de guardado: `S()` — debounce 600 ms, llama a Firebase y `saveLocalBackup()`
- localStorage como fallback offline (`desit_spa_local`)
- Roles: `gerencia`, `finanzas`, `ventas` — cada uno con PIN y tabs habilitados

## Módulos principales
| Página | ID | Función de render |
|---|---|---|
| Dashboard | `page-dash` | `renderDash()` |
| Ventas | `page-ventas` | `renderVentas()` |
| Packs | `page-packs` | `renderPacks()` |
| Inventario | `page-inventario` | `renderInventory()` |
| Finanzas | `page-finanzas` | `renderFinanzas()` |
| Gastos | `page-gastos` | `renderGastos()` |
| Logística | `page-logistica` | `renderLogistica()` |
| Config | `page-config` | `renderConfig()` |

## Cambios importantes ya aplicados

### Bug crítico corregido
- Se eliminó una función `S()` duplicada en la sección "shortcuts" que llamaba a `saveState()` (inexistente), rompiendo silenciosamente TODOS los guardados. La función real `S()` vive cerca de la línea 592.

### Inventario
- Campo `barcode` (Código de barra) agregado a cada producto
- Al editar un producto, el campo N° Factura tiene `<datalist>` con todas las compras existentes para asociar stock a una factura

### Packs — Exportar Cotización
- Botón **📋 Cotización** en el encabezado de cada pack (junto a "✕ Eliminar")
- Genera ventana imprimible con precios **netos** por unidad (precio bruto ÷ 1,19)
- Footer con: subtotal neto → IVA 19% → total final
- Pie de página fijo (`position:fixed;bottom:0` en `@media print`) en **cada hoja**, con:
  - Validez 5 días hábiles
  - Condiciones de armado (4–6 días hábiles, envío adicional)
  - Qué incluye / NO incluye
  - Datos de transferencia bancaria
- Función: `exportCotizacion()` → llama a `printCotizacion(title, content)`

## Flujo de trabajo habitual
1. Editar `index.html`
2. `git add index.html && git commit -m "..."` 
3. `git push origin main`
4. GitHub Pages actualiza en ~1-2 minutos

## Convenciones del código
- Nombres de función cortos: `S()` guarda, `fmt()` formatea moneda, `uid()` genera ID
- `cur()` retorna la moneda activa del selector del header
- `to990(n)` redondea hacia arriba terminando en 990 (ej: 45.230 → 45.990)
- `saleOf(cost, margin)` calcula precio de venta: `cost / (1 - margin/100)`
- `totals(parts)` retorna `{tc, ts, profit, margin}` donde `tc`=costo total, `ts`=venta total
- Para guardar siempre llamar `S()` — nunca escribir directamente a `fbRef`

## Datos de la empresa (para documentos)
- **Desit Gamer Store** · RUT 77.839.066-3
- Banco Chile · Cta. Cte. 166 23169-10
- contacto.desit@gmail.com
