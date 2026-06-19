# Ch03 — Lightspeed Coding Assistant

## Visão Geral

O **Ansible Lightspeed Coding Assistant** (VS Code) é diferente do Intelligent Assistant (chatbot na UI):

| | Coding Assistant | Intelligent Assistant |
|---|---|---|
| **Interface** | VS Code (extensão Ansible) | UI do AAP (chatbot) |
| **Função** | Gerar tasks/playbooks no editor | Responder perguntas sobre o ambiente |
| **Modelo** | IBM watsonx Code Assistant | Qualquer LLM (OpenAI, RHEL AI, etc.) |
| **Fine-tuning** | Sim (com repos Git da org) | Não (usa RAG/BYOK) |

## Modelos de Deploy

| Modelo | Lightspeed | IBM watsonx |
|---|---|---|
| On-premise | RHEL/OCP (self-hosted) | IBM Cloud Pak for Data (self-hosted) |
| Híbrido | RHEL/OCP (self-hosted) | IBM Cloud (SaaS) |

## Ativar no VS Code

```
1. Instalar extensão "Ansible" (Red Hat) no VS Code
2. Settings → Ansible → Lightspeed → Enabled ✓
3. Lightspeed → URL: https://aap.empresa.org  (on-premise)
             ou URL: https://c.ai.ansible.redhat.com (cloud)
4. Ctrl+Shift+P → "Ansible: Connect to Lightspeed"
5. Login com conta Red Hat → Autorizar
```

## Funcionalidades

| Feature | Descrição |
|---|---|
| **Single-task recommendation** | Sugestão para uma task pelo nome |
| **Multi-task recommendation** | Bloco de tasks a partir de prompt |
| **Playbook generation** | Playbook completo a partir de descrição |
| **Playbook explanation** | Explicar o que um playbook faz |
| **Ansible Code Bot** | Bot GitHub/GitLab que abre PRs automáticos |

## Configurar no AAP (Inventory — Containerizado)

```ini
# Variáveis do Lightspeed Coding Assistant
[all:vars]
lightspeed_enabled=true
lightspeed_controller_url=https://c.ai.ansible.redhat.com   # cloud service
# OU para on-premise com watsonx:
# lightspeed_model_url=https://watsonx.empresa.org/api
# lightspeed_api_key=<api_key>
# lightspeed_model_id=<model_id>
```

## Trial Gratuito (90 dias)

```
Requisito: subscrição AAP válida
→ Acessa: https://console.redhat.com/ansible/lightspeed
→ Recursos trial: single-task, multi-task, playbook generation, explanation
→ Após 90 dias: configurar on-premise (watsonx) ou pagar IBM watsonx SaaS
```
