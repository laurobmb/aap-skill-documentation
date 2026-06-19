# Cheatsheet — AAP 2.7 Desenvolvimento

## ansible-dev-tools

```bash
pip install ansible-dev-tools

# Scaffold
ansible-creator init playbook meu-projeto
ansible-creator init collection empresa.colecao

# Lint
ansible-lint site.yml

# Testar role
molecule test
```

## ansible-navigator Essencial

```bash
# Rodar playbook no EE
ansible-navigator run site.yml -m stdout --eei <imagem>

# TUI atalhos: :collections :images :help Esc=voltar
# Replay
ansible-navigator replay /tmp/artifacts/run.json
```

## EE com MCP Server

```yaml
additional_build_steps:
  append_final: |
    RUN ansible-playbook ansible.mcp_builder.install_mcp \
      -e mcp_servers=github_mcp -e github_mcp_mode=remote
```

```yaml
# Playbook MCP
- hosts: github_mcp
  tasks:
    - ansible.mcp.run_tool:
        name: create_repository
        args: {name: my-repo, autoInit: true}
```

## MCP Servers Suportados (2.7)

```
github_mcp | aws_core_mcp | aws_cloud_control_mcp
```

## EDA Rulebook Mínimo

```yaml
- name: Webhook trigger
  hosts: all
  sources:
    - ansible.eda.webhook:
        host: 0.0.0.0
        port: 5000
  rules:
    - name: Disparar JT
      condition: event.payload.type == "deploy"
      action:
        run_job_template:
          name: "Deploy App"
          organization: "DevTeam"
```

## CaC — Módulos ansible.platform

```
organization | team | user | credential | credential_type
inventory | host | project | job_template | workflow_job_template
schedule | execution_environment | role_team_assignment | role_user_assignment
```

## Imagens Base 2.7

```
ee-minimal-rhel9:latest    | ee-supported-rhel9:latest
ee-minimal-rhel10:latest   | ee-supported-rhel10:latest
de-supported-rhel9:latest  (Decision Environment EDA)
```
