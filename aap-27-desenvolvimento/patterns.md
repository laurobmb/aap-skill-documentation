# Patterns — AAP 2.7 Desenvolvimento

## Padrão 1: Workflow Completo de Desenvolvimento com EE Customizado

```
1. scaffold projeto:
   ansible-creator init playbook meu-projeto

2. criar EE definition:
   execution-environment.yml com collections + deps necessárias

3. build EE:
   ansible-builder build --tag registry.empresa.org/aap/meu-ee:latest

4. testar local:
   ansible-navigator run site.yml --eei meu-ee:latest -m stdout

5. publicar EE no Hub:
   podman push registry.empresa.org/aap/meu-ee:latest

6. publicar projeto no Git:
   git push origin feature/nova-automacao

7. no AAP: sync projeto → criar JT com o EE → testar
```

## Padrão 2: EDA integrado com GLPI via Webhook

```
Cenário: GLPI abre chamado → AAP cria VM automaticamente

1. Configurar rulebook:
   source: ansible.eda.webhook (porta 5000)
   rule: condition: event.payload.categoria == "nova_vm"
   action: run_job_template: "Criar VM vCenter"

2. Configurar GLPI:
   Webhook → URL do EDA → POST ao abrir chamado de nova VM

3. Rulebook Activation no AAP:
   Ativar rulebook + credenciais do Controller
   Decision Environment: de-supported-rhel9

4. Testar: abrir chamado no GLPI → ver job disparado no Controller
```

## Padrão 3: MCP em EE para Automação Controlada por AI

```
Cenário: AI agent (Cursor) cria repo GitHub via playbook AAP

1. Build EE com GitHub MCP:
   execution-environment.yml com mcp_servers=github_mcp

2. Criar Custom Credential Type "MCP Bearer Token"
   injector: env.MCP_BEARER_TOKEN = "{{ token }}"

3. Job Template:
   EE: meu-ee-com-mcp
   Credenciais: github-mcp-token
   Playbook: github-mcp-demo.yml

4. Inventory: host github_mcp com ansible_connection=ansible.mcp.mcp

5. Disparar job → MCP server criado efêmeramente → 
   playbook chama GitHub → repo criado → MCP destruído
```

## Padrão 4: CaC GitOps com Aprovação Manual para Produção

```
Branch: feature/* → dev     (auto-apply)
Branch: main      → stage   (auto-apply)
Tag: v*.*.* / PR aprovado → prod (manual-approve)

Pipeline:
  validate: ansible-lint + check mode
  apply-dev: sempre que merge na branch dev
  apply-stage: sempre que merge na main
  apply-prod: gatilho manual após aprovação
```
