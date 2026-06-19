---
name: aap-27-seguranca
description: >-
  Referência técnica de segurança e administração do AAP 2.7. IMPORTANTE: todas as
  autenticações passam pelo Platform Gateway — autenticação direta aos componentes
  (controller, hub, EDA) foi removida em 2.7. Cobre autenticação centralizada (LDAP,
  SAML, OIDC, GitHub, GitHub Enterprise, Google OAuth2, RADIUS, TACACS+, Keycloak,
  Microsoft Entra ID), authenticator maps e triggers, RBAC (organizations, teams,
  roles, role assignment), credenciais e credential types, Personal Access Tokens
  (PATs criados agora APENAS via Platform Gateway — tokens a nível de componente
  foram removidos), OAuth2 applications, timeout de rotas do gateway, e administração
  (awx-manage, backup, restore, logs).
  Use quando precisar configurar LDAP no 2.7, entender authenticator maps para
  controlar acesso, criar roles customizadas, gerenciar credenciais, migrar tokens do
  controller para o gateway, resolver 401 após upgrade, ou entender o novo modelo de
  autenticação centralizada.
disable-model-invocation: true
---

# AAP 2.7 — Segurança e Administração

> **Breaking change 2.7:** Autenticação direta a controller/hub/EDA removida. Tudo passa pelo Platform Gateway.

## Índice

| Capítulo | Conteúdo |
|---|---|
| [ch01-autenticacao](chapters/ch01-autenticacao.md) | Mudanças 2.7, tipos de autenticação, pluggable auth, LDAP, SAML, OIDC |
| [ch02-authenticator-maps](chapters/ch02-authenticator-maps.md) | Authenticator maps, triggers, mapeamento de orgs/times/roles |
| [ch03-rbac](chapters/ch03-rbac.md) | Organizations, teams, roles, permissões, RBAC por recurso |
| [ch04-credentials](chapters/ch04-credentials.md) | Tipos de credenciais, criação, External Secret Providers, Vault |
| [ch05-tokens-admin](chapters/ch05-tokens-admin.md) | PATs, OAuth2 apps, tokens, awx-manage, backup/restore |

## Uso Rápido

- **Configurar LDAP no AAP 2.7:** → ch01
- **Mapear grupos LDAP para Organizations:** → ch02
- **Criar roles customizadas e atribuir a times:** → ch03
- **Credenciais do HashiCorp Vault (External Secret):** → ch04
- **Criar PAT / token de API no 2.7:** → ch05
- **Erro 401 após upgrade de 2.6 → 2.7:** → ch01 (mudanças de autenticação)
