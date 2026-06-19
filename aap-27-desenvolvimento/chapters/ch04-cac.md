# Ch04 — Configuration as Code (CaC) com ansible.platform

## Collection Oficial: ansible.platform

A collection unificada para CaC no AAP 2.7 (substitui `awx.awx` e `redhat_cop.controller_configuration`):

```bash
ansible-galaxy collection install ansible.platform
```

## Módulos Principais

```
ansible.platform.organization          # Organizações
ansible.platform.team                  # Times
ansible.platform.user                  # Usuários
ansible.platform.credential            # Credenciais
ansible.platform.credential_type       # Tipos de credencial customizados
ansible.platform.inventory             # Inventários
ansible.platform.host                  # Hosts em inventários
ansible.platform.project               # Projetos (repos Git)
ansible.platform.job_template          # Job Templates
ansible.platform.workflow_job_template # Workflow Job Templates
ansible.platform.schedule              # Agendamentos
ansible.platform.execution_environment # Registrar EEs
ansible.platform.role_team_assignment  # Atribuir role a time
ansible.platform.role_user_assignment  # Atribuir role a usuário
```

## Playbook CaC Básico

```yaml
---
- name: Configurar organização no AAP
  hosts: localhost
  connection: local
  gather_facts: false

  vars:
    aap_hostname: "https://aap.empresa.org"
    aap_username: "admin"
    aap_password: "{{ vault_aap_password }}"
    aap_validate_certs: true

  tasks:
    - name: Criar organização
      ansible.platform.organization:
        name: "Financeiro"
        description: "Equipe financeira"
        max_hosts: 200
        controller_host: "{{ aap_hostname }}"
        controller_username: "{{ aap_username }}"
        controller_password: "{{ aap_password }}"

    - name: Criar credencial de SCM
      ansible.platform.credential:
        name: "github-financeiro"
        organization: "Financeiro"
        credential_type: "Source Control"
        inputs:
          username: "svc-git"
          ssh_key_data: "{{ vault_ssh_key }}"
        controller_host: "{{ aap_hostname }}"
        controller_username: "{{ aap_username }}"
        controller_password: "{{ aap_password }}"
        no_log: true

    - name: Criar projeto
      ansible.platform.project:
        name: "automacoes-financeiro"
        organization: "Financeiro"
        scm_type: git
        scm_url: "https://github.com/empresa/automacoes-financeiro"
        scm_credential: "github-financeiro"
        scm_update_on_launch: true
        controller_host: "{{ aap_hostname }}"
        controller_username: "{{ aap_username }}"
        controller_password: "{{ aap_password }}"

    - name: Criar Job Template
      ansible.platform.job_template:
        name: "Patching RHEL - Financeiro"
        organization: "Financeiro"
        project: "automacoes-financeiro"
        playbook: "playbooks/patch.yml"
        inventory: "inventario-linux-financeiro"
        execution_environment: "ee-supported-rhel9:latest"
        credentials:
          - "satellite-prod"
        ask_variables_on_launch: true
        controller_host: "{{ aap_hostname }}"
        controller_username: "{{ aap_username }}"
        controller_password: "{{ aap_password }}"
```

## Estrutura Multi-Org (Polyrepo)

```
cac-platform-core/
├── playbooks/apply_platform.yml    # Roda como admin global
└── vars/
    ├── organizations.yml
    ├── credential_types.yml
    └── execution_environments.yml

cac-org-financeiro/
├── playbooks/configure_org.yml     # Roda como svc-cac-financeiro (escopo limitado)
└── vars/
    ├── projects.yml
    ├── job_templates.yml
    ├── inventories.yml
    └── credentials.yml             # Somente credenciais não-privilegiadas
```

## Pipeline GitOps (GitLab CI)

```yaml
# .gitlab-ci.yml no cac-org-financeiro
stages:
  - validate
  - apply

validate:
  stage: validate
  script:
    - ansible-lint vars/*.yml
    - ansible-playbook playbooks/configure_org.yml --check \
        --vault-password-file <(echo "$VAULT_PASSWORD")

apply:
  stage: apply
  script:
    - ansible-playbook playbooks/configure_org.yml \
        --vault-password-file <(echo "$VAULT_PASSWORD")
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
      when: manual    # Aprovação manual para produção
```

## Usar Conta de Serviço por Org

```yaml
# vars de conexão usando conta de serviço com escopo limitado
aap_hostname: "https://aap.empresa.org"
aap_username: "svc-cac-financeiro"    # Acesso apenas à org Financeiro
aap_token: "{{ vault_svc_token }}"    # PAT do Gateway
```
