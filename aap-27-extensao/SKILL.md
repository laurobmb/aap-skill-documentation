---
name: aap-27-extensao
description: >-
  Referência técnica de extensão do AAP 2.7 cobrindo: (1) Ansible MCP Server standalone —
  novo componente deployável que expõe o AAP como ferramentas MCP para AI agents externos
  (disponível no container e operator, dois servidores: ansible-mcp-lightspeed e
  ansible-mcp-controller); (2) Automation Intelligent Assistant (chatbot na UI do AAP)
  com LLMs e integração MCP; (3) BYOK — Bring Your Own Knowledge para customizar a base
  de conhecimento do chatbot com documentação interna; (4) Ansible Lightspeed Coding
  Assistant no VS Code; (5) Integrações — HashiCorp Terraform e Vault.
  Use quando precisar deployar o MCP server AAP para um AI agent externo, instalar o
  chatbot inteligente no AAP, construir uma base de conhecimento interna para o chatbot
  (BYOK), ativar o Lightspeed no VS Code, ou integrar Terraform com o AAP.
disable-model-invocation: true
---

# AAP 2.7 — Extensão: MCP, IA e Integrações

## Índice

| Capítulo | Conteúdo |
|---|---|
| [ch01-mcp-server](chapters/ch01-mcp-server.md) | Ansible MCP Server standalone (novo 2.7), deploy, toolsets, permissões |
| [ch02-intelligent-assistant](chapters/ch02-intelligent-assistant.md) | Chatbot AAP, deploy OCP/containerizado, BYOK, LLM providers |
| [ch03-lightspeed-coding](chapters/ch03-lightspeed-coding.md) | Lightspeed Coding Assistant no VS Code, configuração, trial |
| [ch04-integracoes](chapters/ch04-integracoes.md) | Terraform + Vault, HashiCorp integrations |

## Uso Rápido

- **Conectar AI agent externo ao AAP via MCP:** → ch01
- **Instalar chatbot na UI do AAP (OCP):** → ch02
- **Customizar chatbot com docs internas (BYOK):** → ch02
- **Ativar Lightspeed no VS Code:** → ch03
- **Integrar Terraform com AAP:** → ch04
