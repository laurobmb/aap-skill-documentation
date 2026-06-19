# Ch04 — Integrações: Terraform e HashiCorp Vault

## Terraform + AAP

### AAP como backend de execução Terraform

```hcl
# main.tf — Disparar Job Template AAP via Terraform provider
terraform {
  required_providers {
    aap = {
      source  = "ansible/aap"
      version = "~> 1.0"
    }
  }
}

provider "aap" {
  host     = "https://aap.empresa.org"
  username = "terraform-svc"
  password = var.aap_password
}

resource "aap_job" "provisionar_vm" {
  job_template_id = 42
  extra_vars = jsonencode({
    vm_name   = var.vm_name
    vm_ram_gb = var.vm_ram
    vm_cpu    = var.vm_cpu
  })
  wait_for_completion = true
}
```

### Fluxo GitOps Terraform → AAP

```
Pull Request aprovado
        ↓
  Terraform plan (CI)
        ↓
  terraform apply
        ↓
  AAP Terraform Provider → dispara Job Template
        ↓
  Ansible executa provisionamento real
        ↓
  Output retorna para Terraform (output variables)
```

## HashiCorp Vault como External Secret Provider

```yaml
# Criar credencial Vault no AAP (via CaC)
ansible.platform.credential:
  name: "vault-approle"
  organization: "Platform"
  credential_type: "HashiCorp Vault Secret Lookup"
  inputs:
    url: "https://vault.empresa.org"
    auth_type: "approle"
    role_id: "{{ vault_role_id }}"
    secret_id: "{{ vault_secret_id }}"
    api_version: "v2"     # KV v2
```

```yaml
# Credencial que busca senha do Vault em runtime
ansible.platform.credential:
  name: "vcenter-prod"
  organization: "Platform"
  credential_type: "VMware vCenter"
  inputs:
    host: "vcenter.empresa.org"
    username: "svc-ansible"
  credential_inputs_source:
    - input_field_name: password
      source_credential: "vault-approle"
      metadata:
        secret_path: "secret/data/vcenter/prod"
        secret_key: "password"
```

## Vault Signed SSH (Zero-Trust SSH)

```yaml
# Credencial para assinar chaves SSH via Vault
ansible.platform.credential:
  name: "vault-ssh-signer"
  organization: "Platform"
  credential_type: "HashiCorp Vault Signed SSH"
  inputs:
    url: "https://vault.empresa.org"
    token: "{{ vault_token }}"
    ssh_role: "ansible-ssh-role"
    public_key: "{{ ssh_public_key }}"
```

## Custom Credential Types para Integrações Específicas

```yaml
# ServiceNow
ansible.platform.credential_type:
  name: "ServiceNow API"
  kind: cloud
  inputs:
    fields:
      - id: instance_url
        type: string
        label: ServiceNow Instance URL
      - id: api_user
        type: string
        label: API User
      - id: api_password
        type: string
        label: API Password
        secret: true
  injectors:
    env:
      SN_HOST: "{{ instance_url }}"
      SN_USERNAME: "{{ api_user }}"
      SN_PASSWORD: "{{ api_password }}"
```
