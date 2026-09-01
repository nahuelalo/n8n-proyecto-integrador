# n8n-proyecto-integrador

Proyecto integrador desarrollado en n8n para Coderhouse.

El proyecto evoluciona de forma incremental mediante una arquitectura
multi-agente orientada a soporte IT, incorporando clasificación de
intenciones, sub-workflows especializados, memoria persistente e
integraciones externas.

## Checkpoint 4 - Integraciones Avanzadas

El workflow principal de esta entrega es:

`checkpoint4_nahuel_fernandez.json`

La solución extiende la arquitectura desarrollada en los módulos anteriores
e incorpora integraciones reales con sistemas externos.

### Arquitectura

El Manager central recibe solicitudes, aplica controles preventivos,
clasifica la intención y delega operaciones a componentes especializados.

Integraciones utilizadas:

- Gmail
  - Recepción de correos mediante Gmail Trigger.
  - Lectura controlada de mensajes.
  - Creación exclusiva de borradores mediante Create Draft.
  - No se realiza envío automático de correos.

- HubSpot CRM
  - Búsqueda previa de contactos.
  - Actualización de contactos existentes.
  - Creación solamente cuando el contacto no existe.
  - Prevención de duplicados mediante patrón Lookup Before Create.

- Telegram
  - Canal de observabilidad y notificaciones operativas.
  - El payload es reducido previamente mediante nodos Edit Fields / Set.

- Airtable
  - Memoria persistente correlacionada mediante Session ID.
  - Recuperación de contexto previo.
  - Persistencia de resúmenes consolidados.
  - Contador de mensajes y summarization periódica.

- Groq
  - Modelo LLM utilizado por el Manager y el proceso de summarization.
  - Temperature = 0 para reducir variabilidad.

## Controles de seguridad implementados

### 1. Prevención de auto-respuestas

Después del Gmail Trigger se aplica una validación determinista que detecta
correos automáticos, incluyendo patrones como:

- Auto-reply
- Out of office
- Undeliverable
- no-reply@
- noreply@
- mailer-daemon

Los mensajes detectados se descartan antes de invocar al agente,
evitando loops infinitos de respuestas automáticas.

### 2. Prevención de contactos duplicados

Antes de crear un contacto en HubSpot se realiza una búsqueda previa.

Flujo:

`Search Contact → Existe Contacto? → Update / Create`

De esta forma se evita la creación innecesaria de contactos duplicados
y posibles errores HTTP 409.

### 3. Human-in-the-loop

La respuesta generada para Gmail se almacena únicamente mediante:

`Create Draft`

El workflow no envía automáticamente el correo.

Un usuario debe revisar y aprobar manualmente el borrador antes de su envío.

### 4. Limpieza de payload

Antes de enviar información hacia conectores externos se utilizan nodos
Edit Fields / Set para transmitir únicamente los campos necesarios.

Esto evita:

- payloads innecesariamente grandes;
- datos binarios;
- metadata técnica;
- información no requerida por el sistema destino.

## Arquitectura multi-agente

El Manager utiliza dos sub-workflows especializados:

### Worker 1 - Registrar Incidente

Responsable de:

- validar datos obligatorios;
- registrar incidentes;
- devolver una respuesta estructurada al Manager.

### Worker 2 - Consultar Incidente

Responsable de:

- validar el identificador;
- consultar incidentes existentes;
- manejar casos no encontrados;
- devolver una respuesta estructurada.

Los Workers utilizan `Execute Workflow Trigger` y son invocados desde el
Manager mediante `Execute Workflow`.

## Memoria persistente

La solución mantiene memoria de largo plazo utilizando Airtable.

Campos principales:

- Session_ID
- User_Name
- Fecha_Actualizacion
- Resumen_Consolidado
- Estado_Caso
- Datos_Clave
- Cantidad_Mensajes

El contexto se recupera utilizando el Session ID para evitar contaminación
entre sesiones.

Cuando se supera el umbral configurado de mensajes, un modelo LLM genera
un resumen estructurado de la conversación y actualiza el registro existente.

## Archivos

- `checkpoint4_nahuel_fernandez.json`
  - Workflow Manager principal correspondiente al Checkpoint 4.

- `worker1_registrar_incidente.json`
  - Sub-workflow para registro de incidentes.

- `worker2_consultar_incidente.json`
  - Sub-workflow para consulta de incidentes.

## Requisitos antes de importar

Configurar las credenciales correspondientes en n8n:

- Groq
- Gmail OAuth2
- HubSpot
- Telegram
- Airtable

También es necesario reemplazar cualquier valor placeholder, por ejemplo:

`YOUR_TELEGRAM_CHAT_ID`

por el identificador correspondiente al entorno donde se ejecute.

Los IDs, tokens, secretos OAuth2, API keys y demás credenciales no se
incluyen en este repositorio.

## Importación

1. Importar los dos Workers en n8n.
2. Importar `checkpoint4_nahuel_fernandez.json`.
3. Configurar las credenciales.
4. En los nodos Execute Workflow del Manager, seleccionar nuevamente los
   Workers importados si sus IDs fueron modificados por n8n.
5. Ejecutar pruebas manuales antes de activar el workflow.

## Seguridad

El repositorio no contiene:

- Access Tokens
- Refresh Tokens
- Client Secrets
- API Keys
- contraseñas
- credenciales personales

Las integraciones deben configurarse utilizando credenciales almacenadas
en el gestor de credenciales de n8n.
