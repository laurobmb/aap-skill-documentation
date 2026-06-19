# Glossário — AAP 2.7 Extensão

**Ansible MCP Server** — Componente standalone e deployável do AAP que expõe a plataforma como servidor MCP (porta 8448) para AI agents externos. Novo no AAP 2.7 (Tech Preview).

**MCP (Model Context Protocol)** — Protocolo aberto para que AI agents/LLMs interajam com sistemas externos de forma padronizada via tool calling.

**Toolset** — Conjunto de ferramentas expostas pelo MCP Server por categoria: job_management, inventory_management, system_monitoring, user_management, security_compliance, platform_configuration.

**Server-level permissions** — Controle global do MCP Server: read-only (padrão) ou read-write. Configurado pelo admin no deploy.

**User-level permissions** — Permissões herdadas do token do usuário pelo AI agent. O AI agent só pode fazer o que o usuário pode.

**BYOK (Bring Your Own Knowledge)** — Extensão do Intelligent Assistant com documentação interna da organização via RAG. Requer criar imagem de vector database. Tech Preview.

**RAG (Retrieval-Augmented Generation)** — Técnica que aumenta respostas de LLMs com busca em base de conhecimento vetorial em tempo real, reduzindo alucinações.

**Vector database** — Banco de dados que armazena embeddings de texto para busca semântica. Gerado pelo `rag-content tool` a partir de documentação (.md, .txt).

**rag-content tool** — Imagem container `rag-tool-rhel9` da Red Hat para gerar vector databases a partir de documentação, usados no BYOK.

**Intelligent Assistant** — Chatbot de IA embutido na UI do AAP. Usa LLM + RAG para responder perguntas sobre o ambiente. Diferente do MCP Server standalone.

**Lightspeed Coding Assistant** — Extensão VS Code que gera recomendações de tasks/playbooks Ansible via IBM watsonx Code Assistant.

**Ansible Code Bot** — Bot para GitHub/GitLab que analisa repositórios e abre Pull Requests automáticos com sugestões de melhoria (FQCNs, módulos deprecados).
