# Patterns — AAP 2.7 Extensão (MCP, IA, Integrações)

## Padrão 1: Conectar Cursor ao AAP via MCP (Read-Only)

```
Cenário: time de consultoria usa Cursor para consultar o estado do AAP

1. Deploy MCP Server (containerizado):
   [ansiblemcp]
   aap.empresa.org
   [all:vars]
   mcp_allow_write_operations=false  ← seguro: apenas leitura

2. Criar token de API para cada consultor:
   Gateway → Users → <nome> → Tokens → Create (scope: read)

3. Cada consultor configura mcp.json no Cursor:
   {
     "mcpServers": {
       "aap": {
         "type": "http",
         "url": "https://aap.empresa.org:8448/job_management/mcp",
         "headers": {"Authorization": "Bearer ${env:AAP_TOKEN}"}
       }
     }
   }

4. Usar no chat do Cursor:
   "Show me all failed jobs from today"
   "Which hosts are in the production inventory?"
   "What job templates does the Infra org have?"
```

## Padrão 2: BYOK para Documentação Interna de Automações

```
Cenário: chatbot AAP responde sobre runbooks internos + procedimentos da empresa

1. Preparar docs (markdown):
   mkdir -p docs/runbooks docs/procedures
   # converter PDFs/Word com docling ou pandoc

2. Gerar vector database:
   podman run rag-tool ... generate_embeddings.py -f docs/ -o output/

3. Publicar imagem:
   podman push registry.empresa.org/aap/byok-docs:latest

4. Configurar no AAP (inventory):
   lightspeed_rag_image=registry.empresa.org/aap/byok-docs:latest

5. Chatbot agora responde com conteúdo interno como prioridade
```

## Padrão 3: Terraform + AAP para Provisionamento Orquestrado

```
Fluxo GitOps:
  PR aprovado no repo Terraform
    → terraform apply
      → Terraform AAP provider dispara Job Template
        → Ansible executa provisionamento
          → output volta ao Terraform como resource attributes

Vantagem: Terraform gerencia estado + Ansible executa mudanças →
  melhor que Terraform sozinho (sem estado de configuração do OS)
```
