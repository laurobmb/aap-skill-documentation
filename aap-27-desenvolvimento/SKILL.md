---
name: aap-27-desenvolvimento
description: >-
  Referência técnica de desenvolvimento de automações no AAP 2.7. Cobre ansible-dev-tools
  (conjunto de ferramentas de desenvolvimento: navigator, lint, molecule, creator),
  Ansible Content Navigator (CLI/TUI para playbooks em EEs), criação e publicação de
  coleções, Execution Environments (ansible-builder, EE definition YAML v3),
  Event-Driven Ansible (rulebooks, decision environments, event sources),
  Configuration as Code (CaC com ansible.platform collection),
  e a novidade do 2.7: MCP server integration em EEs — Ansible playbooks chamando
  MCP servers (GitHub, AWS, etc.) como ephemeral agents durante execução de jobs.
  Use quando precisar criar um EE customizado com MCP server, configurar ansible-navigator
  para desenvolvimento local, criar uma coleção com roles para o Private Hub, escrever
  rulebooks EDA, implementar CaC para múltiplas organizações, ou entender o workflow
  completo de desenvolvimento AAP.
disable-model-invocation: true
---

# AAP 2.7 — Desenvolvimento de Automações

## Índice

| Capítulo | Conteúdo |
|---|---|
| [ch01-dev-tools](chapters/ch01-dev-tools.md) | ansible-dev-tools, ansible-creator, navigator, workflow de desenvolvimento |
| [ch02-execution-environments](chapters/ch02-execution-environments.md) | Criação de EEs, ansible-builder, EE definition v3, MCP em EEs |
| [ch03-eda](chapters/ch03-eda.md) | Event-Driven Ansible, rulebooks, event sources, decision environments |
| [ch04-cac](chapters/ch04-cac.md) | Configuration as Code com ansible.platform, multi-org, GitOps |
| [ch05-colecoes](chapters/ch05-colecoes.md) | Criar coleções, roles, publicar no Private Hub |

## Uso Rápido

- **Criar EE com MCP server (GitHub, AWS):** → ch02
- **Configurar ansible-navigator para projeto:** → ch01
- **Escrever rulebook EDA para webhook:** → ch03
- **CaC multi-org com repo separado por org:** → ch04
- **Publicar coleção no Private Hub:** → ch05
