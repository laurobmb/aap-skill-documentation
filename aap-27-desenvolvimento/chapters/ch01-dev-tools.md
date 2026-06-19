# Ch01 — ansible-dev-tools e Workflow de Desenvolvimento

## ansible-dev-tools — Suite Completa

O `ansible-dev-tools` agrupa todas as ferramentas de desenvolvimento Ansible em um único pacote:

| Ferramenta | Função |
|---|---|
| `ansible-creator` | Scaffold de projetos (playbook, collection, role) |
| `ansible-navigator` | CLI/TUI para executar playbooks em EEs |
| `ansible-lint` | Linter de playbooks e roles |
| `molecule` | Framework de testes de roles |
| `ansible-builder` | Construir Execution Environments |
| `tox-ansible` | Testes automatizados com tox |

```bash
# Instalar (requer Python 3.10+)
pip install ansible-dev-tools

# Verificar
ansible-creator --version
ansible-navigator --version
```

## ansible-creator — Scaffold de Projetos

```bash
# Criar projeto de playbook
ansible-creator init playbook meu-projeto --no-ansi

# Estrutura gerada:
meu-projeto/
├── site.yml
├── playbooks/
│   └── main.yml
├── roles/
├── inventory/
│   └── hosts.yml
├── group_vars/
├── host_vars/
├── collections/
│   └── requirements.yml
└── ansible-navigator.yml

# Criar coleção
ansible-creator init collection minha-org.minha-colecao
```

## ansible-navigator — Executar em EEs

```bash
# Rodar playbook no EE padrão (modo TUI)
ansible-navigator run site.yml -i inventory/

# Modo stdout (para CI/CD)
ansible-navigator run site.yml -i inventory/ -m stdout

# EE específico
ansible-navigator run site.yml \
  --eei registry.redhat.io/ansible-automation-platform-27/ee-supported-rhel9:latest \
  -m stdout

# Replay de execução anterior
ansible-navigator replay /tmp/artifacts/run.json

# Listar EEs disponíveis
ansible-navigator images

# Explorar coleções no EE
ansible-navigator collections

# Documentação de módulo
ansible-navigator doc ansible.builtin.template
```

## ansible-navigator.yml — Configuração por Projeto

```yaml
---
ansible-navigator:
  execution-environment:
    image: registry.redhat.io/ansible-automation-platform-27/ee-supported-rhel9:latest
    pull:
      policy: missing    # never | missing | tag | always
  mode: stdout           # stdout | interactive (TUI)
  playbook-artifact:
    enable: true
    save-as: /tmp/artifacts/{playbook_name}-{ts_utc}.json
  ansible:
    inventories:
      - ./inventory
  logging:
    level: warning       # debug | info | warning | error | critical
```

## Workflow Completo de Desenvolvimento

```
1. scaffold         → ansible-creator init playbook meu-projeto
2. desenvolver      → escrever playbooks + roles no VS Code (com extensão Ansible)
3. lint             → ansible-lint site.yml
4. testar local     → ansible-navigator run site.yml -m stdout
5. testes role      → molecule test
6. commit           → git commit + push
7. CI/CD            → pipeline valida + aplica no AAP (via CaC ou webhook)
8. revisar output   → ansible-navigator replay /tmp/artifacts/run.json
```
