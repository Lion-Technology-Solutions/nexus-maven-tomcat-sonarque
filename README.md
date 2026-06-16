# Terraform Nexus Ansible Platform

This repository provisions and configures a DevOps tools platform in AWS:

- Sonatype Nexus Repository on its own EC2 server
- SonarQube with local PostgreSQL on its own EC2 server
- Apache Tomcat and Maven on their own EC2 server

Terraform creates the AWS infrastructure in a dedicated VPC and writes a dynamic Ansible inventory. Ansible then installs and configures each service using role-based playbooks.

## Architecture

- Region: `us-east-1`
- SSH key pair: `rancher0529`
- Dedicated VPC with public subnet
- Security groups:
  - SSH `22`
  - Nexus `8081`
  - SonarQube `9000`
  - Tomcat `8080`
- Ubuntu 24.04 LTS EC2 instances

## Deploy Infrastructure

```bash
cd terraform
terraform init
terraform apply
```

Terraform writes the generated inventory to:

```text
../ansible/inventory/hosts.ini
```

## Configure Servers

```bash
cd ../ansible
ansible-playbook playbook.yml
```

If a previous run created a root-owned Ansible temp directory and you see an unreachable error for `/tmp/.ansible/tmp`, rerun after pulling the latest code. The project now uses `/tmp/ansible-ubuntu/tmp`. If the host still has a locked temp path, remove the old directory once:

```bash
ssh -i ~/.ssh/rancher0529.pem ubuntu@SERVER_PUBLIC_IP 'sudo rm -rf /tmp/.ansible'
```

## Default URLs

Terraform prints service URLs after apply:

- Nexus: `http://NEXUS_PUBLIC_IP:8081`
- SonarQube: `http://SONAR_PUBLIC_IP:9000`
- Tomcat: `http://TOMCAT_PUBLIC_IP:8080`

## Important Production Notes

- Restrict `ssh_allowed_cidr` and `service_allowed_cidr` in `terraform.tfvars`.
- Change `sonarqube_db_password` and `tomcat_admin_password` in Ansible group vars.
- Nexus first admin password is created by Nexus under `/opt/sonatype-work/nexus3/admin.password`.
- SonarQube default web login is `admin/admin` on first login unless changed through the UI.

## Destroy

```bash
cd terraform
terraform destroy
```
