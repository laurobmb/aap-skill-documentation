# Ch01 — Projetos EDA e Decision Environments

## Projetos EDA

Projetos são coleções lógicas de rulebooks armazenados em repositório Git.

> **Localização obrigatória dos rulebooks no repo:** `/extensions/eda/rulebooks/`

> **Dependência de Redis:** Criação e sync de projetos requerem Redis disponível. Sem Redis → operações falham.

## Criar Projeto EDA

```
Automation Decisions → Projects → Create project

Campos:
  Name:                      glpi-automacoes
  Organization:              Infraestrutura
  Source control type:       Git (único tipo disponível)
  Source control URL:        https://github.com/empresa/eda-rulebooks.git
  Source control branch:     main  (ou tag, commit hash, ref)
  Source control refspec:    (opcional — para buscar PRs ou refs especiais)
  Source control credential: (credencial para repos privados)
  Proxy:                     (HTTP/HTTPS proxy, se necessário)
  ☑ Verify SSL               (desmarcar para certs self-signed)
  Content signature validation credential: (GPG, opcional)

→ Create project
```

## Sync Manual do Projeto

```
Automation Decisions → Projects → [projeto]
  → ícone de sync (⟳) ao lado do projeto na listagem

# O Git hash atualiza para representar o commit mais recente.
# Uma activation usa o conteúdo do momento em que foi criada/reiniciada.
# Após atualizar o projeto, REINICIAR as activations para usar o novo conteúdo.
```

## Editar Projeto (Atenção: impacto em activations)

```
⚠️  Ao editar Source control URL, branch, ou refspec:
  → EDA automaticamente dispara um resync
  → Activations em execução continuam com o conteúdo ANTIGO
  → Para aplicar conteúdo novo: reiniciar manualmente as activations afetadas
  → Se a activation usa event streams: re-attach o stream APÓS reiniciar

Novos rulebooks adicionados ao repo: disponíveis após sync
Rulebooks deletados do repo: removidos do banco após sync (activations continuam, mas podem ser reiniciadas)
```

## Event Sources Suportados (de-minimal image)

| Plugin | Tipo | Nome deprecated |
|---|---|---|
| `eda.builtin.webhook` | Event source | `ansible.eda.webhook` |
| `eda.builtin.generic` | Event source | `ansible.eda.generic` |
| `eda.builtin.range` | Event source | `ansible.eda.range` |
| `eda.builtin.pg_listener` | Event source | `ansible.eda.pg_listener` |
| `eda.builtin.dashes_to_underscores` | Event filter | `ansible.eda.dashes_to_underscores` |
| `eda.builtin.json_filter` | Event filter | `ansible.eda.json_filter` |
| `eda.builtin.normalize_keys` | Event filter | `ansible.eda.normalize_keys` |
| `eda.builtin.insert_hosts_to_meta` | Event filter | `ansible.eda.insert_hosts_to_meta` |
| `eda.builtin.event_splitter` | Event filter | — |

> Cloud sources (AWS, GCP, Azure) migraram para coleções dedicadas em `de-supported`. Verificar nomenclatura atual ao usar `de-supported`.

## Decision Environments (DE)

```
# DE padrão: incluída no AAP 2.7 (de-minimal, de-supported)
# Usar as imagens base do Hub:
  hub.empresa.org/de-minimal-rhel9:latest
  hub.empresa.org/de-supported-rhel9:latest

# Pull policy: sempre "always" — cada reinício da activation busca a imagem mais recente

# Construir DE customizada:
ansible-builder build -t hub.empresa.org/de-empresa:1.0 -f de-definition.yml
```
