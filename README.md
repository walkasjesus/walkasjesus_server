# server_configuration

This project will install &amp; configure all the server components which are required to serve the [Jesus Commandments App](https://github.com/Ikbengeenrobot/volto)

## Requirements

Thise Ansible role is made for the following setup:  

- Debian 10 (You should have a bare Debian 10 installation ready)
- Nginx as webserver
- uWSGI as Application server for serving the Python Django Framework 
- We assume that there is a Transparant proxy server in front of our Nginx installation (That one is supposed to offload the SSL traffic and forward the request to this Nginx server). However you can easily edit the Nginx configuration in this role and configure it to your own needs.
- In order to synchronize the latest version of the Jesus Commandments App, you need a private key to the user {{ git_user }} which is allowed to synchronize the repository. (Ex. place the private key in /home/jesuscommandments/.ssh/id_rsa)

## Installation & Configuration

We will Install & Configure:  

- Basic OS Hardening
- Nginx
- uWSGI
- Jesus Commandment App
  - Initial installation
  - Update to newer versions

## Mandatory Variables

1. Please provide a `secretkey` variable for your django project. We suggest to set this variable in a vault.
`secretkey: <secretkey of the project>`

2. Configure the domain names in a list in the variable `domain_names`:

```yaml
domain_names:
  - example.com
  - www.example.com
```

## Usage

To Update and install the Jesus Commandment Application itself, use the following variables:  

Will install all requirements
`install_requirements: false`
Will update the current database structure
`update_database: false`
Will import a statix CSV file. First time use only!
`fill_database: false`
Update translation files
`update_translation_files: false`
Auto translate all files
`auto_translate_files: false`

Will clone the configured origin repository to the {{ git_project_path }}
`git_clone: yes`
Will update the current git repository to the {{ git_project_path }}
`git_update: yes`
Will force a git clone, even if there is no up-to-date and clean local repository
`git_force: yes`

Configure the app_version you would like to give this.
`app_version: 'version_1.0'`
In some situations you would like to force a copy of the previous database to the current one.
`force_copy_database: false`

### Rollback

If you want to rollback to the previous version, take the following steps:

1. Point the symlink to the previous version

ln -sf {{ django_projects_directory }}/{{ django_project_name }}/<app_version_to_restore> {{ django_projects_directory }}/{{ django_project_name }}/current
So this will look by example:
`ln -sf /var/www/jesuscommandments/version_1.0 /var/www/jesuscommandments/current`

2. Determine if there are any changes made to the database and media files in the time between the previous version and the current version.

If there are any changes, copy the last active database to the current path, by example:  
`cp /var/www/jesuscommandments/version_1.0/volto_website/volto_website/db.sqlite3 /var/www/jesuscommandments/current/volto_website/volto_website/db.sqlite3`

And rsync the latest media files to the current path, by example:  
`rsync -av /var/www/jesuscommandments/version_1.0/volto_website/volto_website/media/ /var/www/jesuscommandments/current/volto_website/volto_website/media/`
