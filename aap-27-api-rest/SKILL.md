---
name: aap-27-api-rest
description: >-
  Referência técnica da API REST do AAP 2.7. Cobre o endpoint base da plataforma
  (via Platform Gateway /api/controller/v2/), métodos HTTP (GET/PUT/POST), navegação
  pelo API Browser, ordenação e paginação, filtros avançados (field lookups: exact,
  contains, regex, gt/lt, isnull, in; OR/AND/NOT/chain filters), named URLs para
  identificar recursos por nome (sem PK numérico), autenticação (Session, Basic,
  OAuth2 PAT, SSO), criação e uso de tokens OAuth2, desabilitar basic auth, habilitar
  tokens para usuários externos, e a API do metrics service (/api/metrics/v1/health/,
  /api/v1/tasks/, /api/v1/feature-flags/). Use quando precisar automatizar integração
  com o Controller via scripts Python/curl, criar tokens para CI/CD, consultar jobs via
  API, ou monitorar saúde da plataforma programaticamente.
disable-model-invocation: true
---

# AAP 2.7 — API REST: Referência e Integração

## Índice

| Capítulo | Conteúdo |
|---|---|
| [ch01-navegacao](chapters/ch01-navegacao.md) | Endpoints base, API browser, versões, métodos HTTP, named URLs |
| [ch02-filtros](chapters/ch02-filtros.md) | Field lookups (exact/contains/regex/gt/lt), OR/AND/NOT/chain, paginação |
| [ch03-autenticacao](chapters/ch03-autenticacao.md) | Session, Basic (desabilitar), OAuth2 PAT (criar, usar, revogar), SSO |
| [ch04-exemplos](chapters/ch04-exemplos.md) | Exemplos práticos: listar jobs, lançar JT, criar inventário, filtrar hosts |
| [ch05-metrics-api](chapters/ch05-metrics-api.md) | Metrics service API: /health/, /tasks/, /feature-flags/, autenticação |

## Uso Rápido

- **Lançar Job Template via API:** → ch04
- **Filtrar jobs por status e data:** → ch02 + ch04
- **Criar token OAuth2 para CI/CD:** → ch03
- **Autenticar com curl:** → ch03
- **API de saúde da plataforma:** → ch05
- **Named URL (sem PK numérico):** → ch01
