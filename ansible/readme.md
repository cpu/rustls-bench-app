# Ansible scripts

This directory contains the necessary scripts for the bench server's setup and maintenance. They
allow deploying the bench server application, installing its dependencies, and exposing it to the
internet through HTTPS (including certificate renewal).

Important notes: 

* securing the server after setting up the operating system and creating users for server admins is not included.
* performing a deploy _will_ affect benchmarking results (for example due to compiling Rust code for `ci-bench-runner`).
  Try to avoid deploying while benchmarking is in progress, or re-run benchmarks afterwards to avoid noise.

## Requirements

### Ansible configuration

The ansible scripts assume you are able to login as a sudoer user through passwordless SSH. You will
also need to install some Ansible collections:

1. `ansible-galaxy collection install community.general`

By default, the playbook will be run against the official rustls bench server, as configured in
[`inventory.ini`](inventory.ini).

### App configuration and secrets

The provided playbook requires access to a few secrets which are not part of the git repository. You
will need to:

1. Create a copy of `secrets.example.yml` and rename it to `secrets.yml`.
1. Replace the placeholder secret values by real ones.

Secrets will be injected in the Rust app configuration file, which can be found at
[`roles/deploy/templates/config.json`](roles/deploy/templates/config.json). If you are deploying the
app for testing purposes, you will probably need to update some fields like `github_app_id`,
`github_repo_owner` and `github_repo_name` to match yours.

## General overview

The Rust application is deployed as a systemd unit and exposed to the internet through nginx as the
reverse proxy. Certificates are obtained from Let's Encrypt using certbot, which automatically takes
care of renewal. Rust itself is managed through rustup, because Debian's Rust version is otherwise
too old.

## Useful commands

- Setup everything from scratch and deploy: `ansible-playbook playbook.yml`
- Update Rust to the latest stable version: `ansible-playbook playbook.yml --tags rustup`
- Deploy a new version of the server: `ansible-playbook playbook.yml --tags deploy`
- Check the status of the application: `sudo systemctl status ci-bench-runner`

## Testing

To deploy to a test server, copy `inventory.ini` to a new file `test-inventory.ini`, and update 
the `ansible_host` IP address in `test-inventory.ini`.

You can then deploy using:
``bash
ansible-playbook \
  --inventory test-inventory.ini \
  --user root \
  --extra-vars 'hostname=rustls-bench.example.com' \
  --extra-vars 'github_repo_owner=example' \
  playbook.yml
``
`
