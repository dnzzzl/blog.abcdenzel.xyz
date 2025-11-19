+++
date = '2025-11-16T14:00:00-04:00'
draft = true 
title = 'Infrastructure as Code: From Zero to Deployed'
tags = ['terraform', 'ansible', 'iac', 'devops', 'automation']
categories = ['tech']
+++

I wanted to build a workflow that lets me go from nothing to a deployed application with one command. No clicking through cloud consoles, no manual SSH configuration, no "works on my machine" problems.

This is where I came across Infrastructure as Code (IaC), another lost cousin of SaaS and PaaS that will actually come in useful for coding: It is treating servers and configuration the same way we treat application code: version controlled, reproducible, and automated.

Here's how I set it up, the decisions I made, and what actually worked.

## The Goal

Build a system that:

1. Provisions infrastructure (VMs, networks, DNS)
2. Configures that infrastructure (installs software, sets up services)
3. Deploys applications to it
4. Can be torn down and rebuilt identically

I started with local VMs for development, Will extend to cloud production later.

**In scope:**

- Local virtual machines (Vagrant)
- Infrastructure provisioning (Terraform)
- Configuration management (Ansible)
- Deploying a .NET Core application
- Deploying static sites (Hugo, React)
- DNS registration and management

**Out of scope (for now):**

- CI/CD pipelines (that's phase 2)
- Ongoing updates and maintenance
- Monitoring and alerting

## Tool Selection

### Terraform: Infrastructure Provisioning

Terraform handles the "create servers" part. You write configuration files describing what infrastructure you want, and Terraform makes it happen.

**Why Terraform:**

- Works with multiple providers (local VMs via Vagrant, AWS, Azure, GCP, etc.)
- Declarative—you describe the end state, not the steps to get there
- Good state management (keeps track of what exists)
- Extensible with modules

**Alternative considered:** Vagrant alone for local, cloud provider CLIs for production. Rejected because too much provider-specific knowledge required.

### Ansible: Configuration Management

Ansible handles the "configure servers" part. Once Terraform creates a VM, Ansible installs software, configures services, and deploys applications.

**Why Ansible:**

- Agentless—uses SSH, no need to install anything on target machines
- YAML-based playbooks (readable, version-controllable)
- Idempotent—running the same playbook twice produces the same result
- Large ecosystem of modules for common tasks

**Alternative considered:** Shell scripts. Rejected because they're hard to make idempotent and harder to maintain.

### Vagrant: Local Development

Vagrant wraps VirtualBox (or other hypervisors) to manage local VMs consistently.

**Why Vagrant:**

- Simple Vagrantfile configuration
- Integrates well with Terraform and Ansible
- Snapshots and easy teardown
- Consistent environments across team members

## Architecture: Phased Approach

Rather than trying to build everything at once, I broke it into phases:

**Phase 1: Local Development**

- Vagrant + Ansible
- Spin up local VMs
- Deploy to localhost for testing

**Phase 2: Staging with Internet Access**

- Cloudflare Tunnels for public access without opening ports
- Test with real domains before production
- Distributed DNS for service discovery

**Phase 3: Production Cloud**

- Terraform modules for AWS/Azure/GCP
- Real infrastructure, real costs
- Same Ansible playbooks as Phase 1/2

This way I can develop and test locally before spending money on cloud resources.

## Project Structure

```
iac-project/
├── terraform/
│   ├── local/           # Vagrant provider config
│   ├── aws/             # Future: AWS resources
│   └── modules/         # Reusable Terraform modules
├── ansible/
│   ├── inventory/       # Static inventory files
│   ├── playbooks/       # Deployment playbooks
│   └── roles/           # Reusable Ansible roles
├── apps/
│   └── dotnet-mvc/      # Example .NET application
└── .env.example         # Environment variable template
```

## Phase 1: Local Infrastructure

### Vagrantfile

Simple single-VM setup to start:

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/focal64"

  config.vm.define "web" do |web|
    web.vm.network "private_network", ip: "192.168.56.10"
    web.vm.hostname = "web.local"

    web.vm.provider "virtualbox" do |vb|
      vb.memory = "2048"
      vb.cpus = 2
    end
  end
end
```

This creates a single Ubuntu VM at `192.168.56.10`. Later you can add more VMs for a control plane, database, etc.

Spin it up:

```bash
cd terraform/local
vagrant up
```

### Ansible Inventory

Static inventory file pointing to our VM:

```yaml
# ansible/inventory/local.yml
all:
  hosts:
    web:
      ansible_host: 192.168.56.10
      ansible_user: vagrant
      ansible_ssh_private_key_file: ~/.vagrant.d/insecure_private_key
```

Test connectivity:

```bash
ansible all -i inventory/local.yml -m ping
```

### Deployment Playbook

Ansible playbook to deploy a .NET application:

```yaml
# ansible/playbooks/deploy-dotnet.yml
---
- name: Deploy .NET MVC Application
  hosts: web
  become: yes

  tasks:
    - name: Install .NET 8 runtime
      apt:
        name:
          - dotnet-sdk-8.0
        state: present
        update_cache: yes

    - name: Install Nginx
      apt:
        name: nginx
        state: present

    - name: Copy application files
      copy:
        src: ../../apps/dotnet-mvc/
        dest: /var/www/app/
        owner: www-data
        group: www-data

    - name: Configure systemd service
      template:
        src: ../templates/dotnet-app.service.j2
        dest: /etc/systemd/system/dotnet-app.service
      notify: restart app

    - name: Configure Nginx reverse proxy
      template:
        src: ../templates/nginx-app.conf.j2
        dest: /etc/nginx/sites-available/app
      notify: restart nginx

    - name: Enable Nginx site
      file:
        src: /etc/nginx/sites-available/app
        dest: /etc/nginx/sites-enabled/app
        state: link
      notify: restart nginx

  handlers:
    - name: restart app
      systemd:
        name: dotnet-app
        state: restarted
        enabled: yes
        daemon_reload: yes

    - name: restart nginx
      systemd:
        name: nginx
        state: restarted
```

Run the playbook:

```bash
cd ansible
ansible-playbook -i inventory/local.yml playbooks/deploy-dotnet.yml
```

Now your app is running on the VM, accessible at `http://192.168.56.10`.

## Secrets Management

Don't commit secrets to git. Use environment files:

```bash
# .env (git-ignored)
DB_PASSWORD=supersecret
API_KEY=your_key_here
```

Reference them in Ansible:

```yaml
- name: Configure database connection
  template:
    src: appsettings.json.j2
    dest: /var/www/app/appsettings.json
  vars:
    db_password: "{{ lookup('env', 'DB_PASSWORD') }}"
```

Load the env file before running Ansible:

```bash
source .env
ansible-playbook ...
```

## Terraform State Management

Terraform tracks infrastructure state in a `.tfstate` file. This file contains sensitive info and shouldn't be committed.

For local development:

```
# .gitignore
*.tfstate
*.tfstate.backup
.terraform/
```

For team collaboration, use remote state (S3, Terraform Cloud, etc.), but that's phase 3.

## DNS Decisions

Two options for dynamic DNS:

**Option 1: Per-deployment domain**

- Each deployment gets its own domain (e.g., `project1.example.com`, `project2.example.com`)
- Requires DNS provider API access
- More flexible but more complex

**Option 2: Subdomain pattern**

- Single domain, multiple subdomains (e.g., `*.example.com → different services`)
- Simpler DNS setup
- Less flexible

I went with Option 2 for now—simpler to start, can always migrate later.

## What Actually Works

After building this out, here's what I learned:

**1. Start simple, expand later**
One VM, one app, basic Ansible. Get that working before adding complexity.

**2. Idempotency matters**
Ansible playbooks should be runnable multiple times without breaking things. Use `state: present`, not commands that assume current state.

**3. Separate provisioning from configuration**
Terraform creates the infrastructure. Ansible configures it. Don't try to do both in one tool.

**4. Test locally first**
Vagrant VMs are free and fast. Don't debug Terraform issues on AWS where every run costs money and takes 10 minutes.

**5. Version control everything except secrets**
Infrastructure code is code. Treat it like code: git, branches, pull requests.

## Next Steps

Phase 1 is done. Next up:

- **Phase 2**: Add Cloudflare Tunnels for public access without port forwarding
- **Phase 3**: Extend Terraform modules to support AWS
- **CI/CD**: Automate deployment on git push
- **Monitoring**: Add Prometheus + Grafana

But for now, I can go from zero to a deployed .NET application in about 5 minutes with three commands:

```bash
vagrant up
source .env
ansible-playbook -i inventory/local.yml playbooks/deploy-dotnet.yml
```

That's the point of Infrastructure as Code.

## Resources

- [Terraform Documentation](https://www.terraform.io/docs)
- [Ansible Documentation](https://docs.ansible.com/)
- [Vagrant Documentation](https://www.vagrantup.com/docs)
- [The Phoenix Project](https://www.goodreads.com/book/show/17255186-the-phoenix-project) - DevOps as a story

---

If you're still deploying by SSH'ing into servers and running commands manually, stop. Future you will thank present you for automating this.
