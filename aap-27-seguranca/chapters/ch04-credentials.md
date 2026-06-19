# Ch04 — Credentials e External Secret Providers

## Tipos de Credenciais Principais

| Tipo | Uso |
|---|---|
| Machine | SSH para hosts gerenciados (chave privada, sudo password) |
| Source Control | Git (username/token/SSH key para projetos) |
| Vault | Ansible Vault password para decriptar variáveis |
| Amazon Web Services | AWS access key + secret |
| VMware vCenter | host, username, password do vCenter |
| Red Hat Satellite | URL e credenciais do Satellite |
| OpenShift / Kubernetes | Kubeconfig ou service account token |
| GitHub Personal Access Token | Token PAT do GitHub |
| GitLab Personal Access Token | Token PAT do GitLab |
| HashiCorp Vault Secret Lookup | Token ou AppRole para buscar segredos no Vault |
| HashiCorp Vault Signed SSH | Assinar chaves SSH via Vault |
| CyberArk AIM | Buscar segredos no CyberArk AIM |
| Azure Key Vault | Buscar segredos no Azure Key Vault |
| Centrify Vault | Buscar segredos no Centrify |
| Custom (credential type) | Qualquer integração customizada |

## Criar Credencial

```
Gateway → Credentials → Create Credential
  → Tipo → Preencher campos
  → Organization (quem pode gerenciar)
```

```yaml
# Via CaC
ansible.platform.credential:
  name: "vcenter-prod"
  organization: "Platform"
  credential_type: "VMware vCenter"
  inputs:
    host: "vcenter.empresa.org"
    username: "svc-ansible"
    password: "{{ vcenter_password | encrypt_vault }}"
```

## Credential Types Customizados

Criar tipo de credencial quando nenhum built-in atende:

```yaml
# Exemplo: credencial para API interna da empresa
ansible.platform.credential_type:
  name: "API Interna Empresa"
  kind: cloud
  inputs:
    fields:
      - id: api_url
        type: string
        label: URL da API
      - id: api_token
        type: string
        label: Token da API
        secret: true
  injectors:
    env:
      EMPRESA_API_URL: "{{ api_url }}"
      EMPRESA_API_TOKEN: "{{ api_token }}"
    # ou via extra_vars:
    extra_vars:
      empresa_api_url: "{{ api_url }}"
```

## External Secret Providers

### HashiCorp Vault — Secret Lookup

```yaml
# Credencial do tipo Vault (para lookup)
ansible.platform.credential:
  name: "vault-lookup"
  organization: "Platform"
  credential_type: "HashiCorp Vault Secret Lookup"
  inputs:
    url: "https://vault.empresa.org"
    role_id: "{{ vault_role_id }}"
    secret_id: "{{ vault_secret_id }}"
    auth_type: "approle"
```

```yaml
# Credencial que usa o Vault para buscar o valor
ansible.platform.credential:
  name: "vcenter-prod"
  organization: "Platform"
  credential_type: "VMware vCenter"
  inputs:
    host: "vcenter.empresa.org"
    username: "svc-ansible"
    # password vem do Vault em runtime:
  credential_inputs_source:
    - input_field_name: password
      source_credential: "vault-lookup"
      metadata:
        secret_path: "secret/vcenter/prod"
        secret_key: "password"
```

### Azure Key Vault

```yaml
ansible.platform.credential:
  name: "azure-vault"
  credential_type: "Microsoft Azure Key Vault"
  inputs:
    url: "https://meukeyvault.vault.azure.net"
    client_id: "{{ azure_client_id }}"
    client_secret: "{{ azure_client_secret }}"
    tenant_id: "{{ azure_tenant_id }}"
```

## Boas Práticas de Segurança

```
1. Nunca armazenar credenciais em playbooks ou vars em texto claro
2. Usar organization para isolar credenciais por time
3. Compartilhar credenciais privilegiadas via role "Use" (não transferir ownership)
4. External Secret Provider: segredo nunca entra no Git
5. Usar no_log: true em tasks com credenciais sensíveis
6. Rotacionar tokens/senhas periodicamente (PATs: validade de 1 ano padrão no 2.7)
```
