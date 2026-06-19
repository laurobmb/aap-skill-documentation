# Cheatsheet — AAP 2.7 Extensão (MCP, IA, Integrações)

## Ansible MCP Server — Deploy Rápido

```ini
# Containerizado (inventory)
[ansiblemcp]
aap.exemplo.org

[all:vars]
mcp_allow_write_operations=false   # false=read-only (SEGURO)
```

```yaml
# OCP (Custom Resource)
spec:
  mcp:
    disabled: false
    allow_write_operations: false
```

## Conectar Cursor ao MCP Server

```json
{
  "mcpServers": {
    "aap-jobs": {
      "type": "http",
      "url": "https://aap.exemplo.org:8448/job_management/mcp",
      "headers": {"Authorization": "Bearer ${env:AAP_MCP_TOKEN}"}
    }
  }
}
```

## MCP Toolsets (URLs)

```
/job_management/mcp
/inventory_management/mcp
/system_monitoring/mcp
/user_management/mcp
/security_compliance/mcp
/platform_configuration/mcp
```

## Intelligent Assistant — Secret OCP

```yaml
stringData:
  chatbot_llm_provider_type: openai
  chatbot_url: https://api.openai.com/v1
  chatbot_model: gpt-4o-mini
  chatbot_token: <key>
  # MCP (opcional)
  aap_gateway_url: http://myaap
  aap_controller_url: http://myaap-controller-service
```

## BYOK — Gerar Vector DB

```bash
podman pull registry.redhat.io/lightspeed-core/rag-tool-rhel9:v0.5-latest
podman run --rm -u 0 \
  -v "$(pwd)/docs:/input:Z" \
  -v "$(pwd)/output:/output:Z" \
  <rag-tool> python .../generate_embeddings.py \
    -f /input -o /output/vector_db -i minha-org-index \
    --exclude-model --output-image /output/byok.tar \
    --image-name byok --image-tag latest
```

## Ansible MCP Server vs Intelligent Assistant

| | MCP Server | Intelligent Assistant |
|---|---|---|
| Porta | 8448 | UI interna |
| Consumidor | AI agents externos | Usuários AAP |
| LLM requerido | Não | Sim |
| Status 2.7 | Tech Preview | GA (OCP) / TP (Container) |
