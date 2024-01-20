# Introduction

Fork of [LMtx/ansible-lab-docker](https://github.com/LMtx/ansible-lab-docker)

# Quick start

## Prerequisite

Docker Engine (24.0.7) or Docker Desktop

## Clone repository

Clone this git repository:

`git clone https://github.com/LMtx/ansible-lab-docker.git`

## Build images and run containers

Enter **ansible** directory containing [docker-compose.yml](./ansible/docker-compose.yml) file.

Build docker images and run containers in the background (details defined in [docker-compose.yml](./ansible/docker-compose.yml)):

`docker compose up -d --build`

Connect to **master node**:

`docker exec -it master01 bash`

Verify if network connection is working between master and managed hosts:

`ping -c 2 host01`

Start an [SSH Agent](https://man.openbsd.org/ssh-agent) on **master node** to handle SSH keys protected by passphrase:

`ssh-agent bash`

Load private key into SSH Agent in order to allow establishing connections without entering key passphrase every time:

`ssh-add master_key`

    Enter passphrase for master_key:

As **passphrase** enter: `12345`

Default key passphrase can be changed in [ansible/master/Dockerfile](./ansible/master/Dockerfile)

## Ansible playbooks

Run a [sample ansible playbook](./ansible/master/ansible/ping_all.yml) that checks connection between master node and managed hosts:

`ansible-playbook -i inventory ping_all.yml`

Confirm _every_ new host for SSH connections:

    ECDSA key fingerprint is SHA256:HwEUUnBtOm9hVAR2PJflNdCVchSCzIlpOpqYlwp+w+w.
    Are you sure you want to continue connecting (yes/no)?

Type: `yes` (three times)

Install PHP on web **inventory group**:

In order to group managed hosts for easier maintenance you can use groups in ansible [inventory file](./ansible/master/ansible/inventory).

Run a [sample ansible playbook](./ansible/master/ansible/install_php.yml):

`ansible-playbook -i inventory install_php.yml`

## Copy data between local file system and containers

### Copy directory from container to local file system

`docker cp master01:/var/ans/ .`

### Copy directory from local file system to container:

`docker cp ./ans master01:/var/`

You can check usage executing:

`docker cp --help`

## Cleanup

After you are done with your experiments or want to destroy lab environment to bring new one execute following commands.

Stop containers:

`docker compose kill`

Remove containers:

`docker compose rm`

Remove volume:

`docker volume rm ansible_ansible_vol`

If you want you can remove Docker images (although that is not required to start new lab environment):

`docker rmi ansible_host ansible_master ansible_base`

# Tips

In order to share public SSH key between **master** and **host** containers I used Docker **volume** mounted to all containers:

[docker-compose.yml](./ansible/docker-compose.yml):

    [...]
    volumes:
      - ansible_vol:/var/ans
    [...]

Master container stores SSH key in that volume ([ansible/master/Dockerfile](./ansible/master/Dockerfile)):

    [...]
    WORKDIR /var/ans
    RUN ssh-keygen -t rsa -N 12345 -C "master key" -f master_key
    [...]

And host containers add SSH public key to authorized_keys file ([ansible/host/run.sh](./ansible/host/run.sh)) in order to allow connections from master:

    cat /var/ans/master_key.pub >> /root/.ssh/authorized_keys

**IMPORTANT:** this is valid setup for lab environment but for production deployment you have to distribute the public key other way.

# Troubleshooting

## Host containers stop after creation

Check that [ansible/hosts/run.sh](./ansible/host/run.sh) has proper end of line type - it **should be Linux/Unix (LF)** not Windows (CRLF). You can change end of line type using source code editor (like Notepad++ or Visual Studio Code); under Linux you can use `dos2unix` command.

## Other issue

Please open an [issue](https://github.com/LMtx/ansible-lab-docker/issues/new) and I'll try to help.
