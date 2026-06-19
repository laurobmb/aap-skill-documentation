# Ch02 — Execution Environments e MCP Server Integration

## EE Definition File v3 (ansible-builder 3.x)

```yaml
# execution-environment.yml
---
version: 3

images:
  base_image:
    name: registry.redhat.io/ansible-automation-platform-27/ee-minimal-rhel9:latest

dependencies:
  galaxy:
    collections:
      - name: ansible.posix
      - name: community.general
  python:
    - boto3
    - botocore
  system:
    - git [platform:rpm]

options:
  package_manager_path: /usr/bin/microdnf

additional_build_steps:
  prepend_galaxy: |
    RUN $PKGMGR install -y python3-pip
  append_final: |
    RUN pip3 install my-extra-lib
```

```bash
# Construir EE
ansible-builder build \
  --tag registry.empresa.org/aap/meu-ee:latest \
  --file execution-environment.yml \
  --context /tmp/ee-context

# Publicar no Hub
podman push registry.empresa.org/aap/meu-ee:latest
```

## EE com MCP Server (Novo no 2.7)

MCP servers integram AI agents (LLMs) ao ambiente de execução Ansible. São provisionados **efêmeramente** — existem apenas durante a execução do job.

**Servers suportados:**
| MCP Server | Caso de uso |
|---|---|
| GitHub MCP | Operações em repos GitHub (criar repo, PR, etc.) |
| AWS Core MCP | Ponto de entrada para MCPs AWS |
| AWS Cloud Control API MCP | Gerenciamento de recursos AWS |

### EE Definition com GitHub MCP (modo remote)

```yaml
---
version: 3

images:
  base_image:
    name: ansible-automation-platform-26/ee-minimal-rhel9:latest

dependencies:
  galaxy:
    requirements.yml

options:
  package_manager_path: /usr/bin/microdnf

additional_build_steps:
  append_final: |
    RUN ansible-playbook ansible.mcp_builder.install_mcp \
      -e mcp_servers=github_mcp \
      -e github_mcp_mode=remote
```

### Playbook Usando MCP (chamar GitHub como ferramenta)

```yaml
# playbooks/github-mcp-demo.yml
---
- name: Exemplo GitHub MCP server interaction
  hosts: github_mcp
  tasks:
    - name: Criar repositório no GitHub
      ansible.mcp.run_tool:
        name: create_repository
        args:
          name: my-github-repository
          autoInit: true
```

### Inventory para MCP

```yaml
# inventory/hosts.yml
all:
  hosts:
    github_mcp:
      ansible_connection: ansible.mcp.mcp
      ansible_connect_timeout: 30
      ansible_command_timeout: 30
      ansible_mcp_server_name: github_mcp   # nome no mcpserver.json manifest
```

### Credential Type para MCP Token

```yaml
# Criar credential type customizado para MCP Bearer Token
controller_credential_types:
  - name: "MCP Bearer Token"
    kind: cloud
    inputs:
      fields:
        - id: token
          type: string
          label: Bearer Token
          secret: true
    injectors:
      env:
        MCP_BEARER_TOKEN: "{{ token }}"
```

**Como funciona:**
1. EE é construído com MCP server instalado
2. Job Template usa o EE + credencial do MCP
3. Controller inicia o job → provisiona MCP server efêmero
4. Playbook usa `ansible.mcp.run_tool` para chamar ferramentas do MCP
5. Job termina → MCP server é destruído (sem resíduos)

**Auditoria:** todas as chamadas MCP são logadas (job ID, usuário, timestamp). Segredos nunca aparecem nos logs.

## Imagens Base Disponíveis (AAP 2.7)

```
ee-minimal-rhel9:latest     → Base mínima RHEL 9
ee-supported-rhel9:latest   → Coleções Red Hat suportadas
ee-minimal-rhel10:latest    → Base mínima RHEL 10 (novo no 2.7)
ee-supported-rhel10:latest  → Coleções Red Hat suportadas RHEL 10
de-supported-rhel9:latest   → Decision Environment para EDA
```
