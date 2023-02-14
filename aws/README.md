# Testnets on AWS

A testnet can be launched on AWS, where each node will run on an EC2 instance in a VPC.

The VPC and other infrastructure should have been setup in advance, using our [testnet-infra repo](https://github.com/maidsafe/terraform-testnet-infra). This process will use a basic Terraform configuration to launch the EC2 instances on the VPC, then use Ansible to provision them with the node setup.

## Setup

If you want to run this process on Windows, you will need to use WSL.

Installations of the following tools are required on your platform:

* Terraform
* Ansible (installing with `pip` in a virtualenv is recommended)
* [Just](https://github.com/casey/just) (a modern Makefile alternative)
* AWS CLI
* jq

Obtain the AWS access and secret access keys for the `testnet_runner` account, and also the password for the Ansible vault. Put the Ansible password in a file.

Create a .env at the same level where this directory is, and fill it with the following:
```
AWS_ACCESS_KEY_ID=<testnet-runner-access-key>
AWS_SECRET_ACCESS_KEY=<testnet-runner-secret-key>
AWS_DEFAULT_REGION=eu-west-2
ANSIBLE_VAULT_PASSWORD_PATH=<path>
SSH_PRIVATE_KEY_PATH=<path>
SSH_PUBLIC_KEY_PATH=<path>
SN_TESTNET_DEV_SUBNET_ID=subnet-038968af46e82a7c9
SN_TESTNET_DEV_SECURITY_GROUP_ID=sg-0ad9341d486c81e38
```

The EC2 instances need to be launched with an SSH key pair. You can either generate a new key pair or use an existing one. In either case, set `SSH_PUBLIC_KEY_PATH` to the location of the public key. The `SSH_PRIVATE_KEY_PATH` should be set to the corresponding private key, since Ansible will use this. Similarly, set `ANSIBLE_VAULT_PASSWORD_PATH` to the location where you put the password file.

## Create a Testnet

Run `just init <name>`, where `name` will be the name of the testnet. The name should be a short word, e.g., "alpha" or "beta", or your first name (though the name "dev" cannot be used, because that's the main workspace that cannot be deleted). This will create a Terraform workspace, a key pair on EC2, and Ansible inventory files.

Run `just testnet <name> <node_count>`, e.g., `just testnet alpha 30`.

If you want to use a local node binary, use `just testnet alpha 30 "<path>"`, where `<path>` should be a full, absolute path. Otherwise, the latest binary will be pulled from S3.

Terraform will run to create the instances then Ansible will be used to provision them.

## Working with a Testnet

There are various utility targets that can be called:

* `just ssh-details`: will print out a list of all the nodes and their public IP addresses, which you can then use to SSH to any node, using your private key and the `ubuntu` user.
* `just logs alpha`: will get the logs from all the machines in the testnet and make them available in a `logs` directory locally.
* `just network-contacts alpha`: will copy the network contacts file from the genesis node to the local machine.

## Teardown

When you're finished with the testnet, run `just clean <name>`. This will destroy the EC2 instances, delete the Terraform workspace, remove the key pair on EC2 and delete the Ansible inventory files.