# Glossário — AAP 2.7 Hub Admin

| Termo | Definição |
|---|---|
| **Private Automation Hub** | Registry centralizado da organização para coleções Ansible e Execution Environments |
| **Namespace** | Localização única no Hub para publicar e agrupar coleções. Governada por times com papéis |
| **Staging Repository** | Repositório de entrada para coleções aguardando aprovação. Marcado com `pipeline=staging`. Não aparece na busca |
| **Approved Repository** | Repositório de conteúdo aprovado para uso geral. Marcado com `pipeline=approved`. Aparece na busca |
| **Custom Repository** | Repositório sem pipeline de aprovação. Qualquer usuário com permissão pode publicar diretamente |
| **Collection Approvals** | Dashboard de revisão de coleções enviadas para staging, aguardando aprovação do admin |
| **Remote Configuration** | Configuração de acesso a um servidor externo (console.redhat.com, Galaxy, outro Hub) para sync |
| **rh-certified Remote** | Remote pré-configurado para sincronizar Red Hat Certified Content Collections de `console.redhat.com` |
| **Community Remote** | Remote pré-configurado para sincronizar coleções de `galaxy.ansible.com` |
| **Offline Token** | Token secreto gerado no console.redhat.com para autenticar sync de conteúdo certificado. Expira em 30d de inatividade |
| **Container Registry** | Parte do Hub que funciona como registry de imagens (Execution Environments) |
| **Execution Environment (EE)** | Container image com Ansible core + coleções + dependências para execução de jobs |
| **Auto Approve** | Configuração de repositório que promove automaticamente coleções de staging para approved sem revisão manual |
| **Requirements file** | Arquivo YAML listando coleções e versões a sincronizar de um remote. Evita sincronizar o catálogo inteiro |
| **GPG Signing** | Assinatura digital de coleções para garantir integridade e autenticidade do conteúdo |
| **Signature Keys** | Chaves GPG públicas visíveis no Hub para download e verificação de coleções e containers assinados |
| **Pipeline label** | Atributo de repositório (`pipeline=staging`, `pipeline=approved`) que define o fluxo de publicação |
| **Retained versions** | Número de versões de um repositório mantidas para rollback. Null = ilimitado |
| **Task Management** | Página do Hub que mostra o status de operações assíncronas como syncs de repositório |
