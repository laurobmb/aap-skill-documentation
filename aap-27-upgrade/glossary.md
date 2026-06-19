# Glossário — AAP 2.7 Upgrade

**Breaking change** — Mudança que requer ação do cliente para manter funcionamento. No 2.7: remoção do RPM, remoção de acesso direto, remoção de tokens de componente.

**aap-detect-direct-component-access** — Ferramenta CLI (GitHub) que analisa logs NGINX do AAP 2.5/2.6 para detectar tráfego que bypass o Platform Gateway.

**uvx** — Executor de ferramentas Python via `uv`. Alternativa ao `pip install` para ferramentas temporárias.

**Pre-upgrade checklist** — Lista de verificações obrigatórias antes de atualizar para 2.7, incluindo detecção de acesso direto, migração de usuários e exportação de configs de auth.

**Post-upgrade requirements** — Ações necessárias após upgrade: recriar tokens, atualizar URLs, reconfigurar auth, atualizar CaC.

**ansible.platform** — Collection CaC unificada do AAP 2.7 (substitui `ansible.controller`, `ansible.hub`, `ansible.eda`).

**Fallback auth** — Mecanismo do AAP 2.6 que permitia usuários sem conta no Gateway autenticarem via senha do Controller. **Removido no 2.7**.

**Channel upgrade** — Atualização mudando o canal do Operator do OperatorHub (ex: `stable-2.6` → `stable-2.7`).

**In-channel upgrade** — Atualização de patch dentro do mesmo canal (ex: 2.7.1 → 2.7.2).

**InstallPlan** — Recurso OCP que registra pacotes a serem instalados ou atualizados por um Operator. Pode requerer aprovação manual.

**sos report** — Relatório de diagnóstico do sistema RHEL. Coletado antes do upgrade para uso no rollback ou suporte.

**Rollback** — Processo de restaurar estado anterior após upgrade problemático via `ansible-playbook restore.yml` com backup pré-upgrade.
