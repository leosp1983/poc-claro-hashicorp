# poc-claro-hashicorp

POC de automação Ansible para consulta e inventário de VMs no vCenter, integrada ao **HashiCorp Vault** e ao **Ansible Automation Platform (AAP)**.

## Visão Geral

Este projeto consulta informações de máquinas virtuais no VMware vCenter usando o módulo `community.vmware.vmware_vm_info`. As credenciais do vCenter são injetadas de forma segura pelo AAP por meio de uma credencial do tipo **VMware vCenter** (`poc-claro-hashicorp-VCenter`), que recupera os valores diretamente do HashiCorp Vault antes da execução do playbook.

## Estrutura do Projeto

```
poc-claro-hashicorp/
├── group_vars/
│   └── all.yaml              # Variáveis globais (placeholder)
├── roles/
│   └── vmware/
│       ├── tasks/
│       │   └── main.yaml     # Tarefas: coleta, exibição e relatório de VMs
│       └── vars/
│           └── main.yaml     # Variáveis padrão (vcenter_hostname, validate_certs)
├── ansible.cfg               # Configuração do Ansible (callbacks, SSH, timeouts)
├── vmware.yaml               # Playbook principal
└── README.md
```

## Pré-requisitos

- Ansible >= 2.15
- Coleção `community.vmware` instalada:
  ```bash
  ansible-galaxy collection install community.vmware
  ```
- Acesso ao vCenter via rede
- Credencial do tipo **VMware vCenter** configurada no AAP, integrada ao HashiCorp Vault

## Credenciais e Integração com Vault

As credenciais **não são armazenadas** no repositório. O AAP injeta as seguintes variáveis de ambiente em tempo de execução, lidas a partir do HashiCorp Vault:

| Variável de Ambiente | Descrição                        |
|----------------------|----------------------------------|
| `VMWARE_HOST`        | Hostname/IP do vCenter           |
| `VMWARE_USER`        | Usuário de acesso ao vCenter     |
| `VMWARE_PASSWORD`    | Senha de acesso ao vCenter       |

> **Fallback:** Se `VMWARE_HOST` não estiver definido no ambiente, o valor padrão `vcenter_hostname` definido em `roles/vmware/vars/main.yaml` é utilizado.

## Execução

### Via Ansible Automation Platform (AAP)

1. Importe o projeto no AAP apontando para este repositório.
2. Configure a credencial `poc-claro-hashicorp-VCenter` (tipo: VMware vCenter) com lookup no Vault.
3. Crie um **Job Template** utilizando o playbook `vmware.yaml`.
4. Execute o Job Template — as credenciais serão injetadas automaticamente.

### Via linha de comando (com variáveis de ambiente definidas)

```bash
export VMWARE_HOST="vcsa01.exemplo.com"
export VMWARE_USER="administrator@vsphere.local"
export VMWARE_PASSWORD="senha_segura"

ansible-playbook vmware.yaml
```

### Dry-run (check mode)

```bash
ansible-playbook vmware.yaml --check
```

## O que o Playbook Faz

1. **Coleta** informações de todas as VMs no vCenter (nome, estado de energia, IP, SO, datacenter, cluster).
2. **Exibe** um resumo de cada VM no output do Ansible.
3. **Gera um relatório** em formato YAML salvo localmente em `/tmp/vmware_report_<DATA>.yaml`.

## Configurações do Ansible (`ansible.cfg`)

| Parâmetro                  | Valor                                         | Descrição                                 |
|----------------------------|-----------------------------------------------|-------------------------------------------|
| `timeout`                  | `120`                                         | Timeout de conexão em segundos            |
| `show_custom_stats`        | `true`                                        | Exibe estatísticas customizadas ao final  |
| `host_key_checking`        | `false`                                       | Desabilita verificação de chave SSH       |
| `stdout_callback`          | `yaml`                                        | Saída formatada em YAML                   |
| `callbacks_enabled`        | `timer, profile_tasks, profile_roles`         | Métricas de tempo por task e role         |
| `ssh_connection.retries`   | `30`                                          | Tentativas de reconexão SSH               |
| `pipelining`               | `True`                                        | Melhora performance SSH                   |