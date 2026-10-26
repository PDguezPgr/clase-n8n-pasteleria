# Spec: aviso de entrega mañana (flujo 04)

- **Disparador:** cada día a las 9:00.
- **Datos:** pide los encargos a la API (`GET /api/encargos`) y se queda con los `pendiente` y `pagado` cuya `fecha_entrega` sea mañana.
- **Cliente con el encargo `pagado`:** correo «Mañana puedes recoger tu tarta».
- **Cliente con el encargo `pendiente`:** correo «Mañana es la entrega y aún te faltan X €».
- **Marisa:** si hay entregas, **un solo** correo con la lista. Si no hay ninguna, no se envía nada.
- **Criterios de aceptación:**
  - Los importes salen de la app (`pendiente`), no se calculan en n8n.
  - No hay claves ni credenciales escritas en el JSON.
  - Sin entregas mañana: el flujo termina sin enviar correos y sin fallar.
  - Con una entrega pagada y otra pendiente: 2 correos a clientes + 1 a Marisa, con los importes de la app.
  - Los nombres de los nodos y los textos de los correos se entienden.

## Nota técnica

La API solo filtra por `estado` y `q`, no por fecha de entrega. Para no hacer dos peticiones (pendientes y pagados) se hace **una** y se filtra la fecha en el nodo Code `Buscar entregas de mañana`.
