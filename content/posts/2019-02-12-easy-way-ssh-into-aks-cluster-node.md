---
author: Yoichi Kawasaki
date: "2019-02-12T08:32:40Z"
published: true
status: publish
images: ["/assets/20190212-arch-ssh-jumphost.png"]
tags:
- ssh
- azure
- kubernetes
- azure-kubernetes-services
- kubectl-plugin
title: Easy way to SSH into Azure Kubernetes Service cluster node VMs
---

This is an article on how you can SSH into Azure Kubernetes Service (AKS) cluster node VMs using [kubectl-plugin-ssh-jump](https://github.com/yokawasa/kubectl-plugin-ssh-jump).

## Motivation 

I wanted to SSH into Azure Kubernetes Service (AKS) cluster node VMs, then looking up azure docs I found a relevant page - [Connect with SSH to Azure Kubernetes Service (AKS) cluster nodes for maintenance or troubleshooting](https://docs.microsoft.com/en-us/azure/aks/ssh). But when I first saw this procedure, I thought this was very troublesome. Lazy person like me couldn't accept going thourgh all the steps just to SSH into AKS cluster nodes VMs. This is why I decided to create [kubectl-plugin-ssh-jump](https://github.com/yokawasa/kubectl-plugin-ssh-jump), a kubectl plugin to SSH into Kubernetes nodes using a SSH jump host Pod.

## AKS node access control and SSH access strategy

By default, AKS nodes are not exposed to the internet, and they are completely isolated to their own virtual network. This is why I took a strategy to go through a jump host Pod to connect to the Kubernetes nodes. I believe this is pretty a general strategy in dealing with firewalling, access privileges, etc.

![](/assets/20190212-arch-ssh-jumphost.png)

Regarding SSH key with which you access the nodes via SSH, you need to use the SSH key that you chosen in creating your AKS cluster. Or in the case that you lose the key, you can update SSH Key by using az command like this (See [this](https://docs.microsoft.com/en-us/azure/virtual-machines/extensions/vmaccess#update-ssh-key) for more detail):
```
az vm user update \
  --resource-group myResourceGroup \
  --name myVM \
  --username azureuser \
  --ssh-key-value ~/.ssh/id_rsa.pub
```

As an additional security enhancement, you can configure Access Control(IAM) on the AKS cluster to set permissions on the nodes on who can access them. Please see [What is role-based access control (RBAC)?](https://docs.microsoft.com/en-us/azure/role-based-access-control/overview) for the detail on RBAC.

## SSH into AKS node VMs using kubectl-plugin-ssh-jump

### Pre-requistes
This plugin needs the following programs:
* ssh(1)	
* ssh-agent(1)

### Installation

#### Install through krew
This is a way to install kubectl-ssh-jump through [krew](https://github.com/GoogleContainerTools/krew). After installing krew by following [this](https://github.com/GoogleContainerTools/krew#installation), you can install kubectl-ssh-jump like this:

```sh
$ kubectl krew install ssh-jump
```

Expected output would be like this:
```
Updated the local copy of plugin index.
Installing plugin: ssh-jump
CAVEATS:
\
 |  This plugin needs the following programs:
 |  * ssh(1)
 |  * ssh-agent(1)
 |
 |  Please follow the documentation: https://github.com/yokawasa/kubectl-plugin-ssh-jump
/
Installed plugin: ssh-jump
```

Once it's installed, run:
```sh
$ kubectl plugin list

The following kubectl-compatible plugins are available:

/Users/yoichika/.krew/bin/kubectl-krew
/Users/yoichika/.krew/bin/kubectl-ssh_jump

$ kubectl ssh-jump
```

#### Manual Installation

Install the plugin by copying the script in the $PATH of your shell.

```sh
# Get source
$ git clone https://github.com/yokawasa/kubectl-plugin-ssh-jump.git
$ cd kubectl-plugin-ssh-jump
$ chmod +x kubectl-ssh-jump
# Add kubeclt-ssh-jump to the install path.
$ sudo cp -p kubectl-ssh-jump /usr/local/bin
```

Once in the $PATH, run:
```sh
$ kubectl plugin list

The following kubectl-compatible plugins are available:
/usr/local/bin/kubectl-ssh-jump

$ kubectl ssh-jump
```

### How to use

#### Usage

```TXT
Usage:
  kubectl ssh-jump <dest_node> [options]

Options:
  <dest_node>                     Destination node IP
  -u, --user <sshuser>            SSH User name
  -i, --identity <identity_file>  Identity key file
  -p, --pubkey <pub_key_file>     Public key file
  --skip-agent                    Skip automatically starting SSH agent and adding
                                  SSH Identity key into the agent before SSH login
                                  (=> You need to manage SSH agent by yourself)
  --cleanup-agent                 Clearning up SSH agent at the end
                                  The agent is NOT cleaned up in case that 
                                  --skip-agent option is given
  --cleanup-jump                  Clearning up sshjump pod at the end
                                  Default: Skip cleaning up sshjump pod
  -h, --help                      Show this message
```

##### Option parameters Cache
`username`, `identity`, `pubkey` options are cached, therefore you can omit these options afterward. The options are stored in a file named `$HOME/.kube/kubectlssh/options`
```
$ cat $HOME/.kube/kubectlssh/options
sshuser=azureuser
identity=/Users/yokawasa/.ssh/id_rsa_k8s
pubkey=/Users/yokawasa/.ssh/id_rsa_k8s.pub
```

##### SSH Agent (ssh-agent)

The plugin automatically check if there are any `ssh-agents` started running by the plugin, and starts `ssh-agent`if it doesn't find any `ssh-agent` running and adds SSH Identity key into the agent before SSH login. If the command find that ssh-agent is already running, it doesn't start a new agent, and re-use the agent.
Add `--cleanup-agent` option if you want to kill the created agent at the end of command.

In addtion, add `--skip-agent` option if you want to skip automatic starting `ssh-agent`. This is actually a case where you already have ssh-agent managed or you want to manually start the agent.

#### Examples

Show all node list. Simply executing `kubectl ssh-jump` gives you the list of destination nodes as well as command usage

```sh 
$ kubectl ssh-jump

Usage:
  kubectl ssh-jump <dest_node> [options]

Options:
  <dest_node>                     Destination node IP
  -u, --user <sshuser>            SSH User name
  -i, --identity <identity_file>  Identity key file
  -p, --pubkey <pub_key_file>     Public key file
  --skip-agent                    Skip automatically starting SSH agent and adding
                                  SSH Identity key into the agent before SSH login
                                  (=> You need to manage SSH agent by yourself)
  --cleanup-agent                 Clearning up SSH agent at the end
                                  The agent is NOT cleaned up in case that
                                  --skip-agent option is given
  --cleanup-jump                  Clearning up sshjump pod at the end
                                  Default: Skip cleaning up sshjump pod
  -h, --help                      Show this message

Example:
   ....

List of destination node...
Hostname
aks-nodepool1-18558189-0
aks-nodepool1-18558189-1
aks-nodepool1-18558189-2
```

Then, SSH into a node `aks-nodepool1-18558189-0` with options like:
- usernaem: `azureuser`
- identity:`~/.ssh/id_rsa_k8s`
- pubkey:`~/.ssh/id_rsa_k8s.pub`)
```sh
$ kubectl ssh-jump aks-nodepool1-18558189-0 \
  -u azureuser -i ~/.ssh/id_rsa_k8s -p ~/.ssh/id_rsa_k8s.pub
```

As explained in usage secion, `username`, `identity`, `pubkey` options are cached, therefore you can omit these options afterward.

```sh
$ kubectl ssh-jump aks-nodepool1-18558189-0
```

You can pass the commands to run in the destination node like this (Suppose that `username`, `identity`, `pubkey` options are cached):
```sh
echo "uname -a" | kubectl ssh-jump aks-nodepool1-18558189-0

(Output)
Linux aks-nodepool1-18558189-0 4.15.0-1035-azure #36~16.04.1-Ubuntu SMP Fri Nov 30 15:25:49 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
```


You can clean up sshjump pod at the end of the command with `--cleanup-jump` option, otherwise, the sshjump pod stay running by default.
```sh
$ kubectl ssh-jump aks-nodepool1-18558189-0 \
  -u azureuser -i ~/.ssh/id_rsa_k8s -p ~/.ssh/id_rsa_k8s.pub \
  --cleanup-jump
```

You can clean up ssh-agent at the end of the command with `--cleanup-agent` option, otherwise, the ssh-agent process stay running once it's started.
```sh
$ kubectl ssh-jump aks-nodepool1-18558189-0 \
  -u azureuser -i ~/.ssh/id_rsa_k8s -p ~/.ssh/id_rsa_k8s.pub \
  --cleanup-agent
```
You can skip starting `ssh-agent` by giving `--skip-agent`. This is actually a case where you already have ssh-agent managed. Or you can start new ssh-agent and add an identity key to the ssh-agent like this:

```sh
# Start ssh-agent manually
$ eval `ssh-agent`
# Add an arbitrary private key, give the path of the key file as an argument to ssh-add
$ ssh-add ~/.ssh/id_rsa_k8s
# Then, run the plugin with --skip-agent
$ kubectl ssh-jump aks-nodepool1-18558189-0 \
  -u azureuser -i ~/.ssh/id_rsa_k8s -p ~/.ssh/id_rsa_k8s.pub \
  --skip-agent

# At the end, run this if you want to kill the current agent
$ ssh-agent -k
```
