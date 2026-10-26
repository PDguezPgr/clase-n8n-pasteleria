# Bitácora de Registro de IA e Incidencias — Pastelería La Cuchara

En este documento se registran los problemas técnicos, soluciones aplicadas, elecciones de modelos de Inteligencia Artificial e iteraciones del prompt realizadas durante el desarrollo de la práctica con **n8n**, **Docker**, **Groq API** y **FastAPI**.

## 1. Configuración de Entorno y Flujos de Control

### Incidencia 01: Botón `Publish` deshabilitado en Flujo 00

* **Síntoma:** El botón **Publish** o el conmutador de activación en n8n aparecía deshabilitado / en gris.

* **Causa raíz:** El nodo `Error Trigger` (`Si falla un flujo`) se encontraba en estado desactivado (`Disabled`), lo cual impedía a n8n registrar un disparador válido para el flujo.

* **Solución:** Se reactivó el nodo `Error Trigger` internamente y se guardó el flujo mediante `Ctrl + S`. Con esto el botón **Publish** se activó correctamente.

* **Aprendizaje:** Un flujo con disparador desactivado no puede ser publicado. Los flujos de control como `00 Avisar de errores` deben estar publicados para poder ser seleccionados en la pestaña *Settings -> Error Workflow* de otros flujos (`01`, `02`, `03`).

## 2. Integración con IA (Groq API + Information Extractor)

### Incidencia 02: Error 400 Bad Request — Modelo Deprecado

* **Síntoma:** `Problem in node 'Information Extractor' - Bad request - please check your parameters`.

* **Causa raíz:** Selección de modelos de IA retirados/descontinuados por el proveedor Groq (como `llama3-8b-8192` o `llama2-70b-4096`).

* **Solución:** Se actualizó el nodo **Groq Chat Model** para utilizar un modelo activo de instrucción y chat, concretamente **`llama-3.3-70b-versatile`** (o `llama-3.1-8b-instant`).

* **Regla:** Evitar modelos con etiquetas `guard`, `whisper` o `text-classification`, ya que imponen restricciones de API incompatibles con la extracción estructurada.

### Incidencia 03: Error 400 — Restricción de Modelos de Clasificación

* **Mensaje de Error:** `failed to template request: failed to render text output: messages must contains a single user message for text classification models`.

* **Causa raíz:** Selección accidental de un modelo restringido a tareas específicas de clasificación de texto.

* **Solución:** Cambiar el conector inferior del model a **`llama-3.3-70b-versatile`**, el cual acepta mensajes de sistema (*System Prompts*) y salida JSON flexible.

### Incidencia 04: Expresiones de Entrada Inexistentes (`Text for item 0 is not defined`)

* **Síntoma:** El nodo *Information Extractor* indicaba que la propiedad de texto no estaba definida.

* **Causa raíz:** Se configuró la variable `{{ $json.Correo }}` o `{{ $json.texto }}`, pero el nodo anterior **Pegar correo** (*n8n Form Trigger*) tenía definido el campo como **`Texto del correo`**.

* **Solución:** Corregir la expresión dinámica a:

  ```
  {{ $json['Texto del correo'] }}
  
  ```

### Incidencia 05: Incompatibilidad de Esquema en Casos Borde (Fallo en Correos 2, 4, 6 y 7)

* **Síntoma:** Los datos llegaban bien al disparador, pero el *Information Extractor* devolvía `Connection error` / `Bad Request` en casos específicos:

  * Caso 2: Ordenante con inicial/punto (`JORGE M.`) y concepto con número (`encargo 2`).

  * Caso 4: Concepto con nombre de tarta (`red velvet`).

  * Caso 6: Persona desconocida con dos apellidos (`ANTONIO FERNANDEZ SOTO`).

  * Caso 7: Publicidad bancaria sin datos de pago ni importes.

* **Causa raíz:** Cuando la IA hallaba correos sin pago o con números en el texto, intentaba devolver tipos de datos mixtos (integers en vez de strings, o valores `null`), rompiendo la validación del parser JSON de n8n.

* **Solución (Iteración del System Prompt):**
  Se estableció una regla estricta en el **System Prompt Template** para garantizar la homogenización de tipos:

  ```
  Eres un asistente que analiza notificaciones bancarias. 
  Instrucciones strictly:
  - es_pago: boolean (true para Bizum o transferencia de pago, false si es publicidad u otro tipo de correo).
  - ordenante: string (nombre del pagador tal cual aparece). Si no hay, usa "".
  - importe: number (valor numérico con decimales, por ejemplo 15.0). Si no hay pago, usa 0.
  - forma: string ("bizum" o "transferencia" en minúsculas). Si no se indica, usa "".
  - concepto: string (extrae el texto del concepto tal cual, sin convertir números a integer). Si no hay, usa "".
  
  Regla de oro: Devuelve SIEMPRE texto string para ordenante, forma y concepto. NUNCA devuelvas valores null ni dejes campos omitidos.
  
  ```

## 3. Resumen del Estado de la Práctica

| Componente / Flujo | Estado | Notas | 
| ----- | ----- | ----- | 
| **00 Avisar de errores** | ✅ Publicado | Captura fallos con `Error Trigger` y envía mail vía Mailpit. | 
| **01 Eventos de la pastelería** | ✅ Publicado | Recibe Webhooks y envía confirmaciones/recibos. Conectado a Flujo 00. | 
| **02 Cierre diario** | ✅ Publicado | Se ejecuta por `Schedule Trigger` a las 21:00. Conectado a Flujo 00. | 
| **03 Procesar aviso de pago** | ⏳ En desarrollo | Triggers y nodo de IA (Information Extractor + Groq) validados en los 7 casos. | 
