# Ch03 — Event-Driven Ansible (EDA)

## Conceitos Fundamentais

```
Event Source → Rulebook → Rule → Action
     ↓                              ↓
  (webhook,                  (run_job_template,
   kafka, etc.)               set_fact, debug, etc.)
```

| Componente | Descrição |
|---|---|
| **Event Source** | Plugin que consome eventos (webhook, kafka, alertmanager, etc.) |
| **Rulebook** | Arquivo YAML com sources + rules |
| **Rule** | Condição (when) + ação (action) |
| **Rulebook Activation** | Instância em execução de um rulebook |
| **Decision Environment** | Container com dependências do EDA (equivalente ao EE no Controller) |

## Estrutura de um Rulebook

```yaml
# rulebooks/webhook-vm-create.yml
---
- name: Criar VM via GLPI webhook
  hosts: all
  sources:
    - ansible.eda.webhook:
        host: 0.0.0.0
        port: 5000
        # Token de autenticação (opcional)
        token: "{{ webhook_token }}"

  rules:
    - name: Acionar criação de VM quando chamado GLPI
      condition: event.payload.request_type == "vm_create"
      action:
        run_job_template:
          name: "Provisionar VM vCenter"
          organization: "Infraestrutura"
          job_args:
            extra_vars:
              vm_name: "{{ event.payload.vm_name }}"
              vm_ram: "{{ event.payload.ram_gb }}"
              vm_cpu: "{{ event.payload.cpu_count }}"
```

## Event Sources Suportados (Built-in)

| Source Plugin | Protocolo/Sistema |
|---|---|
| `ansible.eda.webhook` | HTTP webhook genérico |
| `ansible.eda.kafka` | Apache Kafka |
| `ansible.eda.alertmanager` | Prometheus AlertManager |
| `ansible.eda.aws_sqs_queue` | AWS SQS |
| `ansible.eda.azure_service_bus` | Azure Service Bus |
| `ansible.eda.file` | Monitorar arquivo local (dev/teste) |
| `ansible.eda.watchdog` | Monitorar diretório |
| `ansible.eda.url_check` | Polling de URLs HTTP |
| `ansible.eda.journald` | Logs systemd journald |
| `ansible.eda.range` | Gerador de eventos (dev/teste) |

## Ações Disponíveis

| Ação | Descrição |
|---|---|
| `run_job_template` | Disparar Job Template no Controller |
| `run_workflow_template` | Disparar Workflow no Controller |
| `set_fact` | Definir variável no contexto EDA |
| `post_event` | Publicar evento para outro rulebook |
| `print_event` | Log de saída (debug) |
| `shutdown` | Encerrar o rulebook activation |
| `noop` | Não fazer nada (debug) |

## Decision Environment (DE)

```yaml
# decision-environment.yml
---
version: 3

images:
  base_image:
    name: registry.redhat.io/ansible-automation-platform-27/de-supported-rhel9:latest

dependencies:
  galaxy:
    collections:
      - name: ansible.eda
      - name: community.general
```

```bash
# Construir DE
ansible-builder build \
  --tag registry.empresa.org/aap/meu-de:latest \
  --file decision-environment.yml
```

## Configurar Rulebook Activation no AAP

```
EDA Controller → Rulebook Activations → Create Activation
  Project: (repo com rulebooks)
  Rulebook: webhook-vm-create.yml
  Decision Environment: de-supported-rhel9:latest
  Credentials: (se necessário para autenticar no Controller)
  Max running rulebook activations: 5
```

## Scaling Horizontal do EDA (Novo 2.7)

```ini
# Inventory file — escalar workers EDA
[automationeda]
eda1.exemplo.org
eda2.exemplo.org   # segundo nó para HA

[all:vars]
# Número de workers por nó
eda_max_running_activations=12   # padrão: 12 por nó
```

## Webhook GitHub → Job Template (exemplo completo)

```yaml
# 1. No Controller: habilitar webhook no Job Template
#    Job Template → Enable webhook → GitHub → copiar Webhook URL + Key

# 2. Configurar webhook no GitHub repo:
#    Settings → Webhooks → Add webhook
#    Payload URL: https://aap.exemplo.org/api/controller/v2/job_templates/<id>/github/
#    Content type: application/json
#    Secret: <Webhook Key do JT>
#    Events: Push (ou customizado)

# 3. Ao receber push → Controller dispara o Job Template automaticamente
```
