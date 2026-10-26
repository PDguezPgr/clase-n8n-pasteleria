# Pastelería La Cuchara: automatizaciones con n8n

Contexto para agentes de IA que trabajen en este repositorio.

- La app (http://pasteleria:8000 dentro de Docker) guarda encargos y pagos. Se usa por API con la cabecera `X-API-Key` (credencial `API Pasteleria` en n8n).
- n8n corre en Docker. Los correos salen por la credencial SMTP `Mailpit`.
- Los flujos se guardan como JSON en `mis-flujos/` con el formato `NN-nombre.json`.
- Los importes siempre los calcula la app: n8n solo los muestra.
- NO escribir credenciales ni claves dentro de los JSON. NO publicar flujos sin que yo los haya probado.

## La API en lo que afecta a los flujos

`GET /api/encargos` devuelve una lista de encargos con estos campos:

`id`, `fecha`, `cliente`, `email`, `tarta`, `raciones`, `precio`, `fecha_entrega` (AAAA-MM-DD), `estado` (`pendiente` o `pagado`), `pagado`, `pendiente`.

- Filtros disponibles: `estado` y `q` (texto). **No** hay filtro por `fecha_entrega`: para «entregas de mañana» hay que traer los encargos y filtrar la fecha en un nodo Code.
- `pendiente` ya viene calculado por la app (`precio - pagado`). No recalcularlo en n8n.
- Registrar un pago: `POST /api/encargos/{id}/pagos` con `{ "importe": ..., "forma_pago": ... }`.
- La app avisa a n8n por webhook (`encargo.creado`, `pago.registrado`); el flujo 01 manda los correos al cliente.

## Convenciones de los flujos

- Zona horaria `Europe/Madrid`; `$now` ya está en esa zona.
- Nombres de nodo en español y en imperativo («Buscar…», «Avisar…»).
- Un flujo nuevo se entrega como JSON importable y **sin publicar** (`active: false`).
- Destinataria habitual de los avisos internos: `marisa@lacuchara.example`.
