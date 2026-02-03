# entrega-final-n8n
Telegram Support Ticket Severity Automation (n8n + OpenAI)
📌 Descripción general

Este proyecto implementa un workflow automático en n8n para la gestión de tickets de soporte IT utilizando Modelos de Lenguaje (LLMs).

El sistema permite:

Recibir solicitudes de soporte vía Telegram

Validar automáticamente si el mensaje es un ticket real

Clasificar la severidad del ticket (CRITICAL / NON_CRITICAL) usando IA

Almacenar los tickets válidos en Google Sheets

Escalar automáticamente los tickets críticos mediante notificación

Responder siempre al usuario con un mensaje claro

El objetivo es demostrar orquestación de LLMs, lógica condicional basada en IA y automatización end-to-end con un caso de uso real.

🧠 Arquitectura del workflow
Flujo general
Telegram Trigger
→ Normalize Input
→ IF ticket_text not empty
→ LLM1 (Validación + Severidad)
→ Parse LLM1 Output
→ IF is_ticket == true
   → Google Sheets (guardar ticket)
   → IF severity == CRITICAL
      → LLM2 (resumen + impacto)
      → Parse LLM2 Output
      → Notificación a Soporte
      → Respuesta CRITICAL al usuario
   → ELSE
      → Respuesta NON-CRITICAL al usuario
→ ELSE
   → Respuesta "No es un ticket"

🔑 Componentes principales
1️⃣ Trigger: Telegram

Tipo: Telegram Trigger

Evento: mensaje entrante

Función: punto de entrada del sistema

📌 Requiere configuración manual:

Token del bot de Telegram (no incluido)

Chat habilitado para el bot

2️⃣ Normalize Input

Nodo de normalización para desacoplar el flujo del payload de Telegram.

Campos normalizados:

ticket_text

chat_id

from_user

timestamp

Esto permite:

Manejar texto, captions o mensajes vacíos

Evitar acoplamiento directo al JSON de Telegram

3️⃣ LLM1 — Validación y clasificación de severidad

Nodo: Basic LLM Chain + OpenAI Chat Model

Modelo recomendado: gpt-4o-mini

Rol: lógica y clasificación

Funciones:

Determina si el mensaje es un ticket válido (is_ticket)

Clasifica severidad (CRITICAL / NON_CRITICAL)

Salida estructurada (ejemplo):

{
  "is_ticket": true,
  "severity": "CRITICAL"
}


📌 El prompt define reglas explícitas de severidad para evitar ambigüedad.

4️⃣ Parse LLM1 Output

Nodo Function que:

Limpia posibles code fences

Parsea JSON devuelto por el LLM

Garantiza que is_ticket y severity existan siempre

Esto evita errores si el modelo devuelve texto inesperado.

5️⃣ IF is_ticket

Condicional basado en salida del LLM1.

false → mensaje de descarte al usuario

true → continuar flujo

❗ Los mensajes basura nunca se guardan en Sheets

6️⃣ Google Sheets — Persistencia de tickets

Nodo: Append Row

Función: base de datos simple de tickets

📌 Requiere configuración manual:

Google Account

Spreadsheet ID

Hoja con las siguientes columnas mínimas:

Columna	Descripción
timestamp	Fecha y hora
chat_id	ID del chat
from_user	Usuario de Telegram
ticket_text	Texto original
severity	CRITICAL / NON_CRITICAL
status	OPEN

Todos los tickets válidos (críticos o no) se almacenan.

7️⃣ IF severity == CRITICAL

Segundo condicional basado exclusivamente en salida del LLM1.

CRITICAL → escalar

NON_CRITICAL → respuesta simple al usuario

8️⃣ LLM2 — Resumen e impacto (solo CRITICAL)

Modelo: gpt-4o-mini

Rol: generación de contenido

Función:

Resumir el problema

Describir impacto operativo

Salida esperada:

{
  "summary": "Total outage of the billing system",
  "impact": "All invoicing operations are blocked"
}


📌 LLM2 no decide severidad, solo redacta.

9️⃣ Parse LLM2 Output

Nodo Function que:

Parsea el JSON de LLM2

Expone summary e impact para notificaciones

🔔 Notificación a Soporte

Canal: Email / Telegram / Slack (ejemplo con Gmail)

Contenido:

Texto original

Summary

Impact

Usuario y chat_id

📌 Requiere configuración manual:

Credenciales del canal elegido

Destinatarios

💬 Respuesta al usuario

Siempre hay respuesta:

Basura → “No se detectó un pedido de soporte”

NON_CRITICAL → “Ticket recibido”

CRITICAL → “Ticket crítico recibido, soporte notificado”

🧪 Pruebas recomendadas

Ejecutar al menos estos casos:

Mensaje	Resultado esperado
“hola”	descartado
“no anda mi wifi”	ticket NON_CRITICAL
“nadie tiene internet”	ticket CRITICAL + notificación

Capturar:

Ejecución en n8n

Filas en Google Sheets

Mensajes enviados

🔐 Seguridad y credenciales

❌ No se incluyen:

Tokens

API keys

IDs personales

📌 Debe configurarse manualmente:

Telegram Bot Token

OpenAI API Key

Google Sheets credentials

Canal de notificación

Usar credenciales de n8n, nunca hardcodear.

⚠️ Limitaciones conocidas

Clasificación depende del prompt (posible sesgo semántico)

No hay memoria conversacional

No se manejan adjuntos (solo texto)

Google Sheets no es un sistema de tickets completo

📦 Archivos incluidos

workflow.json → importar en n8n

README.md → este documento

prompts.txt → prompts usados en LLM1 y LLM2

Evidencias → capturas o video de ejecución
