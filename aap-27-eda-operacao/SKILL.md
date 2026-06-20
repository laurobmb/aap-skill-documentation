---
name: aap-27-eda-operacao
description: >-
  Referência técnica de operação e administração do Event-Driven Ansible no AAP 2.7.
  Cobre gerenciamento de projetos EDA (SCM, sync, edição), configuração e ciclo de vida
  de Rulebook Activations (criar, habilitar/desabilitar, reiniciar, duplicar, editar, deletar,
  políticas de restart, log levels), Event Streams (6 tipos de autenticação: HMAC, Basic,
  Token, OAuth2, OAuth2 JWT, ECDSA, mTLS; criar credencial, criar stream, headers HTTP,
  UUIDs estáticos para DR, modo test), Rule Audit (visualizar eventos, ações disparadas,
  rastreamento por tid/aiid), e troubleshooting detalhado (filtro de logs com IDs de rastreamento,
  ativações presas em Pending, restart loop, falhas de processamento de eventos, event streams
  não enviando). Use quando precisar configurar uma integração webhook do GitHub/GitLab com EDA,
  criar event streams para Dynatrace ou ServiceNow, diagnosticar rulebook activations que falham
  ou reiniciam, auditar eventos disparados, ou configurar EDA em OCP com Route.
disable-model-invocation: true
---

# AAP 2.7 — Event-Driven Ansible: Operação e Administração

## Índice

| Capítulo | Conteúdo |
|---|---|
| [ch01-projetos-des](chapters/ch01-projetos-des.md) | Projetos EDA (SCM, sync, edição), Decision Environments, event sources suportados |
| [ch02-activations](chapters/ch02-activations.md) | Criar/editar/duplicar/deletar activations, restart policy, log level, habilitar/desabilitar |
| [ch03-event-streams](chapters/ch03-event-streams.md) | Tipos de event stream (HMAC, Basic, Token, OAuth2, ECDSA, mTLS), criar credencial e stream, headers HTTP, UUID estático |
| [ch04-audit-ocp](chapters/ch04-audit-ocp.md) | Rule Audit (eventos, ações), webhooks em OCP via Route, kubectl port-forward |
| [ch05-troubleshooting](chapters/ch05-troubleshooting.md) | Log IDs (rid/tid/aiid), filtro de logs, pending, restart loop, sem eventos, mTLS + load balancer |

## Uso Rápido

- **Criar Rulebook Activation completa:** → ch02
- **Event Stream para webhook do GitHub (HMAC):** → ch03
- **Event Stream para ServiceNow ou Dynatrace:** → ch03
- **Activation presa em Pending:** → ch05
- **Activation em restart loop:** → ch05
- **Auditoria de eventos e ações disparadas:** → ch04
- **Expor webhook EDA em OCP (Route):** → ch04
