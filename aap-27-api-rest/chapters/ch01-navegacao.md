# Ch01 — Navegação pela API e Named URLs

## Endpoint Base (AAP 2.7 via Platform Gateway)

```
# Listar versões disponíveis
GET https://<gateway>/api/

# API do Controller (recomendada: v2)
GET https://<gateway>/api/controller/v2/

# API do Gateway
GET https://<gateway>/api/gateway/v1/

# API do Hub (Galaxy NG)
GET https://<gateway>/api/galaxy/

# API do EDA
GET https://<gateway>/api/eda/v1/

# Configurações (apenas alterações do padrão)
GET https://<gateway>/api/gateway/v1/settings/
```

> **Todas as URIs sem "/" no final recebem redirect 301.**

## API Browser (Interface Visual)

```
# Acessar no browser:
https://<gateway>/api/controller/v2/

Funcionalidades:
  - Click no v2 para ver a versão atual
  - Click em "?" para documentação do endpoint
  - Campos de texto para PUT/POST com JSON
  - Suporte a GET/PUT/POST/DELETE
```

> Dica: usar Firebug, Chrome DevTools ou Charles Proxy para inspecionar chamadas que a UI faz — útil para descobrir a sequência correta de chamadas.

## Métodos HTTP

| Método | Uso |
|---|---|
| **GET** | Ler recursos (listas ou objetos individuais) |
| **POST** | Criar novo recurso |
| **PUT** | Atualizar recurso específico por identificador (ou criar se ID conhecido) |
| **PATCH** | Atualizar parcialmente um recurso |
| **DELETE** | Deletar recurso |

> Formatos especiais preservados: `extra_vars` em YAML retornam como YAML; em JSON retornam como JSON.

## Named URLs — Identificar Recursos por Nome (sem PK)

```
# Acesso por PK numérico (padrão):
GET /api/controller/v2/job_templates/14/

# Acesso por nome (Named URL):
GET /api/controller/v2/job_templates/Deploy+App++Infraestrutura/
# Formato: <name>++<organization.name>

# Named URL para host:
GET /api/controller/v2/hosts/web01.empresa.org++inventario-linux++Infraestrutura/
# Formato: <name>++<inventory.name>++<organization.name>
```

**Formatos de Named URL por recurso:**

```
organizations:              <name>
teams:                      <name>++<organization.name>
credentials:                <name>++<credential_type.name>+<credential_type.kind>++<organization.name>
job_templates:              <name>++<organization.name>
workflow_job_templates:     <name>++<organization.name>
projects:                   <name>++<organization.name>
inventories:                <name>++<organization.name>
hosts:                      <name>++<inventory.name>++<organization.name>
groups:                     <name>++<inventory.name>++<organization.name>
users:                      <username>
instances:                  <hostname>
```

> Consultar `/api/controller/v2/settings/named-url/` → `NAMED_URL_FORMATS` para a lista completa e atual.

## Ver Named URL de um Objeto

```bash
# O campo named_url aparece somente na view de detalhe (não na listagem):
curl -H "Authorization: Bearer <token>" \
  https://<gateway>/api/controller/v2/job_templates/14/ | jq '.named_url'
# Output: "Deploy App++Infraestrutura"
```

## Paginação

```bash
# API retorna max 25 resultados por padrão
# Parâmetros de paginação:
GET /api/controller/v2/jobs/?limit=100&offset=0
# page_size é alternativa para page-based pagination

# Resposta inclui:
{
  "count": 5000,
  "next": "/api/controller/v2/jobs/?limit=100&offset=100",
  "previous": null,
  "results": [...]
}
```
