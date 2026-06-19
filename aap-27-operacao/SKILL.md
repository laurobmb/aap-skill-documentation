---
name: aap-27-operacao
description: >-
  Referência técnica de operação e otimização do AAP 2.7. Cobre performance tuning
  (vertical vs horizontal scaling, forks, capacidade de jobs, EDA scaling, pod resources
  no OCP), observabilidade (metrics-utility para billing reports, Automation Analytics,
  Automation Dashboard), e troubleshooting (logs por componente, sosreport, must-gather,
  erros comuns de auth e conectividade, depuração de mesh e EDA).
  Use quando precisar dimensionar o AAP para alta carga, calcular capacidade de jobs
  por nó, escalar o EDA horizontalmente, gerar relatórios de consumo (metrics-utility),
  configurar o Automation Dashboard, ou diagnosticar problemas de performance e falhas.
disable-model-invocation: true
---

# AAP 2.7 — Operação, Performance e Observabilidade

## Índice

| Capítulo | Conteúdo |
|---|---|
| [ch01-performance](chapters/ch01-performance.md) | Vertical/horizontal scaling, forks, capacidade de jobs, EDA scaling |
| [ch02-ocp-tuning](chapters/ch02-ocp-tuning.md) | Pod resources, node selectors, control plane tuning no OCP |
| [ch03-observabilidade](chapters/ch03-observabilidade.md) | metrics-utility, Automation Analytics, Dashboard, billing reports |
| [ch04-troubleshooting](chapters/ch04-troubleshooting.md) | Logs, sosreport, must-gather, erros comuns, Mesh/EDA debug |

## Uso Rápido

- **Quantos jobs concorrentes meu nó suporta?** → ch01 (capacity algorithm)
- **Escalar EDA horizontalmente:** → ch01
- **Configurar resource requests/limits de pods no OCP:** → ch02
- **Gerar relatório de consumo para billing:** → ch03
- **Coletar logs para suporte Red Hat:** → ch04
