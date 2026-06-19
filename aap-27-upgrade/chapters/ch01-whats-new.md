# Ch01 — O que Há de Novo no AAP 2.7

## Breaking Changes (Ações Obrigatórias)

| O que mudou | Impacto | Ação necessária |
|---|---|---|
| **RPM installer removido** | 2.7 não tem instalador RPM | Migrar para Container ou Operator |
| **Acesso direto a componentes removido** | HTTP 401 em qualquer acesso direto | Usar URLs do Gateway |
| **Tokens de componente removidos** | Controller PATs não funcionam | Recriar tokens no Gateway |
| **LDAP/SAML do Controller removidos** | Auth de 3rd party não funciona no Controller | Reconfigurar no Gateway |
| **Fallback auth do Controller removido** | Usuários sem conta no Gateway não conseguem logar | Criar contas no Gateway ANTES do upgrade |
| **Basic auth em componentes removida** | Scripts com -u user:pass em APIs diretas | Usar tokens Bearer via Gateway |

## Novidades do AAP 2.7

### Ansible MCP Server (Tech Preview)

Novo componente standalone que expõe o AAP como servidor MCP para AI agents externos (Cursor, Claude, etc.):

```ini
# Inventory
[ansiblemcp]
aap.exemplo.org

[all:vars]
mcp_allow_write_operations=false
```

### BYOK — Bring Your Own Knowledge (Tech Preview)

Extensão do Intelligent Assistant com documentação interna via RAG pipeline.

### RHEL 10 Suportado

Agora suportado como OS para instalação containerizada do AAP 2.7.

### EE para RHEL 10

Novos base images disponíveis:
- `ee-minimal-rhel10:latest`
- `ee-supported-rhel10:latest`

### Redis HA (Sentinel)

Suporte a Redis em modo sentinel para enterprise topology:
```ini
redis_mode=sentinel
```

### PostgreSQL 17 Externo Suportado

Além de 15 e 16, agora 17 é suportado para PostgreSQL externo.

### Platform Gateway como Único Ponto de Entrada

Toda arquitetura consolidada: não há mais rotas diretas para Controller/Hub/EDA externamente.

## APIs — Mudanças

| Componente | Antes (2.4) | 2.5–2.6 | 2.7 (obrigatório) |
|---|---|---|---|
| Controller | `/api/v2/` | `/api/controller/v2/` | `https://gateway.host/api/controller/v2/` |
| Hub | `/api/automationhub/` | `/api/galaxy/v1/` | `https://gateway.host/api/galaxy/v1/` |
| Gateway | — | `/api/gateway/v1/` | `https://gateway.host/api/gateway/v1/` |

> Requisições diretas a `controller.exemplo.org/api/...` → HTTP 401 no AAP 2.7.

## Upgrade Path Suportado

```
2.4 → 2.6 → 2.7   (não existe upgrade direto 2.4→2.7)
2.5 → 2.6 → 2.7   (não existe upgrade direto 2.5→2.7)
2.6 → 2.7          ✅ suportado direto
```
