---
name: aap-27-upgrade
description: >-
  Referência técnica de upgrade, migração e novidades do AAP 2.7. IMPORTANTE: somente
  upgrade direto de 2.6 → 2.7 é suportado (2.4/2.5 → primeiro ir para 2.6). RPM removido
  (apenas Container e Operator). Toda autenticação direta a componentes removida — apenas
  via Platform Gateway. Cobre: checklist pré-upgrade, ferramenta de detecção de acesso
  direto (aap-detect-direct-component-access), procedimento de upgrade containerizado e
  OCP, migração de autenticação (LDAP/SAML movidos para Gateway), mudanças de API,
  upgrade de componentes extras (portal, dashboard, plug-ins Developer Hub),
  migração de topologia RPM→Container, e o que há de novo no 2.7.
  Use quando planejar ou executar um upgrade de 2.6 para 2.7, verificar breaking changes,
  migrar scripts/CaC para usar gateway URLs, ou entender as novidades da versão.
disable-model-invocation: true
---

# AAP 2.7 — Upgrade, Migração e Novidades

## Índice

| Capítulo | Conteúdo |
|---|---|
| [ch01-whats-new](chapters/ch01-whats-new.md) | Novidades do AAP 2.7 (breaking changes, novos componentes) |
| [ch02-upgrade-checklist](chapters/ch02-upgrade-checklist.md) | Checklist pré e pós-upgrade 2.6→2.7, detecção de acesso direto |
| [ch03-upgrade-proc](chapters/ch03-upgrade-proc.md) | Procedimento de upgrade containerizado e OCP |
| [ch04-migracao](chapters/ch04-migracao.md) | Migração RPM→Container, auth providers, CaC, API changes |

## Uso Rápido

- **O que mudou no 2.7 vs 2.6?** → ch01
- **Checklist antes de fazer upgrade:** → ch02
- **Detectar scripts com acesso direto antes do upgrade:** → ch02
- **Executar upgrade de instalação containerizada:** → ch03
- **Migrar LDAP/SAML do Controller para Gateway:** → ch04
- **Atualizar CaC/scripts para gateway URLs:** → ch04
