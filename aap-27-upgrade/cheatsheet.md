# Cheatsheet — AAP 2.7 Upgrade e Migração

## Upgrade Path

```
2.4 → 2.6 → 2.7
2.5 → 2.6 → 2.7
2.6 → 2.7   ✅ direto
RPM → Container/Operator 2.7 ✅
```

## Breaking Changes 2.7

```
❌ RPM installer → removido
❌ Acesso direto controller/hub/EDA → HTTP 401
❌ Component-level PATs → removidos
❌ LDAP/SAML no Controller → reconfigurar no Gateway
❌ Basic auth direta → removida
```

## Checklist PRÉ-Upgrade

```
☐ Versão atual: 2.6.x (obrigatório)
☐ Detectar acesso direto: aap-detect-direct-component-access
☐ Listar tokens de componente ativos
☐ Verificar usuários sem conta no Gateway
☐ Exportar config LDAP/SAML do Controller
☐ Fazer BACKUP: ansible-playbook backup.yml
```

## Detectar Acesso Direto

```bash
uvx --from "git+https://github.com/ansible/aap-detect-direct-component-access" \
  aap-detect-direct-component-access /path/to/sosreport/
```

## Checklist PÓS-Upgrade

```
☑ Recriar PATs no Gateway
☑ Atualizar URLs: componente.host → gateway.host
☑ Atualizar CaC: ansible.controller → ansible.platform
☑ Re-autenticar: podman login aap.empresa.org
☑ Reconfigurar LDAP/SAML no Gateway
☑ Testar todos os autenticadores
```

## Migração de URL

```
ANTES: curl https://controller.empresa.org/api/v2/...
DEPOIS: curl https://aap.empresa.org/api/controller/v2/...
         -H "Authorization: Bearer <gateway_token>"
```
