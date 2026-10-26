# Registro de IA

## 2026-10-26 · Paso 8: flujo 04 «Aviso de entrega mañana»

**Herramienta:** OpenCode
**Entrada:** `AGENTS.md` + `docs/spec-04-aviso-entrega.md`
**Salida:** `mis-flujos/04 Aviso de entrega.json`

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
| Los textos de los correos son correctos | Sí: comprobados en Mailpit el 26/10 (ejecución de las 20:48) |

### Pruebas

- [x] Sin entregas mañana → no llega ningún correo y la ejecución termina bien.
- [x] Con una entrega pagada y otra pendiente → 2 correos a clientes + 1 a Marisa.
- [x] Un encargo de mañana sin email → no se envía al cliente y Marisa lo ve marcado.

### Resultado de las pruebas (26/10/2026)

Todo se comprobó sobre Mailpit (`http://localhost:8025`) y las ejecuciones reales de n8n.

**1. Sin entregas mañana** — ejecución **229** (20:29, manual, `success`):

```text
Encargos de la app: 20 item(s)
Buscar entregas de mañana: 0 item(s)
Último nodo ejecutado: Buscar entregas de mañana
```

No se ejecutó ningún nodo de correo, así que no llegó nada a Mailpit. El flujo termina bien porque el Code devuelve 0 items; no hace falta un IF ni marcar «Always Output Data».

**2. Una entrega pagada y otra pendiente** — ejecución **235** (20:48, manual, `success`) con los encargos #21 y #22 del 27/10. Llegaron 3 correos:

| Destinatario | Asunto |
|---|---|
| `trialv7@trial.com` | Mañana puedes recoger tu Tarta de Manzana |
| `trialv8@trial.com` | Mañana es la entrega de tu Tarta sacher |
| `marisa@lacuchara.example` | Mañana hay 2 entrega(s) de tartas |

El resumen de Marisa mostró los importes tal cual vienen de la app:

```text
Mañana, 27/10/2026, hay 2 entrega(s):

- #21 Trial v7: Tarta de Manzana (pagada)
- #22 Trial v8: Tarta sacher (faltan 40,00 €)

Todos los clientes con email ya tienen su aviso.
```

**3. Un encargo de mañana sin email** — ejecución **238** (21:28, manual, `success`) con el encargo de prueba **#25 «Prueba Sin Email»** (entrega 27/10, pendiente 20,00 €, sin email). Los items que pasan por cada nodo:

```text
Encargos de la app: 25 item(s)
Buscar entregas de mañana: 3 item(s)
¿Tiene email el cliente?: 2 item(s)
Avisar al cliente: 2 item(s)
Sin email: lo ve Marisa: 1 item(s)
Resumen para Marisa: 1 item(s)
```

Salieron **2 correos a clientes** (#21 y #22) y **ninguno a #25**, porque no hay dirección a la que escribir. El resumen de Marisa añadió la línea marcada y el aviso:

```text
- #25 Prueba Sin Email · SIN EMAIL: Tarta de prueba (faltan 20,00 €)

Ojo: hay clientes sin email; avísales por teléfono.
```

### Incidencia detectada al probar el caso sin email

Al crear el encargo #25, el flujo **01** intentó mandarle la confirmación y falló con `No recipients defined` (ejecución 236, 21:19). El flujo **00** avisó a Marisa del error, así que la red de seguridad funciona, pero la confirmación no se envió. Queda pendiente proteger el flujo 01 con la misma comprobación de email que ya tiene el 04 (IF «¿Tiene email el cliente?» antes de los envíos).
