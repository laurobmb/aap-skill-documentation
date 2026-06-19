# Ch01 — Ansible MCP Server Standalone (Novo no 2.7)

## O que é o Ansible MCP Server

Componente **standalone e deployável** que expõe o AAP como servidor MCP para **AI agents externos** (Cursor, Claude, ChatGPT, VS Code Copilot, etc.).

Diferente do MCP no Intelligent Assistant (que é interno ao chatbot), o Ansible MCP Server é acessível por qualquer AI agent externo via HTTP.

## Workflow

```
AI Agent (Cursor/Claude) → prompt natural language
        ↓
   AI Model (LLM) traduz para tool call
        ↓
   Ansible MCP Server (porta 8448) ← autentica com token do usuário
        ↓
   AAP Controller API → executa automação
        ↓
   Resultado volta ao AI Agent
```

## Toolsets Disponíveis

| Toolset | URL path | O que faz |
|---|---|---|
| `job_management` | `/job_management/mcp` | Listar JTs, disparar jobs, ver output, relaunch |
| `inventory_management` | `/inventory_management/mcp` | Browse inventories, hosts, grupos |
| `system_monitoring` | `/system_monitoring/mcp` | Health check, logs, auditoria |
| `user_management` | `/user_management/mcp` | Usuários, times, RBAC |
| `security_compliance` | `/security_compliance/mcp` | Credenciais, tipos, compliance |
| `platform_configuration` | `/platform_configuration/mcp` | Settings, EEs, licenças |

## Modelo de Permissões (Dual Layer)

```
Nível 1 — Server Level (admin define no deploy):
  mcp_allow_write_operations=false  → read-only (padrão e seguro)
  mcp_allow_write_operations=true   → read-write (pode executar jobs)

  ⚠️  Read-write: AI agent pode tomar ações reais no ambiente.
      LLMs podem alucinar comandos → usar com cautela.

Nível 2 — User Level (token do usuário):
  O AI agent herda exatamente as permissões do usuário cujo token foi usado.
  Se o token só tem acesso ao inventory "rede", o AI agent não vê "database".
```

## Deploy no Containerizado

```ini
# Adicionar ao inventory file
[ansiblemcp]
aap.exemplo.org

[all:vars]
# Permissões
mcp_allow_write_operations=false    # false = read-only (recomendado)
mcp_ignore_certificate_errors=false

# TLS (se usando certificado customizado)
mcp_tls_cert=/etc/ssl/certs/aap.crt
mcp_tls_key=/etc/ssl/private/aap.key
```

```bash
# Verificar que o pod está rodando
podman ps | grep ansiblemcp
# Porta padrão: 8448 (HTTPS)
# URL base: https://aap.exemplo.org:8448
```

## Deploy no OpenShift (Operator)

```yaml
# Adicionar ao Custom Resource do AAP
spec:
  mcp:
    disabled: false
    allow_write_operations: false
```

> Para mudar permissões após deploy: deletar `AnsibleMCPServer` CR e recriar (reconciliação automática).

```bash
# Verificar URL do MCP server
# OCP: Networking → Routes → aap-mcp → Location
```

## Criar Token de API para o MCP

```
Gateway → Access Management → Users → Selecionar usuário
  → Tokens → Create Token
    Application: (deixar em branco para PAT)
    Scope: Write (se read-write) ou Read (se read-only)
  → Copiar o token (exibido apenas uma vez)
```

## Conectar AI Agent ao MCP Server

### Cursor (mcp.json)

```json
{
  "mcpServers": {
    "aap-jobs": {
      "type": "http",
      "url": "https://aap.exemplo.org:8448/job_management/mcp",
      "headers": {
        "Authorization": "Bearer ${env:AAP_MCP_TOKEN}"
      }
    },
    "aap-inventory": {
      "type": "http",
      "url": "https://aap.exemplo.org:8448/inventory_management/mcp",
      "headers": {
        "Authorization": "Bearer ${env:AAP_MCP_TOKEN}"
      }
    },
    "aap-monitor": {
      "type": "http",
      "url": "https://aap.exemplo.org:8448/system_monitoring/mcp",
      "headers": {
        "Authorization": "Bearer ${env:AAP_MCP_TOKEN}"
      }
    }
  }
}
```

> **Nome do servidor:** máx 20 caracteres (AI agents combinam nome + tool name, limite de 64).

### Testar conexão

```
No chat do AI agent:
"What MCP tools are available for my Ansible Automation Platform?"
→ Deve retornar lista de toolsets disponíveis

"Give me a list of my Ansible Automation Platform jobs."
→ Lista os jobs reais do AAP
```

## Troubleshooting

| Erro | Causa | Solução |
|---|---|---|
| HTTP 406 | Output não é JSON | Pedir ao AI agent para usar formato JSON |
| HTTP 400 | Self-signed certificate | `mcp_ignore_certificate_errors=true` (OCP: extra_settings) |
| Permissões não aplicadas | Mudança pós-deploy | Deletar e recriar AnsibleMCPServer CR |
