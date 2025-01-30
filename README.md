# INFR010ansible

README.MD GROUPE ESTEVES GIRAUD LERICHE MARTEAU

# Prérequis
Avant de commencer, assurez-vous d'avoir :

Un compte Azure actif.
Un terminal Linux ou une machine virtuelle Ubuntu.
Les droits nécessaires pour créer des ressources sur Azure.
Étape 1 : Installation des outils nécessaires

# Mettre à jour les paquets et installer curl :

sudo apt update -y
sudo apt install curl

# Installer uv (Universal Version Manager) :
curl -LsSf https://astral.sh/uv/install.sh | sh

# Installer Python et créer un environnement virtuel :
uv python install
uv venv ansible_ipi
ansible_ipi/bin/activate

# Activer l'environnement virtuel :
source ansible_ipi/bin/activate

# Mettre à jour pip et installer Ansible :
uv tool run pip install --upgrade pip
uv tool run pip install ansible

# Installer la collection Azure pour Ansible :
ansible-galaxy collection install azure.azcollection --force
uv tool run pip install -r ~/.ansible/collections/ansible_collections/azure/azcollection/requirements.txt
uv tool run pip install ansible[azure]

# Configurer les alias pour les commandes Ansible :
alias pip="uv tool run pip"
alias ansible="uv tool run ansible"
alias ansible-playbook="uv tool run ansible-playbook"
alias ansible-vault="uv tool run ansible-vault"

Voici le Bash d'installation:
#!/bin/bash
# Mettre à jour les paquets système
sudo apt update -y
# Installer curl s'il n'est pas déjà installé
sudo apt install -y curl
# Installer le gestionnaire d'installation Astral (uv)
curl -LsSf https://astral.sh/uv/install.sh | sh
# Installer Python et venv si nécessaire
sudo apt install -y python3 python3-venv
# Créer un environnement virtuel Python
python3 -m venv ansible_ipi
# Activer l'environnement virtuel
source ansible_ipi/bin/activate
# Mettre à jour pip dans l'environnement virtuel
pip install --upgrade pip
# Installer Ansible dans l'environnement virtuel
pip install ansible
# Installer la collection Azure pour Ansible
ansible-galaxy collection install azure.azcollection --force
# Installer les dépendances supplémentaires de la collection Azure
pip install -r ~/.ansible/collections/ansible_collections/azure/azcollection/requirements.txt
# Installer Azure CLI
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
echo "Installation terminée !"

# Étape 2 : Configuration des variables

Créez un fichier .yml contenant les variables suivantes :

resource_group_name: MARTEAUTEST
location: westeurope
vnet_name: TESTNETWORK
subnet_name: TESTSUBNET
public_ip_name: my_public_ip
security_group_name: TESTSECURITYGROUP
network_interface_name: nic001
vm_name: TESTVM
admin_username: emarteau
admin_password: AzErCXZ876!
vm_size: Standard_B2ms
image_offer: UbuntuServer
image_publisher: Canonical
image_sku: '18.04-LTS'
image_version: latest
address_prefix_vnet: 10.1.0.0/16
address_prefix_subnet: 10.1.0.0/24
Assurez-vous de remplacer les valeurs par celles correspondant à votre environnement.

# Étape 3 : Création des ressources Azure
Créez un fichier .yml avec le contenu suivant :

- hosts: localhost
  connection: local
  vars_files:
    - vars.yml

  tasks:
    - name: Create a resource group
      azure_rm_resourcegroup:
        name: "{{ resource_group_name }}"
        location: "{{ location }}"
        state: present
        tags:
          testing: testing
          delete: never

    - name: Create a virtual network
      azure_rm_virtualnetwork:
        resource_group: "{{ resource_group_name }}"
        name: "{{ vnet_name }}"
        location: "{{ location }}"
        state: present
        address_prefixes_cidr:
          - "{{ address_prefix_vnet }}"

    - name: Create a subnet
      azure_rm_subnet:
        resource_group: "{{ resource_group_name }}"
        virtual_network_name: "{{ vnet_name }}"
        name: "{{ subnet_name }}"
        address_prefix_cidr: "{{ address_prefix_subnet }}"

    - name: Create a public IP address
      azure_rm_publicipaddress:
        resource_group: "{{ resource_group_name }}"
        name: "{{ public_ip_name }}"
        location: "{{ location }}"
        state: present
        allocation_method: static

    - name: Create a security group
      azure_rm_securitygroup:
        resource_group: "{{ resource_group_name }}"
        name: "{{ security_group_name }}"
        purge_rules: true
        rules:
          - name: "Deny-AllIn"
            priority: 101
            direction: Inbound
            access: Deny
            protocol: "*"
            source_address_prefix: "*"
            destination_address_prefix: "*"
            source_port_range: "*"
            destination_port_range: "*"

          - name: 'AllowSSHIn'
            protocol: Tcp
            access: Allow
            direction: Inbound
            priority: 100
            source_address_prefix: "*"
            destination_address_prefix: "*"
            destination_port_range: 22

          - name: "Deny-AllOut"
            priority: 101
            direction: Outbound
            access: Deny
            protocol: "*"
            source_address_prefix: "*"
            destination_address_prefix: "*"
            source_port_range: "*"
            destination_port_range: "*"

          - name: 'AllowSSHOut'
            protocol: Tcp
            access: Allow
            direction: Outbound
            priority: 100
            source_address_prefix: "*"
            destination_address_prefix: "*"
            destination_port_range: 22

    - name: Create a network interface
      azure_rm_networkinterface:
        name: "{{ network_interface_name }}"
        resource_group: "{{ resource_group_name }}"
        virtual_network: "{{ vnet_name }}"
        location: "{{ location }}"
        state: present
        subnet_name: "{{ subnet_name }}"
        security_group: "{{ security_group_name }}"
        ip_configurations:
          - name: ipconfig1
            public_ip_address_name: "{{ public_ip_name }}"
            primary: true

    - name: Create VM with defaults
      azure_rm_virtualmachine:
        resource_group: "{{ resource_group_name }}"
        name: "{{ vm_name }}"
        admin_username: "{{ admin_username }}"
        admin_password: "{{ admin_password }}"
        location: "{{ location }}"
        vm_size: "{{ vm_size }}"
        state: present
        network_interfaces: "{{ network_interface_name }}"
        image:
          offer: "{{ image_offer }}"
          publisher: "{{ image_publisher }}"
          sku: "{{ image_sku }}"
          version: "{{ image_version }}"
		  
# Étape 4 : Exécution du playbook

Exécuter le playbook :
ansible-playbook deploy_vm.yml

Vérifier la création de la machine virtuelle :
az
