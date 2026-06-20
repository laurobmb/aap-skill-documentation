# Ch02 — Namespaces, Times e Aprovação de Coleções

## O que são Namespaces?

Namespaces são localizações únicas no Hub para publicar e distribuir coleções. Organizam o conteúdo por equipe, produto ou domínio.

```
Automation Content → Namespaces

Exemplos:
  empresa_infra    → coleções do time de infraestrutura
  empresa_network  → coleções do time de redes
  empresa_windows  → coleções do time Windows
```

## Criar Namespace

```
Automation Content → Namespaces → Create namespace

Campos:
  Name:         empresa_infra   (lowercase, underscores permitidos)
  Description:  Coleções do time de infraestrutura
  Company:      Empresa SA
  Logo URL:     (opcional)
  Resources:    (texto markdown com documentação)

→ Create namespace
→ Team Access tab → Add roles → selecionar time e papel
```

## Criar Time e Atribuir Permissões ao Namespace

```
# 1. Criar time
Access Management → Teams → Create team
  Name: Content Engineering - Infraestrutura
  Organization: Infraestrutura

# 2. Atribuir permissões ao namespace
Automation Content → Namespaces → [namespace] → Team Access
  → Add roles → selecionar time → Next
  → selecionar papel (ex: Namespace Publisher, Namespace Editor) → Finish
```

**Papéis disponíveis para Namespaces:**

| Papel | Permissões |
|---|---|
| Namespace Owner | Total: editar, deletar, gerenciar times |
| Namespace Publisher | Upload de coleções para o namespace |
| Namespace Editor | Editar metadados do namespace |

## Upload de Coleção para Namespace

```
# Via UI:
Automation Content → Namespaces → [namespace]
  → Collections tab → Upload collection
  → Browse: selecionar arquivo .tar.gz
  → Repository: Staging repos (para aprovação) | Repositories without pipeline
  → Upload collection

# Via CLI:
ansible-galaxy collection build  # gera <namespace>-<name>-<version>.tar.gz
ansible-galaxy collection publish <namespace>-<name>-<version>.tar.gz \
  --server https://aap.empresa.org/api/galaxy/
```

> **Limite:** Máx 20MB para console.redhat.com / Ansible Galaxy. Para Hub privado, evitar >200MB — usar EE para ambientes completos.

## Verificar Importação (logs)

```
Automation Content → Namespaces → [namespace]
  → More Actions (⋮) → Imports

Status:
  ✓ completed    → pronto para aprovação
  ✗ failed       → verificar logs de import para erros de validação
  ⏳ waiting for approval → aguardando revisão do administrador
```

## Fluxo de Aprovação de Coleções

```
Developer                Hub Admin
    │                        │
    ├─ upload → staging ─────┤
    │                        ├─ Collection Approvals → "Needs review"
    │                        ├─ Revisar conteúdo
    │                        ├─ ✓ Aprovar (👍) → move para Published
    │                        │   ou
    │                        └─ ✗ Rejeitar (👎) → move para Rejected
```

```
# Aprovar/Rejeitar:
Automation Content → Collection Approvals
  → filtrar por Namespace / Repository / Status
  → 👍 aprovar | 👎 rejeitar
```

**Auto-approve** (bypass do fluxo de aprovação):
```
# Quando habilitado em um repositório:
qualquer upload ao staging → promovido automaticamente para todos os repos "approved"
```

## Deletar Coleção (opções)

```
Automation Content → Collections → [coleção] → More Actions (⋮)

Opções:
  Delete version from system        → remove versão específica de TODA a instância
  Delete version from repository    → remove versão do repositório atual apenas
  Delete entire collection from repo → remove todas as versões do repositório atual
  Delete entire collection from system → remove todas as versões de toda a instância

⚠️  Verificar aba Dependencies antes de deletar!
```

## Deletar Namespace

```
Automation Content → Namespaces → [namespace] → More Actions (⋮) → Delete namespace

⚠️  Requer que nenhuma coleção do namespace tenha dependências.
Se o botão Delete estiver desabilitado → deletar dependências primeiro.
```
