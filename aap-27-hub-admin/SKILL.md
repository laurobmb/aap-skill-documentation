---
name: aap-27-hub-admin
description: >-
  Referência técnica de administração do Private Automation Hub no AAP 2.7. Cobre
  sincronização de conteúdo de repositórios remotos (rh-certified desde console.redhat.com,
  community desde Ansible Galaxy, remotos customizados), gerenciamento de namespaces
  (criar, permissões por time, upload de coleções, fluxo de aprovação), gerenciamento
  de repositórios (staging, approved pipeline, custom, versões), assinatura de conteúdo
  com GPG, container registry do Hub (pull/push/tag de Execution Environments, sync
  de registros externos, credenciais de registro, permissões de times), e tokens de
  autenticação (offline token via Gateway/Keycloak). Use quando precisar configurar
  o Hub para distribuir coleções certificadas para 30+ organizações, montar um fluxo
  de aprovação de coleções internas, gerenciar EEs no registry do Hub, sincronizar
  conteúdo de Satellite ou Galaxy, ou configurar assinatura GPG de conteúdo.
disable-model-invocation: true
---

# AAP 2.7 — Private Automation Hub: Administração

## Índice

| Capítulo | Conteúdo |
|---|---|
| [ch01-tokens-remotes](chapters/ch01-tokens-remotes.md) | Autenticação (offline token, API token), remote rh-certified, community Galaxy, proxy |
| [ch02-namespaces](chapters/ch02-namespaces.md) | Criar namespace, times/permissões, upload de coleções, aprovação/rejeição, pipeline |
| [ch03-repositorios](chapters/ch03-repositorios.md) | Tipos de repo (staging/approved/custom), criar, versões, sync de remoto |
| [ch04-container-registry](chapters/ch04-container-registry.md) | Pull/push/tag de EEs, sync de registry externo, credenciais, README, permissões |
| [ch05-assinatura](chapters/ch05-assinatura.md) | GPG signing, configurar Hub para assinatura, verificar coleções no cliente |

## Uso Rápido

- **Sincronizar coleções Red Hat Certified (console.redhat.com):** → ch01 + ch03
- **Configurar fluxo de aprovação de coleções internas:** → ch02
- **Fazer push de Execution Environment para o Hub:** → ch04
- **Habilitar assinatura GPG de conteúdo:** → ch05
- **Configurar time de devs com acesso a namespace específico:** → ch02
- **Criar remote customizado para Galaxy privado:** → ch03
