# Ch03 — RBAC: Organizations, Teams, Roles

## Modelo de Hierarquia

```
Platform (superuser)
  └── Organization (org-admin)
        ├── Teams (team-admin)
        │     └── Users
        └── Resources: Projects, Inventories, Credentials, JTs, Workflows
```

## Criar Organization

```
Gateway → Access Management → Organizations → Create Organization
```

```yaml
# Via CaC (ansible.platform)
ansible.platform.organization:
  name: "Financeiro"
  description: "Equipe financeira"
  max_hosts: 500
  default_environment: "ee-supported-rhel9:latest"
```

## Criar Team

```
Gateway → Access Management → Teams → Create Team
  Selecionar Organization
```

## Criar e Atribuir Roles

### Roles Built-in por Recurso

| Role | Escopo | Permissões |
|---|---|---|
| Admin | Org / Recurso | CRUD total |
| Member | Org | Pertencer à organização |
| Execute | Job Template | Disparar jobs |
| Use | Credential / EE / Inventory | Usar em JTs |
| Read | Qualquer | Visualizar |
| Auditor | Sistema | Visualizar tudo (read-only global) |

### Criar Role Customizada

```
Gateway → Access Management → Roles → Create Role
  → Definir nome e permissões granulares
```

### Atribuir Role a Usuário

```
Gateway → Access Management → Users → Selecionar usuário
  → Roles → Assign Roles
```

### Atribuir Role a Time (Bulk)

```
Gateway → Access Management → Teams → Selecionar time
  → Roles → Assign Roles
  → Selecionar recurso e role
```

### Via API

```bash
# Atribuir role "Execute" num Job Template para um time
curl -X POST https://aap.exemplo.org/api/gateway/v1/job_templates/<id>/roles/ \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"role": "execute", "actors": [{"type": "team", "id": <team_id>}]}'
```

## RBAC Multi-Org: Objeto Compartilhado

Credenciais, EEs e inventários podem ser compartilhados **entre organizações** via role `Use`:

```
Organization: Platform (dono)
  Credential: vcenter-prod
    → Assign role "Use" → Organization: Financeiro
    → Assign role "Use" → Organization: Infra
```

Assim o time Financeiro pode usar a credencial `vcenter-prod` em seus JTs, sem ver a senha.

## Organizações — Configurações Relevantes

| Campo | Descrição |
|---|---|
| `max_hosts` | Limite de hosts gerenciados pela org |
| `default_environment` | EE padrão para JTs sem EE explícito |
| Assign EE to Org | Liberar EE para uso por todos os JTs da org |
| Assign Notifier to Org | Notificações padrão da org |

## RBAC via CaC (ansible.platform)

```yaml
# Atribuir role "Execute" num JT para um time
ansible.platform.role_team_assignment:
  team: "Financeiro-Devs"
  role_definition: "JobTemplate Execute"
  object_ansible_id: "{{ jt_id }}"

# Ou para usuário:
ansible.platform.role_user_assignment:
  user: "joao.silva"
  role_definition: "Organization Member"
  object_ansible_id: "{{ org_id }}"
```
