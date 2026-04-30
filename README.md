# Andrea — Agente WhatsApp IA para Dayana Muñoz

Asistente conversacional de ventas construido sobre **n8n + Evolution API + OpenAI + GoHighLevel + Chatwoot**.

Atiende leads de **dos negocios simultáneamente** desde un solo WhatsApp de forma autónoma: detecta el negocio de interés, califica, agenda citas, envía documentos informativos, escala a Dayana cuando es necesario y respeta el STOP del cliente.

---

## Stack

| Servicio | Uso |
|---|---|
| **n8n** (self-hosted) | Orquestación de flujos |
| **Evolution API** | Canal WhatsApp — instancia `Dayana Munoz` |
| **OpenAI gpt-4o-mini** | Conversación + Whisper (audio) + Vision (imágenes) |
| **GoHighLevel** | CRM, contactos, calendario, tags |
| **Chatwoot** | Bandeja unificada, escalamiento humano |
| **Twilio** | Llamadas telefónicas — conecta lead con Dayana |

---

## Negocios

| Negocio | Modo | Descripción |
|---|---|---|
| **Dayana Finance** | Modo A | Seguros de vida, salud, complementarios — familias hispanas en EE.UU. |
| **DOMD PRO** | Modo B | Marketing digital — redes sociales, páginas web, anuncios, contenido |

La IA detecta automáticamente el negocio por el contexto del mensaje. Si no es claro, pregunta al inicio.

---

## Capacidades

| Función | Detalle |
|---|---|
| **Conversación IA** | gpt-4o-mini, en español, doble personalidad (Finance / Marketing) |
| **Memoria** | Historial de conversación por contacto (n8n Static Data) |
| **Disponibilidad en tiempo real** | Consulta slots libres en GHL Calendar (próximos 7 días) |
| **Calificación de leads** | Pregunta necesidad, tipo de producto, situación actual |
| **Citas en CRM** | Detección en 2 pasos: `[QUIERE_AGENDAR]` → `[RESERVAR:fecha=X\|hora=Y]` → crea cita en GHL |
| **Reagendar / Cancelar citas** | `[REAGENDAR:]` + `[CANCELAR_CITA]` — gestión completa de citas |
| **Link de booking** | Envía link del calendario si no hay fecha confirmada |
| **Documentos informativos** | 9 tipos por token `[ENVIAR_DOC:tipo]` — Finance y DOMD PRO |
| **Solicitud de llamada** | `[SOLICITA_LLAMADA]` → Twilio llama a Dayana + notifica por WhatsApp |
| **Escalamiento humano** | `[ESCALAR_HUMANO]` → asigna conversación en Chatwoot + pausa bot |
| **Audio** | Whisper transcribe notas de voz |
| **Imágenes** | GPT-4o-mini Vision analiza imágenes |
| **STOP / CONTINUE** | Frases detectadas; escribe tag `lucy-paused` en GHL (persiste entre reinicios) |
| **Pausa por intervención** | Dayana responde en WhatsApp/Chatwoot → bot se pausa automáticamente |
| **Follow-up** | Cada 2h: 24h sin respuesta → recordatorio; 48h → último mensaje |
| **GHL → WhatsApp** | Mensajes de agentes en GHL se reenvían automáticamente a WhatsApp |
| **Sync en GHL** | Mensajes entrantes y respuestas IA se loguean en conversación GHL |
| **Anti-grupos** | Ignora mensajes de grupos |
| **Dedup** | Triple filtro: fromMe + messageId + rate limit (10/min por teléfono) |

---

## Workflows

| Archivo | ID n8n | Función | Nodos |
|---|---|---|---|
| `workflows/whatsapp-main.json` | `dYnMainWA2026xxAB` | Flujo principal | 61 |
| `workflows/whatsapp-followup.json` | `dYnFollowup2026xCD` | Follow-up 24h/48h | 6 |
| `workflows/ghl-to-whatsapp.json` | `ghl-to-whatsapp-dayana` | GHL → WhatsApp bridge | 8 |

---

## Flujo principal (simplificado)

```
WhatsApp Webhook
    ↓
¿Es mensaje válido? (filtra grupos/reacciones)
    ↓
¿Comando de Roy/Dayana? (fromMe=true + comando STOP/CONTINUE)
    ↓
Dedup mensajes
    ↓
Extraer datos (phone, message, pushName, media)
    ↓
Filtro STOP y pausa (detecta STOP inline, escribe tag GHL)
    ↓
¿Es media? → audio/imagen/video → respuesta contextual
    ↓ es texto
¿Es comando STOP/CONTINUE? → Ejecutar + Confirmar
    ↓ no es comando
¿Contacto pausado? (staticData + fallback GHL tag)
    ↓ no pausado
Cargar historial → Disponibilidad GHL → Preparar prompt
    ↓
OpenAI Chat → Extraer respuesta AI
    ↓
Guardar historial
    ↓
    ┌──────────┬──────────┬──────────┬──────────┬──────────┐
[LLAMADA]  [HUMANO]  [AGENDAR] [RESERVAR] [DOC]   [Sync/Log]
   ↓           ↓         ↓         ↓        ↓        ↓
 Twilio   Chatwoot  Link booking  Cita GHL  PDF   GHL Conv
 + notif  + pausa             ↕ reagendar/cancelar
```

---

## Flujo GHL → WhatsApp

```
GHL Webhook (respuesta de agente en CRM)
    ↓
¿Es respuesta de agente? (no es mensaje automático)
    ↓
Obtener contacto GHL → ¿Tiene teléfono?
    ↓
Enviar a WhatsApp (Evolution API)
    ↓
Pausar bot para ese contacto
```

---

## Tokens de control (AI → n8n)

| Token | Acción |
|---|---|
| `[SOLICITA_LLAMADA]` | Twilio llama a Dayana, notifica por WhatsApp |
| `[MOTIVO:texto]` | Motivo de la llamada (acompaña a SOLICITA_LLAMADA) |
| `[ESCALAR_HUMANO]` | Asigna en Chatwoot, pausa bot |
| `[QUIERE_AGENDAR]` | Paso 1 citas: AI pregunta disponibilidad |
| `[RESERVAR:fecha=YYYY-MM-DD\|hora=HH:MM]` | Paso 2 citas: crea cita en GHL |
| `[REAGENDAR:fecha=YYYY-MM-DD\|hora=HH:MM]` | Reagenda cita existente en GHL |
| `[CANCELAR_CITA]` | Cancela cita existente en GHL |
| `[ENVIAR_DOC:tipo]` | Envía documento — tipos: `seguro_vida`, `cancer`, `dental`, `hospital`, `accidente`, `catalogo_domd`, `portafolio_domd`, `precios_domd`, `finanzas` |

---

## Tags GHL

| Tag | Significado |
|---|---|
| `whatsapp-lead` | Lead identificado por WhatsApp |
| `lucy-paused` | STOP enviado — bot no responde (persiste en GHL) |
| `followup-24h` | Seguimiento de 24h enviado |
| `followup-48h` | Seguimiento de 48h enviado |

---

## IDs clave

| Servicio | ID / Valor |
|---|---|
| n8n host | `n8n-n8n.78s07r.easypanel.host` |
| Evolution API host | `n8n-evolution-api.78s07r.easypanel.host` |
| Instancia WhatsApp | `Dayana Munoz` |
| GHL Location ID | `IYslAlpLthyYmzorDhnP` |
| GHL Calendar ID | `xyMoZ5iLareyhwQNijO5` |
| Chatwoot account_id | `1` |
| Chatwoot inbox_id | `6` |
| Número Dayana | `+16155467023` |
| Webhook path principal | `dayana-whatsapp-main` |

---

## Control manual (Dayana desde WhatsApp)

Dayana puede controlar el bot enviando mensajes **desde su propio número**:

### Pausar un contacto
```
STOP 14077912171
```

### Reanudar un contacto
```
CONTINUE 14077912171
```

### Responder directamente a un cliente
Simplemente responde en la conversación del cliente en WhatsApp o Chatwoot → el bot se pausa automáticamente para ese contacto.

---

## Documentos (PENDIENTE configurar URLs)

Los siguientes documentos están definidos en el nodo `Enviar Documento` pero requieren que se suban los PDFs y se actualicen las URLs:

| Key | Nombre archivo | Estado |
|---|---|---|
| `seguro_vida` | Seguro-de-Vida-DayanaFinance.pdf | ⏳ PENDIENTE URL |
| `cancer` | Poliza-Cancer-DayanaFinance.pdf | ⏳ PENDIENTE URL |
| `dental` | Plan-Dental-DayanaFinance.pdf | ⏳ PENDIENTE URL |
| `hospital` | Hospital-Indemnity-DayanaFinance.pdf | ⏳ PENDIENTE URL |
| `accidente` | Poliza-Accidente-DayanaFinance.pdf | ⏳ PENDIENTE URL |
| `catalogo_domd` | Catalogo-Servicios-DOMDPRO.pdf | ⏳ PENDIENTE URL |
| `portafolio_domd` | Portafolio-DOMDPRO.pdf | ⏳ PENDIENTE URL |
| `precios_domd` | Precios-DOMDPRO.pdf | ⏳ PENDIENTE URL |
| `finanzas` | Plan-Financiero-DayanaFinance.pdf | ⏳ PENDIENTE URL |

---

## Estado del sistema

| Subsistema | Estado |
|---|---|
| n8n — 3 workflows | ✅ importados |
| Evolution API `Dayana Munoz` | ⏳ pendiente escanear QR |
| Chatwoot inbox 6 | ⏳ pendiente conectar instancia |
| GHL Location + Calendar | ✅ configurados |
| Documentos PDF | ⏳ pendiente subir y configurar URLs |

---

## Diferencias vs Lucy (Roma Insurance)

Dayana tiene capacidades **adicionales** que Lucy no tiene:

| Función extra | Descripción |
|---|---|
| **Doble negocio** | Detecta Finance vs Marketing automáticamente |
| **Disponibilidad real-time** | Consulta slots libres de GHL en el prompt |
| **Reagendar / Cancelar** | Gestión completa de citas existentes |
| **GHL → WhatsApp bridge** | Respuestas de CRM llegan al cliente por WhatsApp |
| **Sync conversación GHL** | Mensajes y respuestas IA se loguean en GHL |

---

## Licencia

Uso privado — Dayana Muñoz / Marketing IA
