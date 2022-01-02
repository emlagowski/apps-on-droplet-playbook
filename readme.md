# Apps on droplet playbook

This repository contains Ansible playbook to 
- create and prepare Digital Ocean droplet for running applications
- run reverse-proxy (nginx) on created droplet with companion container for certificate generation (ssl)
- run applications based on docker-compose files from project repositories

## Requirements

To run this playbook you need to prepare `secrets.yaml` file. You will need:

1. Digital Ocean API token ([link](https://cloud.digitalocean.com/account/api/tokens?i=9d16eb)) - for creating droplet and firewall rules.
1. GitHub personal token with `read:packages` permission ([link](https://github.com/settings/tokens/new)) - for pulling docker images from GitHub Container Registry (GCR).
1. GitHub personal token with `write:publik_key` permission ([link](https://github.com/settings/tokens/new)) - for cloning repositories into your droplet in the future.

File with above values should looks like example below:

```yaml
# digital ocean API token
oauth_token: <do_token>
# github personal token for read:packages - to login into dokcer registry
gh_gcr_token_ro: <gh_read_packages_token>
# github personal token for write:public_key - to add SSH key to allow to clone repositories
gh_api_key: <gh_write_public_key_token>
```

## Roles

This playbook contains several roles described below.

### Create Droplet

Create droplet on Digital Ocean from scratch.

1. Add your local SSH public key to Digital Ocean
1. Create a new Droplet and assign it into existing project
1. Get information about droplet
1. Show Droplet info
1. Create a Firewall for created droplet (inbound & outband rules)
1. Add droplet to hosts

### Prepare Droplet

Preapre created Droplet with installing all neccessary packages on it.

1. Install aptitude using apt & update cache
1. Prepare droplet for running docker containers
    1. Install required system packages for Docker
    1. Add Docker GPG apt Key
    1. Add Docker Repository
    1. Update apt and install docker-ce 
    1. Update apt and install docker-compose
1. Prepare droplet for pulling dokcer images
    1. Log into docker registry
1. Prepare droplet for cloning repositories
    1. Generate SSH key
    1. Read SSH public key to authorize
    1. Add generated SSH key to GitHub account

### Initialize nginx

This role runs on droplet nginx as reverse proxy along with companion container responsible for automatical certificates generation for all applications.

### Run Project

This role clones docker-compose from project repository and runs it on created droplet.

## Usage

Run command below

```sh
ansible-playbook play_all.yaml
```

Expected output should looks like one below

```sh
PLAY RECAP ***********************************
**.**.**.**                : ok=26   changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
localhost                  : ok=7    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

### How to add another application?

For adding another application to this playbook just duplcate usage of `run-project` role. Remember to adjust variables like:
- `git_repo_ssh` - application repo with docker-compose file
- `repo_dest` - destination directory for cloned git repository (must be unique)
- `domain` - domain which you want to be assigned to your application
- `project_name` - digital ocean project name

Example:

```yaml
- name: Run example-project with public domain
  hosts: droplet
  vars_files:
    - secrets.yaml
  roles:
  - role: run-project
    git_repo_ssh: git@github.com:emlagowski/droplet-example-project.git
    repo_dest: repos/droplet-example-project
    domain: test2.mysite.com
    project_name: all-in-one
  environment:
    DOMAINS: test.mysite.com,www.test.mysite.com
```
