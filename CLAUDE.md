# Desit SPA — Contexto para Claude Code

## Qué es este proyecto
Sistema de gestión interno para **Desit Gamer Store** (RUT 77.839.066-3).
SPA en vanilla JS — un solo archivo `index.html` (~2100 líneas). Sin framework, sin build step.
Deploy en GitHub Pages: https://darsito92.github.io/desit-spa/
Repositorio: https://github.com/darsito92/desit-spa (rama `main`)

## Stack
- HTML/CSS/JS puro en `index.html`
- **Supabase** (migrado desde Firebase) — PostgreSQL + Realtime vía CDN `@supabase/supabase-js@2`
  - Project URL: `https://xkywixwsovybcsisubrf.supabase.co`
  - Tabla: `app_data` — columna `id int PK` + `state jsonb` (fila única id=1)
  - Las credenciales NO están en el código — se guardan en `localStorage` del navegador (`sb_url`, `sb_key`)
- GitHub Pages auto-deploy desde `main`

## Arquitectura clave
- Estado global: objeto `state` sincronizado con Supabase vía `sbClient.from('app_data').upsert({id:1,state})`
- Función de guardado: `S()` — debounce 600 ms, llama a Supabase y `saveLocalBackup()`
- localStorage como fallback offline (`desit_spa_local`)
- Realtime: `sbChannel` escucha `postgres_changes` en `app_data` para sincronizar entre PCs
- Usuarios nombrados: `diego` (gerencia), `veronica` (finanzas), `alexis` (gerencia) — cada uno con contraseña propia en `state.users`
- Roles: `gerencia` (todos los tabs), `finanzas` (todos excepto Config) — rol `ventas` eliminado
- Verónica (finanzas) puede agregar/eliminar tiendas
- `logAction(desc)` registra movimientos en `state.historial` con timestamp y usuario activo
- `currentUser` guarda el nombre del usuario logueado; `currentRole` su rol

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

### Migración Firebase → Supabase
- SDK de Firebase eliminado, reemplazado por `@supabase/supabase-js@2`
- `sbClient` reemplaza a `fbRef` en todas las operaciones
- Modal de configuración pide solo Project URL + anon key (2 campos vs 7 de Firebase)
- Credenciales guardadas en localStorage, nunca en el código fuente
- `exportDashJSON()` descarga directo desde Supabase para garantizar datos completos

### Login por usuario (reemplaza PINs por rol)
- Pantalla de login con 3 tarjetas: Diego / Verónica / Alexis
- Contraseña individual por usuario (default: `1234`), almacenada en `state.users`
- `state.users = { diego:{pwd,role}, veronica:{pwd,role}, alexis:{pwd,role} }`

### Historial de movimientos
- `logAction(desc)` registra en `state.historial[]` — máximo 500 entradas
- Se registra: login/logout, ventas completadas, packs, inventario, compras, gastos, contraseñas
- Visible en Config (solo gerencia) con botón limpiar

### Inventario
- Campo `barcode` (Código de barra) agregado a cada producto
- Al editar, campo N° Factura tiene `<datalist>` con compras existentes para asociar stock a factura
- Descuento de stock solo al marcar venta como **completada** (no al seleccionar del autocompletado)

### Packs — Exportar Cotización
- Botón **📋 Cotización** en el encabezado de cada pack
- Precios **netos** por unidad (precio bruto ÷ 1,19) + IVA 19% al pie
- Pie de página fijo en cada hoja impresa (`position:fixed;bottom:0` en `@media print`):
  - Validez 5 días hábiles · Condiciones de armado · Qué incluye/NO incluye · Datos de transferencia
- Función: `exportCotizacion()` → `printCotizacion(title, content)`

## Flujo de trabajo habitual
1. Editar `index.html`
2. **Antes de cada commit/push: hacer backup de Supabase primero:**
   ```bash
   curl -s "https://xkywixwsovybcsisubrf.supabase.co/rest/v1/app_data?select=state&id=eq.1" \
     -H "apikey: <ANON_KEY>" \
     -H "Authorization: Bearer <ANON_KEY>" \
     -o "backup_datos_YYYYMMDD.json"
   ```
3. `git add index.html && git commit -m "..."`
4. `git push origin main`
5. GitHub Pages actualiza en ~1-2 minutos

## Convenciones del código
- Nombres de función cortos: `S()` guarda, `fmt()` formatea moneda, `uid()` genera ID
- `cur()` retorna la moneda activa del selector del header
- `to990(n)` redondea hacia arriba terminando en 990 (ej: 45.230 → 45.990)
- `saleOf(cost, margin)` calcula precio de venta: `cost / (1 - margin/100)`
- `totals(parts)` retorna `{tc, ts, profit, margin}` donde `tc`=costo total, `ts`=venta total
- Para guardar siempre llamar `S()` — nunca escribir directamente a `sbClient`

## Datos de la empresa (para documentos)
- **Desit Gamer Store** · RUT 77.839.066-3
- Banco Chile · Cta. Cte. 166 23169-10
- contacto.desit@gmail.com
