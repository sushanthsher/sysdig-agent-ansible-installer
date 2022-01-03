# Ansible Sysdig agent installer 
## What it can do
You can use this tool to install Sysdig-Agent on multiple clusters, hosts and systems like:
- Kubernetes
- Docker container on Linux 
- Systemd daemon on Linux
- OpenShift 3.11 / 4.8+

## Prerequisites
- Python 3.8+
- pip3
- Sysdig account access key

> Note: for the Openshift installation the oc command must be installed on the target node, also the ansible user used must have the cluster-admin role in order to modify ssc

## Installation

Create a virtual environment and activate it: 

```bash
python3 -m venv .venv
source .venv/bin/activate
```

Install the requirements.txt using pip (it might take a long time depending on your machine)

```bash
pip install -r requirements.txt
```

Install the ansible-galaxy requirements.yml

> It might not be required

```bash
ansible-galaxy install -r requirements.yml
```

## Personalize your inventory file, group_vars and secret

### Inventory file
First of all, you need to personalize your inventory file with all hosts that you want to install Sysdig. In the case of Systemd installation (any kind of Linux machine), you need to specify all hosts, but in the case of a Kubernetes installation, you only need to specify a host that has access to the Kubernetes environment (like a "bastion" machine).

For each type of machine, there is a children class that defines the type of installation that will be performed.

The inventory should look like this:

```ini
[cluster1]
host1

[cluster2]
host2
host3

[cluster3]
host4
host5

[cluster4]
host6

# clusters with docker installation
[docker:children]
cluster2

# clusters with systemd installation
[systemd:children]
cluster3

# clusters with k8s installation
[k8s:children]
cluster1

# clusters with ocp installation
[ocp:children]
cluster4

```

> you can also convert this INI file into a YAML for a new fashion look 

### group_vars
The group_vars directory contains YAML files that have to be named as the groups defined in the inventory. Inside of one of those files you have to define the following parameters:

|Parameter Name| Description|
|---|---|
|sysdig_collector|Sysdig backend endpoint|
|sysdig_collector_port|Sysdig backend endpoint port|
|cluster_name|Cluster name|
|agent_version|Image tag|
|sysdig_agent_namespace|Kubernetes namespace that will be created where the Sysdig agent will be installed|
|kube_config_path|Kubernetes / Openshift Connector path|
|resources_requests_cpu|Defines the cpu requests for the sysdig-agent container|
|resources_requests_memory|Defines the memory requests for the sysdig-agent container|
|resources_limit_cpu|Defines the cpu limit for the sysdig-agent container|
|resources_limit_memory|Defines the memory limit for the sysdig-agent container|
|nia_enabled|It installs the Node Image Analyzer component if set to True|
|nia_version|Node Image Analyzer Image tag|
|nia_api_endpoint|Node Image Analyzer api endpoint|
|nia_collector_endpoint|Node Image Analyzer collector endpoint|

### Vault
The vault file will contain a single variable that will be the SysDig account access key for the SaaS backend. 

|Parameter Name| Description|
|---|---|
|sysdig_access_key|a variable containing the account access key|

> you can find this parameter in your personal settings in the Sysdig SaaS or in the Agent Installation section

The vault file should look like this: 

```yaml
sysdig_access_key: XXXXXX-YOUR-AGENT-KEY-XXXXXX
```

To create it, you can simply run:

```bash
ansible-vault encrypt_string --name=sysdig_access_key > ./vaultfile.yaml
```

> you will be asked to insert the ansible vault password and after that will be reading plaintext input from stdin. (ctrl-d to end input, twice if your content does not already have a newline)

#### Vault

You can define a secret.txt where the ansible-vault password will be stored, __make sure that only you can access this file!__

```bash
echo "YOUR_SUPER_SECRET_PASSWORD" > secret.txt
```

At this point you can do:

```bash
ansible-vault encrypt_string --vault-password-file=secret.txt --name=sysdig_access_key XXXXXX-YOUR-AGENT-KEY-XXXXXX > vaultfile.yaml
```

## Run ansible to install sysdig
At this point you can start the installation process simply by running:
```bash
ansible-playbook sysdig-agent.install.yml --vault-password-file=secret.txt
```
## Contacts
If you have questions or suggestions feel free to contact us:

- ivan.villa@vistatech.it
- daniel.gessaghi@vistatech.it
