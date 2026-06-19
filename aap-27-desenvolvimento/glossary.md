# Glossário — AAP 2.7 Desenvolvimento

**ansible-dev-tools** — Pacote Python que agrupa ferramentas de desenvolvimento Ansible: navigator, lint, molecule, builder, creator, tox-ansible.

**ansible-creator** — Ferramenta de scaffold para gerar estrutura de projetos de playbook ou coleções com melhores práticas.

**ansible-navigator** — CLI/TUI que executa playbooks dentro de Execution Environments. Modo stdout (CI/CD) ou TUI (interativo).

**Execution Environment (EE)** — Container OCI com Ansible, coleções e dependências. Garante reprodutibilidade de execução.

**EE Definition v3** — Formato atual do arquivo `execution-environment.yml` para ansible-builder 3.x.

**Decision Environment (DE)** — Container equivalente ao EE, mas para o EDA Controller (inclui source plugins e dependências de rulebooks).

**ansible-builder** — Ferramenta para construir EEs e DEs a partir de um definition file.

**Rulebook** — Arquivo YAML do EDA que define sources (entrada de eventos) e rules (condição + ação).

**Rulebook Activation** — Instância em execução de um rulebook no EDA Controller.

**Event Source** — Plugin que consome eventos (webhook, Kafka, AlertManager, etc.) para alimentar o EDA.

**MCP Server** — Servidor Model Context Protocol que expõe APIs externas (GitHub, AWS) como ferramentas chamáveis por AI agents ou playbooks Ansible.

**Ephemeral MCP** — Instância de MCP server provisionada apenas durante a execução de um job no Controller, sem resíduos após conclusão.

**ansible.mcp.run_tool** — Módulo Ansible da collection ansible.mcp para invocar ferramentas de um MCP server.

**ansible.platform** — Collection oficial Red Hat para Configuration as Code (CaC) no AAP 2.7.

**galaxy.yml** — Arquivo de metadados de uma coleção Ansible (namespace, name, version, dependencies).
