# Ansible Playbook to deploy Ethereum Clients, MEVBoost and DVT Nodes

This Ansible playbook automates the deployment and configuration of Ethereum Clients and DVT Nodes. It ensures that the necessary dependencies, configuration files, and services are properly set up and running.

Ethereum Clients:
- Lighthouse
- Prysm
- Geth
- Nethermind

DVT Clients:
- Obol
- SSV

## Table of Contents

- [Requirements](#requirements)
- [Prerequisites](#prerequisites)
- [Setup](#setup)
- [Variables](#variables)
- [Usage](#usage)

## Requirements

Before using this playbook, ensure the following requirements are met:

1. **Ansible version**: Make sure you have Ansible 2.15+ installed.
2. **SSH Access**: Passwordless SSH access to all target servers.
3. **Python**: Python 3.x installed on the control node and all target hosts.
4. **Privileges**: The user running the playbook must have sudo privileges on the target machines.

## Prerequisites

**Install HashiCorp Vault**

This playbook relies on HashiCorp Vault to securely retrieve sensitive files, such as validator keys and secrets. Follow the [HashiCorp Vault Installation Guide](https://developer.hashicorp.com/vault/tutorials/getting-started/getting-started-install) to set up Vault on your infrastructure.

**Note on Secrets Management**

The playbook dynamically retrieves private keys and environment secrets for Mina from HashiCorp Vault. The keys and secrets follow a structured path format:
`<environment>/<project>/<organization>/{{ cluster }}/<type>/<file_name>`

This structure ensures easy organization and secure retrieval of secrets.

For obol setup we need a folder structure to fetch all the validator keys:
`<environment>/<project>/<organization>/{{ cluster }}/<type>/<folder_name>`

## To set up the services, you need to upload specific key files with the following names:

1. For ssv-dkg service, upload:
- encrypted_private_key.json
- password
- tls.key

2. For ssv-node service, upload:
- encrypted_private_key.json
- password

3. For obol-charon service, upload:
- cluster-lock.json
- charon-enr-private-key
- validator_keys (directory containing all validator key files)

## Setup

### 1. Install Ansible

If Ansible is not installed, visit the official documentation for detailed instructions on how to install Ansible on various Linux distributions:

[Ansible Installation Guide](https://docs.ansible.com/ansible/latest/installation_guide/installation_distros.html)

### 2. Clone the repository

Clone this repository to your Ansible control node:

```bash
git clone https://github.com/encapsulate-xyz/ethereum-ansible.git
cd ethereum-ansible
```

### 3. Inventory

Define your target servers' IP address or DNS in the inventory folder, and select either `mainnet.yml` or `testnet.yml` to update.

Example for `mainnet.yml`

```yaml
---
# maintains mainnet inventory grouped by project names
all:
  vars:
    env: mainnet
  children:
    ethereum:
      hosts:

        lighthouse.consensus.ethereum.mainnet.encapsulate.xyz:
          type: consensus
          client: lighthouse

        nethermind.execution.ethereum.mainnet.encapsulate.xyz:
          type: execution
          client: nethermind

        prysm.consensus.ethereum.mainnet.encapsulate.xyz:
          type: consensus
          client: prysm

        geth.execution.ethereum.mainnet.encapsulate.xyz:
          type: execution
          client: geth

        lighthouse.mevboost.ethereum.mainnet.encapsulate.xyz:
          client: lighthouse
          type: mevboost

        prysm.mevboost.ethereum.mainnet.encapsulate.xyz:
          client: prysm
          type: mevboost

    ssv:
      hosts:

        arid.node.ssv.mainnet.encapsulate.xyz:
          type: node
          cluster: arid

        arid.dkg.ssv.mainnet.encapsulate.xyz:
          type: dkg
          cluster: arid

        manta.node.ssv.mainnet.encapsulate.xyz:
          type: node
          cluster: manta

        manta.dkg.ssv.mainnet.encapsulate.xyz:
          type: dkg
          cluster: manta

    obol:
      hosts:

        zephyr.charon.obol.mainnet.encapsulate.xyz:
          type: charon
          cluster: zephyr

        zephyr.ejector.obol.mainnet.encapsulate.xyz:
          type: ejector
          cluster: zephyr
          obol_dv_exit_message_dir: /opt/obol-dv-exit-{{ cluster }}/data

        zephyr.dvexit.obol.mainnet.encapsulate.xyz:
          type: dv_exit
          cluster: zephyr
          charon_run_time_dir: /opt/obol-charon-{{ cluster }}/key

        zephyr.lodestar.obol.mainnet.encapsulate.xyz:
          type: lodestar
          cluster: zephyr
```

## Variables

This playbook allows customization through several variables. You can define these variables in the following locations:

- **`group_vars/all.yml`**: Contains all the port, source URL configurations.
- **`group_vars/mainnet.yml`** or **`group_vars/testnet.yml`**: Contains version-specific variables.

**Note**: Sensitive variables such as private keys and passwords are not stored in `vault.yml`. They are dynamically retrieved from HashiCorp Vault during the playbook run.

### Usage

1. First, install the dependencies:

```bash
ansible-galaxy role install -r roles/requirements.yml
ansible-galaxy collection install -r collections/requirements.yml
```

2. Configure your remote server username and private key file path in the `ansible.cfg` file. Additionally, set the SSH port for your server by adjusting the `ansible_port` variable in `group_vars/all.yml`.

3. Run the playbook:

- To deploy ethereum lighthouse client:

```bash
ansible-playbook setup_node.yml -i inventory/mainnet.yml --limit lighthouse.consensus.ethereum.mainnet.encapsulate.xyz
```

**Example Output:**

```bash
TASK [Display environment being deployed to] ***************************************************************************************************
ok: [lighthouse.consensus.ethereum.mainnet.encapsulate.xyz] => {
    "msg": [
        "Deploying to Host: lighthouse.consensus.ethereum.mainnet.encapsulate.xyz",
        "Groups: ['ethereum']",
        "Project: ethereum'",
        "Environment: mainnet",
        "Type: consensus",
        "Client: lighthouse",
        "Version: v7.0.0",
        "Service Name: ethereum-consensus-lighthouse'",
        "Operating System: linux",
        "System Architecture: amd64"
    ]
}

TASK [Confirm deployment details] **************************************************************************************************************
[Confirm deployment details]
Press 'Enter' to continue with the deployment or Ctrl+C to cancel:
ok: [lighthouse.consensus.ethereum.mainnet.encapsulate.xyz]

TASK [Please confirm again] ********************************************************************************************************************
ok: [lighthouse.consensus.ethereum.mainnet.encapsulate.xyz] => {
    "msg": [
        "Deploying to Host: lighthouse.consensus.ethereum.mainnet.encapsulate.xyz",
        "Project: ethereum'",
        "Environment: mainnet",
        "Type: consensus"
    ]
}

TASK [Confirm deployment details] **************************************************************************************************************
[Confirm deployment details]
Press 'Enter' to continue with the deployment or Ctrl+C to cancel:
ok: [lighthouse.consensus.ethereum.mainnet.encapsulate.xyz]
```
