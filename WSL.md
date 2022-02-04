# Kubernetes / WSL Ninja

How to setup a ninja Linux terminal for kubernetes, specifically a cluster running on AWS EKS. This guide is intended to take noobs from zero to ninja, specifically using Windows as your OS.

References:
- [Scott's intro to WSL](https://www.youtube.com/watchv=A0eqZujVfYU)
- [Terminal preview](https://www.youtube.com/watch?v=NfgAOxfv0QU)
- [Switch panes](https://docs.microsoft.com/en-us/windows/terminal/panes#switching-between-panes)

## Linux shell

A power shell (not PowerShell) is essential for your ninja skills, you need a linux shell, believe me.  You don't need linux though, you can roll WSL:

- [Get Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/install-win10)

Why Linux?
- The whole container eco system is Linux. When you interact with your containers, you'll use linux
- Productivity, the command history, zsh and the lot just makes you way more productive
- Embrace the keyboard, it's fun

## Terminal

Shell <> terminal, a terminal is a program that runs the shell. Windows just launched a great one:

- [Get Windows terminal](https://www.microsoft.com/en-us/p/windows-terminal-preview/9n0dx20hk701?activetab=pivot:overviewtab)

## AWS CLI

Install the AWS cli on WSL. You can install it on Windows as well but you need it primarily on WSL.

Plenty of docs around on how to install it but I did this:

```
$ sudo apt-get update
$ sudo apt-get install -y python3-pip
$ pip3 install awscli --upgrade --user
```

AWS uses config files with your credentials to connect to the cloud, each account works a little different. Assuming a role (which I do a lot) can get tricky.

AWS CLI config files are located at `<user>/.aws`

Config file
```sh
[default]
output = json
region = eu-west-1
```

Credentials file (No real keys here of course):
```sh
## Your personal account
[default]
aws_access_key_id = XTTTIXXXX7WTTXXEXXXA
aws_secret_access_key = +OMXxJGx3xx8pXuQxxxxAVxx58XXp7xxxXxxXXe

## Company account
[company]
aws_access_key_id: ZTTTIZZZZ7WTTZZEZZZA
aws_secret_access_key: +OMXzJGz3zz8pXuQzzzzAVzz58XXp7zzzXzzXXe

## Connect with Company account and assume role to another account
[client-at-company]
role_arn=arn:aws:iam::111175111141:role/OrganizationAccountAccessRole
source_profile=company # Links back to company profile
```

You can use the CLI like this now:
```sh
aws sts get-caller-identity # Get the identity the CLI is using
aws s3 ls # Outputs S3 buckets for default (personal) account in eu-west-1
aws s3 ls --profile company # Company root account in eu-west-1
aws s3 ls --pfofile client-at-company # Assume role through company root account to client account 111175111141 and list S3 buckets in eu-west-1
```

## kubectl

Just follow the guide?  https://kubernetes.io/docs/tasks/tools/install-kubectl/

For my Windows friends I'll give a short explanation of what the heck you're doing:  On Linux, binaries (think exe, sort of) are awesome.  Kubernetes tools all use this mechanism:

1\. Download the binary (`kubectl`, `helm`) etc. I like getting it from github (e.g. https://github.com/kubernetes/kubectl/releases) but you'll always find it on the main page of the tool. Super ninja's use curl to download it.

```sh
$ curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
```

2\. Move it to your PATH. You should be familier with PATH from Windows. In Linux you just have a thousand options for getting binaries in your PATH, which could make it confusing.  My method (the best one, ahem): Move the binary to the `/usr/local/bin` dir, it's already in your PATH!

```sh
$ sudo mv ./kubectl /usr/local/bin/kubectl
```

3\. Permissions. This is a topic of it's own. Point is, you need to give the binary execution permissions:

```sh
chmod +x /usr/local/bin/kubectl
```

## ~/.kube/config

Just like the AWS CLI (and probably all CLI's), it stores a config in your user directory with access credentials to all your clusters.  A cool tip is to do a `git init` inside `~/.kube` to track changes you make by adding more clusters. There are plenty of ways to setup your config, we'll use AWS CLI for EKS:

To setup your `~/.kube/config` to connect to an AWS cluster:

```sh
$ aws eks --profile <AWS_PROFILE> --region eu-west-1 update-kubeconfig --name <CLUSTER_NAME>
```

## Zsh

This is optional but... The ultimate ninja uses Z shell and https://ohmyz.sh/

I use the [Lambda gitster](https://github.com/ergenekonyigit/lambda-gitster) theme, with my favourite feature being the current working directory showing the root of the git repo when you cd into a git repo.

![lambda-gitster.png](lambda-gitster.png)

## Command History

This is your number one skill.  CTRL + R is your friend, you search through commands you've already run. My whole brain is stored in my linux shell history.

I also use [this](https://github.com/zsh-users/zsh-autosuggestions) for auto suggestions based on history:

![auto-complete.png](auto-complete.png)

## Docker on Windows

Can be a little tricky. Docker runs on Windows, not inside WSL. You install the docker client on WSL and then point it to the daemon running on Windows.

https://nickjanetakis.com/blog/setting-up-docker-for-windows-and-wsl-to-work-flawlessly

## Folder structure

There are some important folders for Kubernetes on Linux:

```sh
~/.ssh
    - id_rsa # SSH'ing into stuff, this is where your key is
~/.kube
    - config # kubectl config
~/.aws # AWS CLI
    - config
    - credentials
/usr/local/bin # Where you store your binaries
    - kubectl
    - kubens
    - kubectx
    - helm
```

# Non-core tools

## eksctl

There are many ways to spin up a cluster, currently I've settled on https://eksctl.io/, a CLI to create & maintain a cluster on EKS.

Download it, you'll see it works the same way as `kubectl`

## kubectx & kubens

When you're Kubernetes'ing, you need to switch to different clusters and namespaces. You could do this:

```sh
$ kubectl get pods -n production
```

But a better way is two great tools:

- [`kubectx`](https://github.com/ahmetb/kubectx/blob/master/kubectx)
- [`kubens`](https://github.com/ahmetb/kubectx/blob/master/kubens)

These are just shell scripts (think bat files). Simply move them to your `/usr/local/bin` folder, give them execute permissions and your ready to switch contexts and namespaces with ease:

```
$ kubectx cluster-1
$ kubens production
```

## stern

You can use `kubectl logs POD` to see the logs of a pod. But if you have 10 pods with your API running, you won't know which of the 10 handles a request you want to debug.  [Stern](https://github.com/wercker/stern) allows you to quickly log all pods with a SQL LIKE feel:

```sh
# Replace this
$ kubectl logs my-api-3fj3078
$ kubectl logs my-api-739fjal
$ kubectl logs my-api-1fj2309
$ kubectl logs my-api-234aoe8

# With this
$ stern my-api
```

## helm

Coming soon...

## Terminal editors

Coming soon!

- Nano
- Vim

## VS Code Kubernetes Extension

Coming soon!

## Tricks

Coming soon!