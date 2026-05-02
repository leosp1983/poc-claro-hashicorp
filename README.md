# poc-claro-hashicorp

POC de automação Ansible para gestão de infraestrutura HashiCorp na Claro.

## Estrutura do Projeto

```
poc-claro-hashicorp/
├── group_vars/
│   └── all.yaml          # Variáveis globais (Vault, Consul, TFE)
├── roles/
│   └── hashicorp/
│       ├── vault/
│       │   ├── tasks/main.yaml
│       │   └── vars/main.yaml
│       ├── consul/
│       │   ├── tasks/main.yaml
│       │   └── vars/main.yaml
│       └── terraform/
│           ├── tasks/main.yaml
│           └── vars/main.yaml
├── ansible.cfg           # Configuração do Ansible
├── hosts.yaml            # Inventário estático
├── site.yaml             # Playbook principal
└── README.md
```

## Pré-requisitos

- Ansible >= 2.15
- Python >= 3.10
- Acesso SSH aos hosts de destino

## Variáveis de Ambiente

| Variável        | Descrição                          |
|-----------------|------------------------------------|
| `VAULT_TOKEN`   | Token de acesso ao HashiCorp Vault |
| `CONSUL_TOKEN`  | Token de acesso ao Consul          |
| `TFE_TOKEN`     | Token do Terraform Enterprise      |

## Execução

```bash
# Rodar todas as roles
ansible-playbook site.yaml -i hosts.yaml

# Rodar apenas a role do Vault
ansible-playbook site.yaml -i hosts.yaml --tags vault

# Rodar apenas a role do Consul
ansible-playbook site.yaml -i hosts.yaml --tags consul

# Rodar apenas a role do Terraform
ansible-playbook site.yaml -i hosts.yaml --tags terraform

# Dry-run (check mode)
ansible-playbook site.yaml -i hosts.yaml --check
```