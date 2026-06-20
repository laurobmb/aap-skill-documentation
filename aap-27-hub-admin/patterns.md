# Patterns — AAP 2.7 Hub Admin

## Pattern 1: Hub Multi-Org — Namespace por Time com Fluxo de Aprovação

```
Cenário: 30 organizações, cada time publica suas coleções internas com revisão obrigatória.
```

```
Estrutura de namespaces:
  empresa_infra      → time Infraestrutura
  empresa_network    → time Redes
  empresa_windows    → time Windows
  empresa_security   → time Segurança
  empresa_platform   → time Plataforma (coleções certificadas Red Hat)

Estrutura de repositórios:
  staging-interno    → pipeline=staging (recebe uploads dos times)
  published-interno  → pipeline=approved (aprovado pelo time de plataforma)
  rh-certified       → pipeline=approved (coleções Red Hat sync de console.redhat.com)
```

```
# Permissões por time (exemplo para time de Infraestrutura):
Namespace empresa_infra:
  Time "Infra Developers" → Namespace Publisher (upload)

Repositório published-interno:
  Time "Platform Admins" → Manage Repository (aprovação e gestão)
```

## Pattern 2: Sync Red Hat Certified + Filtro por Requirements

```
Cenário: Sincronizar apenas as coleções necessárias do Red Hat, não o catálogo completo.
```

```yaml
# requirements.yml para o remote rh-certified
collections:
  # Network
  - name: ansible.netcommon
    version: ">=5.0.0"
  - name: cisco.ios
  - name: arista.eos

  # Satellite / RHEL
  - name: redhat.satellite
    version: ">=3.2.0"
  - name: ansible.posix
  - name: redhat.rhel_system_roles

  # VMware
  - name: community.vmware
    version: ">=4.0.0"

  # Windows
  - name: ansible.windows
  - name: community.windows

  # Cloud
  - name: amazon.aws
    version: ">=7.0.0"
```

```
Automation Content → Remotes → rh-certified → Edit remote
  → YAML requirements: colar conteúdo acima
  → Save → Sync (via repositório)
```

## Pattern 3: EE Customizada — Build, Push e Gerenciar no Hub

```
Cenário: EE customizada com coleções internas + certificadas + ferramentas da empresa.
```

```yaml
# ee-infra.yml (EE definition para ansible-builder)
version: 3
build_arg_defaults:
  ANSIBLE_GALAXY_SERVER_LIST: "my_hub"

images:
  base_image:
    name: hub.empresa.org/ee-supported-rhel9:2.7  # base do Hub interno

dependencies:
  galaxy:
    collections:
      - name: redhat.satellite
      - name: empresa_infra.linux   # coleção interna
  python:
    - pyaml>=6.0
  system:
    - git

additional_build_steps:
  append_final:
    - RUN pip install netmiko
```

```bash
# Build e push para o Hub
ansible-builder build -t hub.empresa.org/ee-infra-custom:1.0 -f ee-infra.yml
podman push hub.empresa.org/ee-infra-custom:1.0
```

## Pattern 4: Air-Gapped — Importar Coleções sem Acesso à Internet

```bash
# No ambiente com internet: baixar coleções
ansible-galaxy collection download redhat.satellite ansible.netcommon \
  -p /tmp/collections-bundle/

# Compactar para transferência
tar -czf collections-bundle.tar.gz /tmp/collections-bundle/

# No ambiente air-gapped: transferir e importar via Hub UI
# Automation Content → Namespaces → [namespace]
#   → Upload Collection → selecionar .tar.gz → pipeline sem aprovação
```
