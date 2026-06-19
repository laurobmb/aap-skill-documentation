# Ch02 — Authenticator Maps e Triggers

## O que são Authenticator Maps

Authenticator maps controlam **o que um usuário pode fazer** após se autenticar. São regras processadas em ordem (como firewall rules) que determinam:

- Se o usuário tem **acesso** ao sistema (Allow)
- A qual **organização** o usuário pertence
- A qual **time** o usuário pertence
- Qual **role** o usuário recebe
- Se o usuário é **superuser**

## Tipos de Authenticator Maps

| Tipo | O que controla |
|---|---|
| **Allow** | Acesso ao sistema (allow ou deny) |
| **Organization** | Associação do usuário a uma organização |
| **Team** | Associação do usuário a um time |
| **Role** | Atribuição de role (ex: System Auditor) |
| **Is Superuser** | Se o usuário é superuser |

## Triggers (Condições de Disparo)

| Trigger | Quando aplica |
|---|---|
| **Always** | Sempre — sem condição |
| **Never** | Nunca — útil para desabilitar temporariamente |
| **Group** | Com base nos grupos do usuário no IdP (LDAP groups, GitHub teams, etc.) |
| **Attribute** | Com base em atributos do usuário (email, department, etc.) |

## Lógica de Processamento (Ordem Importa)

```
Login → Autenticação → Permissions iniciais em memória:
  Access allowed = True
  Superuser = Undefined
  Teams = None

→ Processar maps em ordem:
  Map 1: Allow (never)     → Access allowed = False
  Map 2: Allow (group X)   → Access allowed = True  (se usuário está no grupo X)
  Map 3: Superuser (attr)  → Superuser = True/False  (ou Skipped se Revoke=off)
  Map 4: Team (group Y)    → Teams = ["Meu Time"]

→ Reconciliar com permissões existentes no banco
```

**ALLOW** — match encontrado, permissão concedida  
**SKIPPED** — sem match, continua para o próximo map  
**SKIPPED com Revoke=true** → **DENY** — sem match, permissão revogada

## Exemplos Práticos

### Mapear Grupo LDAP para Organização

```
Authenticator: LDAP
Map type: Organization
Trigger: Group
  Operation: or
  Groups: cn=equipe-infra,ou=groups,dc=empresa,dc=org
Organization: Infraestrutura
```

### Mapear Grupo LDAP para Time Específico

```
Authenticator: LDAP
Map type: Team
Trigger: Group
  Operation: or
  Groups: cn=admins-aap,ou=groups,dc=empresa,dc=org
Team: AAP-Admins
Organization: Infraestrutura
```

### Bloquear Acesso (exceto membros de grupo)

```
Map 1: Allow → Trigger: Never         (bloqueia todos por padrão)
Map 2: Allow → Trigger: Group         (libera apenas o grupo permitido)
         Groups: cn=aap-users,ou=groups,dc=empresa,dc=org
```

### Conceder Superuser para Grupo Específico

```
Map type: Is Superuser
Trigger: Group
  Groups: cn=aap-admins,ou=groups,dc=empresa,dc=org
Revoke: true  (remove superuser de quem NÃO está no grupo)
```

### Mapear por Atributo (ex: departamento)

```
Map type: Organization
Trigger: Attribute
  Attribute: department
  Comparison: equals
  Value: TI-Infraestrutura
Organization: Infra
```

## Configurar Authenticator Maps na UI

```
Gateway → Access Management → Authentication Methods
  → Selecionar autenticador
  → Mapping
  → Add Rule
    → Tipo: Organization / Team / Role / Allow / Is Superuser
    → Trigger: Group / Attribute / Always / Never
    → Definir target (org, time, role)
    → Ordem (número menor = processa primeiro)
```

## Grupos LDAP — Boas Práticas

- Inserir grupos em **lowercase**: `cn=aap-admins,dc=empresa,dc=org`
- Usar `or` para grupos alternativos, `and` quando o usuário precisa estar em todos
- Testar mapeamento com `aap-gateway-manage detect_changed_emails --audit`
