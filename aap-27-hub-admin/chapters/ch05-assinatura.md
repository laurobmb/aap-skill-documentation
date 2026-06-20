# Ch05 — Assinatura de Conteúdo com GPG

## Por que Assinar Coleções?

Assinatura GPG garante que:
- As coleções distribuídas pelo Hub não foram alteradas após publicação
- Os usuários podem verificar a autenticidade antes de instalar
- Coleções rejeitadas/não assinadas podem ser bloqueadas na instalação

## Configurar Hub para Assinatura

**Pré-requisitos:**
- Par de chaves GPG criado e gerenciado de forma segura pela organização
- Script de assinatura que aceita apenas um filename como argumento

**1. Criar script de assinatura:**

```bash
#!/usr/bin/env bash
# /usr/local/bin/collection_signing.sh

FILE_PATH=$1
SIGNATURE_PATH="$1.asc"
ADMIN_ID="$PULP_SIGNING_KEY_FINGERPRINT"
PASSWORD="senha-da-chave-gpg"

gpg --quiet --batch --pinentry-mode loopback --yes \
    --passphrase $PASSWORD \
    --homedir ~/.gnupg/ \
    --detach-sign --default-key $ADMIN_ID \
    --armor --output $SIGNATURE_PATH $FILE_PATH

STATUS=$?
if [ $STATUS -eq 0 ]; then
  echo "{\"file\": \"$FILE_PATH\", \"signature\": \"$SIGNATURE_PATH\"}"
else
  exit $STATUS
fi
```

**2. Configurar variáveis no inventory do installer:**

```ini
[all:vars]
automationhub_create_default_collection_signing_service = True
automationhub_auto_sign_collections = True
automationhub_require_content_approval = True
automationhub_collection_signing_service_key = /abs/path/to/galaxy_signing_service.gpg
automationhub_collection_signing_service_script = /abs/path/to/collection_signing.sh
```

> Após deploy com assinatura habilitada, novas funcionalidades de UI de assinatura aparecem.

## Download da Chave Pública

```
Automation Content → Signature Keys

Tipos de chaves:
  collections-*  → verificar coleções
  container-*    → verificar imagens de containers

→ Download Key  ou  Copy to clipboard
```

## Configurar ansible-galaxy CLI para Verificar Assinaturas

```bash
# 1. Importar chave pública para keyring local
gpg --import --no-default-keyring \
    --keyring ~/.ansible/pubring.kbx \
    my-public-key.asc

# 2. Instalar coleção com verificação de assinatura
ansible-galaxy collection install namespace.collection \
  --signature https://hub.empresa.org/detached_signature.asc \
  --keyring ~/.ansible/pubring.kbx

# 3. Via requirements.yml com assinaturas
collections:
  - name: empresa.infra
    version: 1.0.0
    signatures:
      - https://hub.empresa.org/assinaturas/empresa_infra_1.0.0.asc

ansible-galaxy collection verify -r requirements.yml --keyring ~/.ansible/pubring.kbx

# 4. Verificação offline (sem consultar o servidor)
ansible-galaxy collection verify -r requirements.yml \
  --keyring ~/.ansible/pubring.kbx \
  --offline
```

## Forçar Apenas Coleções Assinadas (Remote Config)

```
Automation Content → Remotes → [remoto] → Edit remote
  → ☑ Signed collections only

# Somente coleções com assinatura válida serão sincronizadas desse remoto.
```
