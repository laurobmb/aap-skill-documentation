# Ch02 — Filtros, Ordenação e Queries Avançadas

## Filtros Básicos

```bash
# Filtrar por valor exato (default):
GET /api/controller/v2/hosts/?name=web01.empresa.org

# Filtrar por organização:
GET /api/controller/v2/job_templates/?organization__name=Infraestrutura

# Caracteres especiais: URL-encode obrigatório
GET /api/controller/v2/hosts/?name=host%20com%20espaco
```

## Field Lookups (Operadores de Comparação)

```bash
# Sintaxe: ?field__lookup=value

# Comparações de texto:
?name__exact=web01          # exato (padrão)
?name__iexact=WEB01         # case-insensitive
?name__contains=web         # contém
?name__icontains=WEB        # contém, case-insensitive
?name__startswith=web       # começa com
?name__endswith=.org        # termina com
?name__regex=^web[0-9]+     # expressão regular
?name__iregex=^web[0-9]+    # regex case-insensitive

# Comparações numéricas/datas:
?created__gt=2024-01-01             # criado depois de
?created__gte=2024-01-01            # criado a partir de
?finished__lt=2024-12-31            # terminou antes de
?elapsed__gte=3600                  # durou mais de 3600 segundos
?job_slice_count__gt=1              # job templates com slicing

# Nulo:
?credential__isnull=true            # sem credencial
?finished__isnull=false             # jobs que terminaram

# Lista (IN):
?status__in=failed,error            # status failed ou error
?id__in=1,5,10,15                  # IDs específicos

# Boolean:
?is_superuser=true  (ou 1 ou True)
?active=false       (ou 0 ou False)

# Acesso por nível de role:
?role_level=admin_role
```

## Filtros AND, OR, NOT, Chain

```bash
# AND (padrão — todos os filtros aplicados juntos):
GET /api/controller/v2/jobs/?status=failed&job_type=run

# OR — prefixo or__:
?or__status=failed&or__status=error   # failed OU error

# NOT — prefixo not__:
?not__status=successful               # não bem-sucedido

# OR NOT — combinar:
?or__not__status=successful&or__not__status=pending

# CHAIN — aplicar filtros separadamente para cada objeto relacionado:
?chain__related__field=value&chain__related__field2=othervalue
# (diferente de AND: AND aplica ambos filtros ao mesmo objeto relacionado)
```

## Filtros em Campos Relacionados

```bash
# Usar __ para traversar relações (até o banco de dados):
?inventory__organization__name=Infraestrutura
?job_template__name__icontains=deploy

# Exemplos práticos:
GET /api/controller/v2/jobs/?job_template__name=Deploy+App&status=failed
GET /api/controller/v2/hosts/?inventory__name__icontains=linux
GET /api/controller/v2/credentials/?credential_type__name=Machine
```

## Ordenação

```bash
# Ordenar por campo (crescente):
GET /api/controller/v2/jobs/?order_by=started

# Descendente (prefixo -):
GET /api/controller/v2/jobs/?order_by=-started

# Múltiplos campos:
GET /api/controller/v2/jobs/?order_by=-status,started
```

## Buscar Hosts por Fatos do Ansible

```bash
# Requer use_fact_cache=True no Job Template
# Sintaxe: ansible_facts__<chave>

GET /api/controller/v2/hosts/?host_filter=ansible_facts__ansible_processor_vcpus=8
GET /api/controller/v2/hosts/?host_filter=ansible_facts__ansible_distribution=RedHat
GET /api/controller/v2/hosts/?host_filter=ansible_facts__ansible_os_family=Debian

# Combinado com groups:
?host_filter=(name=localhost or name=database) and groups__name=producao
```
