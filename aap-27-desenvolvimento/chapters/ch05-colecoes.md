# Ch05 — Criação e Publicação de Coleções

## Scaffold de Coleção

```bash
# Criar estrutura de coleção
ansible-creator init collection empresa.infra

# Estrutura gerada:
empresa/infra/
├── galaxy.yml          # Metadados da coleção
├── README.md
├── roles/
│   └── minha_role/
│       ├── tasks/main.yml
│       ├── defaults/main.yml
│       └── meta/main.yml
├── plugins/
│   ├── modules/
│   └── filter/
└── tests/
```

## galaxy.yml — Metadados

```yaml
namespace: empresa
name: infra
version: 1.2.0
readme: README.md
description: Coleção de roles de infraestrutura da empresa
license:
  - GPL-2.0-or-later
authors:
  - Time de Automação <automacao@empresa.org>
repository: https://github.com/empresa/ansible-collection-infra
dependencies:
  ansible.posix: ">=1.5.0"
  community.general: ">=7.0.0"
```

## Publicar no Private Automation Hub

```bash
# 1. Construir o tarball da coleção
ansible-galaxy collection build empresa/infra/

# 2. Publicar no Hub privado
ansible-galaxy collection publish empresa-infra-1.2.0.tar.gz \
  --server-url https://aap.empresa.org/api/galaxy/ \
  --api-key <token_do_hub>
```

```ini
# ansible.cfg para usar o Hub privado por padrão
[galaxy]
server_list = meu_hub, automation_hub

[galaxy_server.meu_hub]
url=https://aap.empresa.org/api/galaxy/
auth_url=https://aap.empresa.org/api/gateway/v1/tokens/
token=<token>

[galaxy_server.automation_hub]
url=https://console.redhat.com/api/automation-hub/
auth_url=https://sso.redhat.com/auth/realms/redhat-external/protocol/openid-connect/token
token=<offline_token>
```

## requirements.yml — Instalar Coleções no EE

```yaml
---
collections:
  - name: empresa.infra
    version: ">=1.2.0"
    source: https://aap.empresa.org/api/galaxy/

  - name: ansible.posix
    version: "1.5.4"

  - name: redhat.satellite
    version: "3.15.0"
    source: https://console.redhat.com/api/automation-hub/

roles:
  - name: minha.role
    src: https://github.com/empresa/ansible-role-nginx
    version: v2.1.0
```

## Estrutura Recomendada de Role

```
roles/patching/
├── tasks/
│   ├── main.yml           # Entrada: inclui outros arquivos de tasks
│   ├── pre_checks.yml
│   ├── apply_patches.yml
│   └── post_checks.yml
├── defaults/main.yml      # Variáveis com valores padrão
├── vars/main.yml          # Variáveis internas (não override)
├── handlers/main.yml      # Handlers
├── templates/             # Templates Jinja2
├── files/                 # Arquivos estáticos
├── meta/main.yml          # Dependências de roles
│   # galaxy_info.role_name, author, min_ansible_version, platforms
└── README.md
```
