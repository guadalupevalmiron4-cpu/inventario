# Inventario AIPSA

Prototipo (borrador) del sistema de inventario de AIPSA. Todo está en un solo archivo: `aipsa-inventario-poc_BORRADOR.html` (≈4.950 líneas, HTML + CSS + JS, sin dependencias ni build). Se prueba abriéndolo en el navegador.

- Los datos son de ejemplo y viven en memoria: no hay backend ni persistencia; todo se pierde al recargar.
- La usuaria escribe en español (rioplatense). Responder y escribir la interfaz y la documentación en español.
- Resumen técnico completo en Claude Docs: https://claude.ai/artifact/X7A5aPmF2qDBy7wgFTsExc

## Estructura del archivo (de arriba a abajo)

Estilos → HTML → datos de ejemplo (líneas ~255–4100) → `state` (~4105) → funciones auxiliares → pantallas → `wire()` → modales → arranque.

- **Render sin framework:** cada pantalla es una función que devuelve un string HTML. `render()` elige la pantalla según `state.view`, la vuelca en `#content` y llama a `wire()` para volver a enganchar los handlers. Cada acción re-renderiza todo.
- **Estado:** un único objeto `state` (vista, perfil `viewAs`, rango `desde`/`hasta`, buscadores, mapas de acordeones abiertos `openFam`, `openTpl`, `openRow`, `openDesc`…).
- **Perfiles:** selector "Ver como" (`admin`, `nomontos`, `reader`). `canWrite()` y `canMontos()` deciden qué se muestra. Es demostración, no hay autenticación.
- **Tema:** claro/oscuro con variables CSS en tres bloques que hay que mantener sincronizados (`:root`, `@media (prefers-color-scheme: dark)`, `:root[data-theme="dark"]`).
- **Búsqueda:** `normTxt`, `searchTokens`, `hayIncludes` (ignoran tildes y espacios).

## Modelo de datos

El código usa Plantilla/Variante; la interfaz muestra **Familia → Tipo → Ítem** (`fam` de la plantilla = Familia, plantilla = Tipo, variante = Ítem).

- `TEMPLATES` (Tipos; `kind`: RAW, MATERIAL, KIT, SUPPLY) y `VARIANTS` (Ítems: stock por sección, receta, costo, `tec`, `campos`, `obs`, `adj`).
- `DEPOSITOS`, `SECCIONES`, `DEPARTAMENTOS`: ubicaciones y responsables.
- `EVENTOS` (presupuestos vivos con reservas), `LIBRO` (movimientos), `SOLICITUDES` (compras).
- `KIT_TPLS`, `KIT_ROWS`, `DEMO_KITS`: catálogo de kits (cargado de `SIST_STOCK.xlsx`).
- Orden de familias: `MP_TIPOS`/`FAMS_MP` (Materia prima) y `FAM_ORDER` (Materiales).

## Pantallas (`state.view`)

- `mp` / `mat`: `pageInventario` → `famCard` → `tplBlock` → `varRow` → `varDetail` (ficha técnica, secciones A–G).
- `kits`: `pageKits`, `kitDetail`. `deps`: `pageDepositos`. `evs`: `pageEventos`. `compras`: `pageCompras`, `panelCompras`, `fichaSolicitud`.
- El inventario es de solo lectura: varios modales (`modalTransferir`, `modalProducir`, `modalEditarVariante`, `drawerPlantilla`…) siguen en el código sin botón que los llame.

## Reglas de negocio

- Físico = suma de `v.stock`. Reservado = reservas de eventos en la ventana de fechas que siguen en estado "Reservado". Disponible = físico − reservado.
- Alertas: `alertaGlobal` (físico ≤ `minG`) y `alertasSeccion` (mínimo por almacén).
- Costo: `v.costo` si se compra; si tiene receta, suma de materias primas; kits = A (ítems) + B (mano de obra) + C (logística).
- Kits sin stock propio: `kitCapacidad` calcula cuántos se pueden armar.
- Responsable único (`respItem`): el de la sección con más stock del ítem, o "Sin responsable".

## Pendientes conocidos

- Persistencia real (base de datos + API), adjuntos a un almacenamiento real, autenticación y roles.
- Usar el código interno de la planilla (`Cod Int`) en vez de los generados (`genCode`).
- Unificar nombres de familias (Materiales: Vidrios/Hierros/Textiles; Materia prima: Vidrio/Hierro/Textil).
- No hay pruebas automáticas.
