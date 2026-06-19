# Ch02 — Automation Intelligent Assistant e BYOK

## Intelligent Assistant vs Ansible MCP Server

| | Intelligent Assistant (Chatbot) | Ansible MCP Server |
|---|---|---|
| **Onde fica** | Embutido na UI do AAP | Serviço separado (porta 8448) |
| **Usuário** | Usuários internos do AAP | AI agents externos (Cursor, Claude) |
| **LLM** | Configurado pelo admin | Interpretação do AI agent externo |
| **MCP** | Opcional: busca dados em tempo real | É o próprio servidor MCP |
| **Deploy** | Requer LLM (RHEL AI, OCP AI, OpenAI) | Não requer LLM externo |

## Deploy Intelligent Assistant no OCP

### Passo 1: Criar Secret de Configuração

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: chatbot-configuration-secret
  namespace: aap
stringData:
  chatbot_llm_provider_type: openai         # rhelai_vllm | rhoai_vllm | openai | azure_openai
  chatbot_url: https://api.openai.com/v1
  chatbot_model: gpt-4o-mini
  chatbot_token: <api_key>

  # Opcional: habilitar MCP para dados em tempo real do AAP
  aap_gateway_url: http://myaap
  aap_controller_url: http://myaap-controller-service
```

### Passo 2: Habilitar no Operator

```yaml
spec:
  lightspeed:
    disabled: false
    chatbot_config_secret_name: chatbot-configuration-secret
```

## Deploy Intelligent Assistant no Containerizado

```ini
# inventory file
[all:vars]
lightspeed_enabled=true
lightspeed_chatbot_url=https://api.openai.com/v1
lightspeed_chatbot_model=gpt-4o-mini
lightspeed_chatbot_token=<api_key>
lightspeed_chatbot_llm_provider_type=openai

# MCP server (opcional, Tech Preview no containerizado)
lightspeed_aap_gateway_url=https://aap.empresa.org
lightspeed_aap_controller_url=https://aap.empresa.org
```

## BYOK — Bring Your Own Knowledge (Tech Preview)

Extensão do chatbot com documentação interna da organização via RAG (Retrieval-Augmented Generation).

**Pipeline de conhecimento:**
```
BYOK docs (prioridade alta) → RAG pipeline
                                    ↓ chatbot responde com conteúdo interno
Docs padrão AAP (fallback)  → RAG pipeline
```

### Workflow BYOK

```
1. Preparar documentos (Markdown .md ou texto .txt)
2. Gerar vector database com rag-content tool
3. Construir imagem OCI com o vector database
4. Publicar imagem no registry
5. Referenciar imagem no AAP (inventory ou CR)
```

### Preparar Documentos

```bash
mkdir -p knowledge-sources/minha-org
mkdir -p output/vector_db

# Formatos suportados: .md e .txt
# Para converter PDF: usar docling
# Para converter HTML/AsciiDoc: usar Pandoc

# Boas práticas:
# - Nomear arquivos descritivamente
# - Organizar em subdiretórios por tema
# - Remover senhas/secrets antes de incluir
# - Volume ideal: 1 MB a 999 MB (max: 2 GB)
```

### Gerar Vector Database

```bash
# 1. Baixar ferramenta
podman pull registry.redhat.io/lightspeed-core/rag-tool-rhel9:v0.5-latest

# 2. Gerar embeddings e empacotar como imagem
podman run --rm \
  -u 0 \
  -v "$(pwd)/knowledge-sources/minha-org:/input:Z" \
  -v "$(pwd)/output:/output:Z" \
  registry.redhat.io/lightspeed-core/rag-tool-rhel9:v0.5-latest \
  python /rag-content/scripts/generate_embeddings.py \
    -f /input \
    -o /output/vector_db \
    -i minha-org-index \
    --exclude-model \
    --output-image /output/rag-content-output-latest.tar \
    --image-name rag-content-output \
    --image-tag latest

# 3. Publicar no registry
podman push minha-org/rag-content-output:latest registry.empresa.org/aap/byok:latest
```

### Referenciar BYOK no Containerizado

```ini
# inventory file
lightspeed_rag_image=registry.empresa.org/aap/byok:latest
```

### Referenciar BYOK no OCP

```yaml
spec:
  lightspeed:
    disabled: false
    chatbot_config_secret_name: chatbot-configuration-secret
    rag_image: registry.empresa.org/aap/byok:latest
```

## LLM Providers Suportados

| Provider | `chatbot_llm_provider_type` | Tipo |
|---|---|---|
| Red Hat Enterprise Linux AI (vLLM) | `rhelai_vllm` | Self-hosted |
| Red Hat OpenShift AI (vLLM) | `rhoai_vllm` | Self-hosted |
| Red Hat AI Inference Server | — | Self-hosted |
| OpenAI | `openai` | SaaS |
| Microsoft Azure OpenAI | `azure_openai` | SaaS |
