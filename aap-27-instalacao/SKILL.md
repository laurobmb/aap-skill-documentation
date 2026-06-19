---
name: aap-27-instalacao
description: >-
  Referência técnica do AAP 2.7 cobrindo planejamento e instalação. IMPORTANTE:
  o instalador RPM foi removido no 2.7 — apenas Container (RHEL/Podman) e Operator
  (OpenShift) são suportados. Cobre topologias testadas (Container Growth, Container
  Enterprise, Operator Growth, Operator Enterprise), requisitos de hardware por VM,
  exemplo completo de inventory file, portas e protocolos de rede por componente,
  Redis (standalone vs HA), PostgreSQL suportado (15, 16, 17 externo), e variáveis
  de inventário essenciais por serviço (gateway, controller, hub, eda, database).
  Use quando precisar escolher a topologia correta, dimensionar VMs, configurar o
  inventory file para instalação containerizada ou Operator, liberar portas no firewall,
  entender Redis HA, ou verificar requisitos de PostgreSQL externo.
disable-model-invocation: true
---

# AAP 2.7 — Instalação e Planejamento

> **Breaking change 2.7:** instalador RPM removido. Usar Container (RHEL+Podman) ou Operator (OpenShift).

## Índice

| Capítulo | Conteúdo |
|---|---|
| [ch01-topologias](chapters/ch01-topologias.md) | Modelos de instalação, topologias testadas, requisitos de hardware |
| [ch02-inventario](chapters/ch02-inventario.md) | Inventory file completo, variáveis por componente, Redis/PostgreSQL |
| [ch03-operator](chapters/ch03-operator.md) | Instalação no OpenShift (Operator), custom resource, nonfunctional requirements |
| [ch04-rede](chapters/ch04-rede.md) | Portas e protocolos por componente, diagrama de rede, automação mesh |
| [ch05-pos-instalacao](chapters/ch05-pos-instalacao.md) | Subscription, registry, seed de coleções, verificação |

## Uso Rápido

- **Qual topologia usar para POC/início?** → ch01 (Container Growth — 1 VM)
- **Qual topologia para produção?** → ch01 (Container Enterprise — 11 VMs)
- **Inventory file completo para containerizado:** → ch02
- **Instalar no OpenShift:** → ch03
- **Quais portas liberar no firewall?** → ch04
- **Redis HA vs standalone:** → ch02
