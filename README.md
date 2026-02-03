# entrega-final-n8n
Support Ticket Severity via Telegram (n8n + 2 LLMs + Conditional + Notifications)

Workflow de n8n que recibe tickets de soporte por Telegram, clasifica severidad con un LLM, enruta por condicional y, para casos críticos, genera un resumen/impacto con un segundo LLM y notifica automáticamente por Telegram.

Cumple: trigger automático, 2 nodos LLM, condicional dependiente del LLM, notificación, manejo básico de errores y pruebas reproducibles.

Arquitectura

Telegram Trigger (nuevo mensaje)

LLM #1: Clasificación severidad (CRITICAL / NON_CRITICAL) en JSON estructurado

IF: rama CRITICAL vs NON_CRITICAL

(CRITICAL) LLM #2: resumen + impacto + preguntas de triage + acciones sugeridas (JSON)

Notificación: mensaje al chat de soporte (Telegram)

Acknowledgement: respuesta automática al usuario

Manejo de error: si el JSON del LLM1 no parsea, se notifica a soporte y se avisa al usuario

Requisitos

n8n (self-hosted o cloud)

Bot de Telegram (token)

OpenAI API key

Variables de entorno en n8n:

OPENAI_API_KEY

SUPPORT_CHAT_ID (ID numérico del chat/grupo donde llegan las alertas críticas)

Setup paso a paso
1) Crear el bot de Telegram

En Telegram, abrí @BotFather

/newbot y seguí los pasos

Guardá el token del bot (ej: 123456:ABC...)

2) Obtener el SUPPORT_CHAT_ID

Opción rápida:

Agregá el bot a un grupo de soporte (o usá un chat directo).

Enviá un mensaje en ese chat.

En n8n, ejecutá temporalmente un workflow con Telegram Trigger y mirá el message.chat.id en la ejecución para copiar el ID.

Ese número (puede ser negativo si es grupo) va en SUPPORT_CHAT_ID.

3) Configurar variables de entorno

En n8n (docker/.env o tu sistema), agregá:

OPENAI_API_KEY=...

SUPPORT_CHAT_ID=...

Reiniciá n8n si corresponde.

4) Importar el workflow

n8n → Workflows → Import from File / Paste JSON

Pegá el contenido de workflow.json

5) Configurar credenciales en n8n

Telegram:

Credentials → Telegram API

Pegá el bot token

Asigná esa credencial a:

Telegram Trigger

Todos los nodos Telegram sendMessage

OpenAI:

Este workflow usa HTTP Request con header Authorization: Bearer {{$env.OPENAI_API_KEY}}

No requiere credencial nativa de OpenAI en n8n, solo la variable de entorno.

6) Activar el workflow

Activá el workflow (Active = ON)

Uso

Enviá un mensaje al bot con el texto del ticket (texto libre).

El sistema clasifica severidad automáticamente.

Si es CRITICAL:

Notifica al chat/grupo SUPPORT_CHAT_ID

Responde al usuario confirmando escalación

Si es NON_CRITICAL:

Responde al usuario confirmando recepción

Pruebas (para la evidencia del curso)

Ejecutá al menos 3 entradas:

CRITICAL (caída general)

“Desde las 9 AM ningún usuario puede conectarse al WiFi de la oficina central. Operación detenida.”

NON_CRITICAL (consulta / menor)

“¿Cómo cambio la contraseña del WiFi de invitados?”

Caso borderline (para mostrar criterio)

“Un usuario no puede imprimir desde la impresora del piso 2.”

Guardá:

Capturas de la ejecución en n8n (logs)

Capturas del chat del bot (entrada)

Capturas del mensaje al grupo de soporte (si crítico)

Prompts (prompts.txt sugerido)
Prompt LLM1 (Clasificación)

El prompt está embebido en el nodo “LLM1 - Classify Severity”.

Salida estricta JSON:

{ "severity": "CRITICAL" | "NON_CRITICAL", "confidence": 0-1, "category": ... }

Prompt LLM2 (Generación)

Embebido en el nodo “LLM2 - Summary & Triage”.

Salida estricta JSON:

{ "summary": "...", "impact": "...", "triage_questions": [...], "first_actions": [...] }

Seguridad / buenas prácticas

No subas credenciales reales al repo.

El workflow usa variables de entorno (OPENAI_API_KEY, SUPPORT_CHAT_ID).

El bot token se configura como credencial interna de n8n (no se versiona).

Manejo de errores: si LLM1 devuelve algo no parseable, se notifica a soporte y se responde al usuario informando revisión manual.

Limitaciones y sesgos

La severidad depende del texto provisto por el usuario; mensajes incompletos pueden ser clasificados incorrectamente.

El LLM puede sesgar hacia CRITICAL en textos alarmistas o hacia NON_CRITICAL en textos vagos.

Recomendación: reforzar criterios en el prompt y/o añadir un paso adicional de verificación para producción.
