# Configuration for Sample GPU Cluster

This repository contains configuration files that implements Sample GPU Cluster based on [Conda Compute Cluster](https://github.com/vicoslab/ccc-deployment).

Server contains management for the following nodes:
* node1
* node2
* node3

## Deployment and management:

Infrastructure is deployed using `playbook/cluster-deploy.yml`: 
```bash
ansible-playbook playbook/cluster-deploy.yml -i inventory \
                 --vault-password-file <path-to-secret> -e vars_file=inventory/vault_vars/cluster-secrets.yml \
                 -e machines=<node-or-group-pattern> \
                 -e only_roles=<list of roles> 
```

Conda containers are deployed using `playbook/containers-deploy.yml`:
```bash
ansible-playbook playbook/containers-deploy.yml -i inventory \
                 -e machines=<node-or-group-pattern> \
                 -e containers=<list of STACK_NAME> \
                 -e users=<list of USER_EMAIL>
```

For more information see [github wiki](https://github.com/vicoslab/ccc-deployment#deplyoment-playbooks).

## Automatic deployment of containers:

**NOTE: Commits/push to this repo will automatically trigger updates of containers deployed on the cluster using `containers-deploy.yml` playbook.**

This can be used to add new containers or modify existing ones by simply updating [`group_vars/ccc-cluster/user-containers.yml`](sample-inventory/group_vars/ccc-cluster/user-containers.yml).
However, update of the cluster infrastructure with `cluster-deploy.yml` *IS NOT* done automatically and must be performed manually.

## Configuration organization:

Configuration is stored in `inventory` folder as:

* hosts definitions: [`your-cluster.yml`](inventory/your-cluster.yml) with `ccc-cluster` as main group of our cluster nodes
* cluster settings: [`group_vars/ccc-cluster/cluster-vars.yml`](inventory//group_vars/ccc-cluster/cluster-vars.yml)
* cluster secrets: [`vault_vars/cluster-secrets.yml`](inventory/vault-vars/cluster-secrets.yml) (requires --vault-password-file to unlock which should not be in repo!)
* host-specific settings: [`inventory/host_vars`](inventory/host_vars)

Cluster-wide settings contain principal configuration of the whole cluster and are sectioned into settings for individual roles. Settings are used both by the `playbook/cluster-deploy.yml` and `playbook/containers-deploy.yml` playbooks. 

Container and user configurations are stored in `group_vars/ccc-cluster/user-*` files as:

* list of containers for deployment as `deployment_containers` var in [`group_vars/ccc-cluster/user-containers.yml`](sample-inventory/group_vars/ccc-cluster/user-containers.yml)
* list of users for deployment as `deployment_users` var in [`group_vars/ccc-cluster/user-list.yml`](sample-inventory/group_vars/ccc-cluster/user-list.yml)
* list of users types as `deployment_types` var in [`group_vars/ccc-cluster/user-list.yml`](sample-inventory/group_vars/ccc-cluster/user-list.yml)

### Cluster secrets 

Cluster secrets are stored in a seperate `vault_vars` folder and should not be in present in `group_vars` to allow running `containers-deploy.yml` without needing vault secret. Secrets are instead loaded only for cluster deployment by adding `-e vars_file=inventory/vault_vars/cluster-secrets.yml` to `cluster-deploy.yml` playbook call.

Encrypted files can be edited by running:

```bash
ansible-vault edit inventory/vault_vars/cluster-secrets.yml --vault-pass <path-to-secret>
```
 
