# Patterns — AAP 2.7 EDA Operação

## Pattern 1: Event Stream GitHub → Criar VM automaticamente

```
Cenário: Abertura de issue no GitHub com label "vm-request" dispara criação de VM no vCenter.
```

```
GitHub Issue (label: vm-request)
  → Event Stream (GitHub HMAC)
  → Rulebook Activation
  → run_job_template: "Criar VM vCenter"
```

```yaml
# Credencial:
Name: cred-github-vms
Type: GitHub Event Stream
HMAC Secret: <secret-configurado-no-github>

# Event Stream:
Name: stream-github-vm-requests
Type: GitHub Event Stream
Credential: cred-github-vms
Headers: X-GitHub-Event,X-Hub-Signature-256
Forward events: ☑

# Rulebook:
- name: Processar requisição de VM via GitHub
  hosts: all
  sources:
    - eda.builtin.webhook:
        host: 0.0.0.0
        port: 5000
  rules:
    - name: Criar VM quando issue criada com label vm-request
      condition: >
        event.payload.action == "labeled" and
        event.payload.label.name == "vm-request"
      action:
        run_job_template:
          name: "Criar VM vCenter"
          organization: "Infraestrutura"
          job_args:
            extra_vars:
              issue_title: "{{ event.payload.issue.title }}"
              requester: "{{ event.payload.sender.login }}"
```

## Pattern 2: GLPI Webhook → EDA no OCP

```
Cenário: Chamado aberto no GLPI dispara automação via EDA rodando em OCP.
```

```bash
# 1. Criar Event Stream do tipo Token (ServiceNow type adapts well)
#    ou usar HMAC com secret compartilhado com GLPI

# 2. Criar Rulebook Activation com porta 5000 exposta
#    Service name: eda-glpi-svc

# 3. Criar Route no OCP:
oc apply -f - <<'EOF'
kind: Route
apiVersion: route.openshift.io/v1
metadata:
  name: eda-glpi
  namespace: aap
spec:
  host: eda-glpi.apps.empresa.org
  to:
    kind: Service
    name: activation-job-<id>-5000
  port:
    targetPort: 5000
  tls:
    termination: edge
EOF

# 4. Configurar webhook no GLPI:
#    URL: https://eda-glpi.apps.empresa.org
#    Method: POST
#    Secret: <mesmo secret do event stream>
```

## Pattern 3: Multi-Event Stream para Alta Disponibilidade

```
Cenário: Múltiplos sistemas enviam eventos para a mesma automation. 
         Usar event streams separados por sistema, activation única.
```

```
[GitHub Event Stream]  ──┐
[GLPI Event Stream]    ──┤── Activation: "Criar VM Unificado"
[ServiceNow Stream]    ──┘         │
                                   └── run_job_template: "Criar VM"
```

```yaml
# Rulebook com múltiplos sources via event streams:
- name: Processar requisições de VM de qualquer fonte
  hosts: all
  sources:
    # Sources substituídos por event streams no momento da criação da activation
    - eda.builtin.webhook:
        host: 0.0.0.0
        port: 5000
    - eda.builtin.webhook:
        host: 0.0.0.0
        port: 5001
  rules:
    - name: Criar VM
      condition: event.payload.request_type == "create_vm"
      action:
        run_job_template:
          name: "Criar VM vCenter"
          organization: "Infraestrutura"
```
