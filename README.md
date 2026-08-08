# n8n-proyecto-integrador
Proyecto de n8n para coderhouse

el agente usa un modelo Groq, tiene Max Iterations = 6, Temperature = 0, una Data Table como Tool autónoma y Telegram como observabilidad determinística.

Antes de importar y ejecutar el workflow:

- Configurar una credencial de Groq.
- Configurar una credencial de Telegram.
- Reemplazar `YOUR_TELEGRAM_CHAT_ID` por el Chat ID correspondiente.
- Crear una Data Table llamada `incidentes_soporte` con las columnas:
  - fecha
  - categoria
  - descripcion
  - servicio_afectado
  - prioridad

Las credenciales y datos personales no se incluyen en el repositorio.
