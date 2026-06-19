# Patterns — AAP 2.7 Segurança

## Padrão 1: LDAP com Segregação por Grupo → Organização

```
Cenário: 30 orgs, grupos LDAP por time. Usuário só acessa sua org.

Configuração (Gateway → Authentication → LDAP):
  1. Configurar LDAP com User Search + Group Search
  2. Map 1: Allow → Never         (bloqueia tudo por padrão)
  3. Map 2: Allow → Group         (libera apenas grupos aap-*)
     Groups: cn=aap-users,ou=groups,dc=empresa,dc=org
  4. Map 3: Organization → Group  (mapeia grupo → org)
     trigger_groups: cn=time-financeiro,...
     target: Organization: Financeiro
  5. Repetir Map 3 para cada grupo/org
```

## Padrão 2: Credencial Privilegiada Compartilhada Sem Exposição

```
Cenário: credencial do vCenter usada por 5 orgs, sem nenhuma ver a senha.

Solução:
  1. Criar credencial "vcenter-prod" na Org "Platform"
  2. Para cada org que precisa: Assign role "Use" → Org/Time destino
  3. Desenvolvedores usam a credencial pelo nome em JTs
  4. Valor da senha nunca visível na UI para não-admins da Platform
```

## Padrão 3: Migração de Tokens Controller → Gateway (Upgrade 2.6 → 2.7)

```
Situação: usuários e CI/CD usando tokens do controller. Após upgrade → 401.

Passos:
  1. Identificar todas as integrações com tokens de componente
     (Jenkins, GitLab CI, scripts, Terraform provider)
  2. Para cada integração:
     a. Criar novo PAT no Gateway via UI ou API
     b. Atualizar integração com novo token
     c. Revogar token antigo do controller
  3. Validar: curl -H "Authorization: Bearer <novo_token>" .../api/gateway/v1/ping/
```

## Padrão 4: Auditoria de Segurança Pós-Upgrade

```bash
# 1. Verificar emails modificados (possível account hijacking)
podman exec -it aap-gateway aap-gateway-manage detect_changed_emails --audit

# 2. Listar tokens antigos de componente ainda ativos
curl .../api/controller/v2/tokens/ -H "Authorization: Bearer <admin>"
# → Migrar ou revogar

# 3. Verificar autenticadores ativos
curl .../api/gateway/v1/authenticators/ -H "Authorization: Bearer <admin>"
```
