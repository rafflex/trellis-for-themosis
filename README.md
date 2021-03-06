# Trellis for Themosis [WORK IN PROGRESS]


Ansible playbooks for setting up a LEMP stack for WordPress.

- Local development environment with Vagrant
- High-performance production servers
- Zero-downtime deploys for your [Themosis](https://framework.themosis.com/)-based WordPress sites
- [trellis-cli](https://github.com/roots/trellis-cli) for easier management

## What's included

Trellis will configure a server with the following and more:

* Ubuntu 18.04 Bionic LTS
* Nginx (with optional FastCGI micro-caching)
* PHP 7.3
* MariaDB (a drop-in MySQL replacement)
* SSL support (scores an A+ on the [Qualys SSL Labs Test](https://www.ssllabs.com/ssltest/))
* Let's Encrypt for free SSL certificates
* HTTP/2 support (requires SSL)
* Composer
* WP-CLI
* SMTP (mail delivery)
* MailHog
* Memcached
* Fail2ban and ferm

## What's different
* It's prepared to work with Themosis Framework, NOT Bedrock
* Option to manage database and uploads migration (based on [this package](https://github.com/valentinocossar/trellis-database-uploads-migration))
* Option to manage plugins folder migration
* Option to manage mu-plugins folder migration

* Option to manage private composer packages migration
* Package [vagrant-trellis-cert](https://github.com/TypistTech/vagrant-trellis-cert) installed as default
* You need to set vagrant machine name in vagrant_default.yml

## To-Do List
* Incorporate creating of ssl cert into main playbook
* Generating APP_KEY variable during installation


## Documentation

Full documentation is available at [https://roots.io/trellis/docs/](https://roots.io/trellis/docs/).

## Requirements

Make sure all dependencies have been installed before moving on:

* [Virtualbox](https://www.virtualbox.org/wiki/Downloads) >= 4.3.10
* [Vagrant](https://www.vagrantup.com/downloads.html) >= 2.1.0
* **Recommended**: [trellis-cli](https://github.com/roots/trellis-cli)

**Windows user?** [Read the Windows getting started docs](https://roots.io/getting-started/docs/windows-development-environment-trellis/) for slightly different installation instructions.

## Installation

The recommended directory structure for a Trellis project looks like:

```shell
example.com/      # → Root folder for the project
├── trellis/      # → Your clone of this repository
└── site/         # → A Bedrock-based WordPress site
    └── htdocs/
        ├── content/  # → WordPress content directory (themes, plugins, etc.)
        └── cms/   # → WordPress core (don't touch!)
```
<br>

Create a new project directory:
```shell
$ mkdir example.com && cd example.com
```
Install Trellis:
```shell
$ git clone --depth=1 git@github.com:rafflex/trellis-for-themosis.git trellis && rm -rf trellis/.git
```
Install Themosis into the `site` directory:
```shell
$ composer create-project themosis/themosis site
```

[Read the Themosis Framework installation docs](https://framework.themosis.com/docs/2.0/installation) for more information.

## Local development setup

Configure your WordPress sites in
- `group_vars/local/wordpress_sites.yml`
- `group_vars/local/vault.yml`
- `group_vars/all/vault.yml`

Copy APP_KEY from `site/.env` file to `group_vars/all/vault.yml`

Ensure you're in the trellis directory: `cd trellis`

Spin up machine
```shell
vagrant up
```

When it's done run
```shell
./bin/post-install-1.sh
```
and after that
```shell
./bin/post-install-2.sh
```

[Read the local development docs](https://roots.io/trellis/docs/local-development-setup/) for more information.

## Remote server setup (staging/production)

A base Ubuntu 18.04 (Bionic) server is required for setting up remote servers.

Configure your WordPress sites in
- `group_vars/<environment>/wordpress_sites.yml`
- `group_vars/<environment>/vault.yml` (see the [Vault docs](https://roots.io/trellis/docs/vault/) for how to encrypt files containing passwords)

Add your server IP/hostnames to `hosts/<environment>`

Specify public SSH keys for `users` in `group_vars/all/users.yml` (see the [SSH Keys docs](https://roots.io/trellis/docs/ssh-keys/))

For remote servers, installing Ansible locally is an additional requirement. See the [docs](https://roots.io/trellis/docs/remote-server-setup/#requirements) for more information.

Provision the server:
```bash
$ ansible-playbook server.yml -e env=<environment>
```

[Read the remote server docs](https://roots.io/trellis/docs/remote-server-setup/) for more information.

## Deploying to remote servers

1. Add the `repo` (Git URL) of your Bedrock WordPress project in the corresponding `group_vars/<environment>/wordpress_sites.yml` file
2. Set the `branch` you want to deploy (defaults to `master`)

Deploy a site:
```bash
$ ./bin/deploy.sh <environment> <site>
```

Rollback a deploy:
```bash
$ ansible-playbook rollback.yml -e "site=<site> env=<environment>"
```

[Read the deploys docs](https://roots.io/trellis/docs/deploys/) for more information.

## Manging Database

Push database to remote server:
```bash
$ ./bin/database.sh <environment> <site> <mode>
```
Available modes: `push`, `pull`, `backup`

## Manging Uploads folder

Push uploads to remote server:
```bash
$ ./bin/uploads.sh <environment> <site> <mode>
```
Available modes: `push`, `pull`

## Manging Plugins folder

Push plugins to remote server:
```bash
$ ./bin/plugins.sh <environment> <site> <mode>
```
Available modes: `push`, `pull`

## Manging MU-Plugins folder

Push mu-plugins to remote server:
```bash
$ ./bin/mu-plugins.sh <environment> <site> <mode>
```
Available modes: `push`, `pull`

## Manging private composer packages folder

Update lines 30, 31, 46, 47 in`private-composer.yml` file.

Push mu-plugins to remote server:
```bash
$ ./bin/private-composer.sh <environment> <site> <mode>
```
Available modes: `push`, `pull`
