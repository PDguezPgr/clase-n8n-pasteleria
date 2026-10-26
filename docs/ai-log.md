# Registro de IA

## 2026-10-26 · Paso 8: flujo 04 «Aviso de entrega mañana»

**Herramienta:** OpenCode
**Entrada:** `AGENTS.md` + `docs/spec-04-aviso-entrega.md`
**Salida:** `mis-flujos/04-aviso-entrega.json`

### El prompt

> Lee `AGENTS.md` y esta spec. Genera el flujo como un JSON importable en `mis-flujos/04-aviso-entrega.json`.

### Qué hizo bien

- Respetó la regla de oro: los importes salen de la app (`pendiente`); n8n solo los muestra.
- No escribió ninguna clave: los nodos solo llevan el nombre y el id de las credenciales.
- Añadió el caso «sin entregas mañana» sin nodos extra: si el Code devuelve 0 items, los correos no se ejecutan.
- Usó `$now` (zona `Europe/Madrid`) para calcular «mañana», así que el flujo vale cualquier día.

### Qué hubo que corregir / decidir

- La spec decía «pide los encargos `pendiente` y `pagado`», pero la API **no** filtra por `fecha_entrega`. Se resolvió con **una** llamada a `GET /api/encargos` y el filtro de fecha en el nodo Code (menos nodos y menos peticiones).
- Hay encargos con `email` vacío (por ejemplo, los de prueba). Sin protección, `Send Email` fallaría con «No recipients defined». Se añadió el IF `¿Tiene email el cliente?` y el resumen de Marisa marca `SIN EMAIL`.

### Revisión (lista del Paso 8)

| Comprobación | Resultado |
|---|---|
| Los importes los toma de la app, no los calcula | Sí: usa `pendiente`; solo cambia el formato (12,00 €) |
| No hay claves escritas en los nodos | Sí: solo nombre e id de las credenciales |
| Funciona sin entregas mañana | Sí: el Code devuelve 0 items y no se ejecuta ningún correo |
| Los nombres de los nodos se entienden | Sí |
| Los textos de los correos son correctos | Revisados en seco; falta verlos en Mailpit |

### Pruebas

- [x] Sin entregas mañana → no llega ningún correo y la ejecución termina bien.
- [x] Con una entrega pagada y otra pendiente → 2 correos a clientes + 1 a Marisa.
- [x] Un encargo de mañana sin email → no se envía al cliente y Marisa lo ve marcado.
