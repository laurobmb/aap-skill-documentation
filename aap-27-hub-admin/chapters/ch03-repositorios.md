# Ch03 — Repositórios do Hub: Tipos, Criação e Sincronização

## Tipos de Repositório

| Tipo | Pipeline | Quem publica? | Visível na busca? |
|---|---|---|---|
| **Staging** | `pipeline=staging` | Qualquer usuário com permissão no namespace | Não — aparece no approval dashboard |
| **Approved** | `pipeline=approved` | Somente via promoção do staging | Sim (padrão) |
| **Custom (None)** | sem pipeline | Qualquer usuário com permissão no repo | Opcional (pode ser privado) |

> Hub padrão vem com: 1 staging repo + 1 approved repo + 1 rejected repo (pré-instalados)

## Criar Repositório Customizado

```
Automation Content → Repositories → Create repository

Campos:
  Name:                     repo-automacoes-infra
  Description:              Coleções aprovadas do time de infraestrutura
  Retained number of versions: 5  (null = ilimitado; útil para rollback)
  Pipeline:
    Staging  → qualquer um com namespace permission pode publicar
    Approved → requer aprovação via staging
    None     → sem pipeline (publicação direta para quem tem permissão no repo)
  ☐ Hide from search         → ocultar da busca global
  ☐ Make private             → oculto para quem não tem permissão
  Remote:    selecionar remoto para sync automático (opcional)

→ Create repository
```

## Gerenciar Versões do Repositório (Rollback)

```
Automation Content → Repositories → [repo] → Versions tab

Cada adição/remoção de coleção cria nova versão.
Para rollback:
  → More Actions (⋮) → Revert to this version → confirmar
```

> Rollback é seguro — não deleta conteúdo do sistema, apenas muda quais coleções estão associadas ao repo.

## Configurar Acesso ao Repositório (RBAC)

```
Automation Content → Repositories → [repo] → Team Access tab
  → Assign teams → selecionar time → Next
  → selecionar papéis → Finish
```

> Por padrão: repositórios privados e conteúdo são ocultos. Repositórios públicos podem ser vistos por todos mas não modificados.

## Criar Remote Configuration

```
Automation Content → Remotes → Create Remote

Campos:
  Name:    remote-galaxy-empresa
  URL:     https://galaxy.ansible.com/api/ (ou URL de outro Hub)
  Token:   (se necessário para autenticação)
  Username / Password: (alternativa ao token)

Opções avançadas:
  TLS validation: ☑ (recomendado)
  CA certificate: (para certificados self-signed)
  Proxy URL/User/Pass: (se atrás de proxy)
  Download concurrency: N (aceleração de download paralelo)
  Rate limit: N (req/s — cuidado com servidores com limite)
  ☑ Signed collections only: sincronizar apenas coleções assinadas
  ☑ Sync all dependencies: incluir dependências automaticamente

Requirements file (filtrar quais coleções sincronizar):
  collections:
    - name: community.kubernetes
    - name: community.aws
      version: ">=5.0.0"
```

## Sincronizar Repositório com Remote

```
# Via UI:
Automation Content → Repositories → [repo] → More Actions (⋮) → Sync repository

# Verificar status:
Automation Content → Task Management
```

## Exportar/Importar Coleções

```
# Exportar (download .tar.gz):
Automation Content → Collections → [coleção] → Install tab → Download tarball

# Importar para outro Hub (ou ambiente air-gapped):
Automation Content → Namespaces → [namespace] → Collections → Upload Collection
  → selecionar .tar.gz → selecionar pipeline → Upload
```

## Sync de Conteúdo Certificado (rh-certified)

```bash
# Sequência completa para sincronizar Red Hat Certified Collections:
# 1. Configurar remote rh-certified (ch01)
# 2. Associar o remote a um repositório:
Automation Content → Repositories → rh-certified → Edit
  → Remote: rh-certified  → Save

# 3. Sincronizar:
Automation Content → Repositories → rh-certified
  → More Actions (⋮) → Sync repository

# 4. Verificar em Task Management
```
