# Sesión 01 — Resumen de Trabajo
**Fecha:** 2026-04-29

## Lo que se hizo

### 1. Repo clonado (NO tocar)
- **Local:** `C:\Users\dayan\Desktop\TRABAJOS DE CODE\ASISTENTE DAYANA\whatsapp-ia-agente-seguros`
- **GitHub:** `https://github.com/marketing-ia/whatsapp-ia-agente-seguros`
- Contiene: `workflows/`, `.env.example`, `.gitignore`, `README.md`

### 2. Nuevo repo creado: whatsapp-dayana
- **Local:** `C:\Users\dayan\Desktop\TRABAJOS DE CODE\ASISTENTE DAYANA\whatsapp-dayana`
- **GitHub:** `https://github.com/marketing-ia/whatsapp-dayana`
- Contiene carpeta `.md/` (aquí van los documentos del proyecto)

### 3. API Keys
- Archivo de referencia: `whatsapp-ia-agente-seguros/api-keys/.env.example`
- APIs necesarias:
  - `EVOLUTION_API_KEY` — WhatsApp (Evolution API)
  - `OPENAI_API_KEY` — IA / GPT
  - `GHL_API_KEY` + `GHL_LOCATION_ID` + `GHL_CALENDAR_ID` — GoHighLevel CRM
  - `CHATWOOT_API_TOKEN` — Escalamiento humano
  - `N8N_API_KEY` + `N8N_API_URL` — n8n automatizaciones

### 4. n8n-MCP (PENDIENTE completar)
- Archivo `.mcp.json` creado en `whatsapp-ia-agente-seguros/`
- **Falta:** reemplazar `REEMPLAZAR_URL` y `REEMPLAZAR_API_KEY` con valores reales
- Comando alternativo para registrar globalmente (correr en terminal):
  ```
  claude mcp add n8n-mcp -e N8N_API_URL=TU_URL -e N8N_API_KEY=TU_KEY -- npx -y n8n-mcp
  ```

### 5. Claude Code — Permisos
- `bypassPermissions` activado en `~/.claude/settings.json`
- **Requiere reiniciar Claude Code** para que tome efecto
- Después del reinicio ya NO pide autorizaciones en cada paso

## Próxima sesión
1. Dar `N8N_API_URL` y `N8N_API_KEY` para terminar configuración n8n-mcp
2. Verificar que `bypassPermissions` funciona tras reinicio
