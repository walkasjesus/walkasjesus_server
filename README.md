# server_configuration

This project will install &amp; configure all the server components which are required to serve the [Jesus Commandments App](https://github.com/jesuscommandments/jesus_commandments_app)

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

1. Please provide a `jc_secretkey` variable for your django project. We suggest to set this variable in a vault.
`jc_secretkey: <secretkey of the project>`

2. Configure the domain names in a list in the variable `jc_domain_names`:

```yaml
jc_domain_names:
  - example.com
  - www.example.com
```

3. Create a superuser by settings the following variables
```yaml
jc_superuser_username: <username>
jc_superuser_email: <email>
```
#  we suggest to set the password variable in a vault
`jc_superuser_password: <password>`


## Usage

To Update and install the Jesus Commandment Application itself, use the following variables:  

Will install all requirements  
`jc_install_requirements: false`
Will update the current database structure
`jc_update_database: false`
Will import a statix CSV file. First time use only!
`jc_fill_database: false`
Update translation files
`jc_update_translation_files: false`
Auto translate all files
`jc_auto_translate_files: false`
Clean Thumbnail Cache
`jc_clean_thumbnail_cache: false`

If you running a non-production environment, you can set the following variable to false
`jc_production: false`

Add your own WAN ip addresses to allow the developer toolbar on your machine for performance and debug statistics 
```yaml
jc_developer_ips:
  - 1.1.1.1
  - 2.2.2.2
```

Will clone the configured origin repository to the {{ jc_git_project_path }}
`jc_git_clone: yes`
Will update the current git repository to the {{ jc_git_project_path }}
`jc_git_update: yes`
Will force a git clone, even if there is no up-to-date and clean local repository
`jc_git_force: yes`

Configure the jc_app_version you would like to give this.
`jc_app_version: 'version_1.0'`
In some situations you would like to force a copy of the previous database to the current one.
`jc_force_copy_database: false`

### Rollback

If you want to rollback to the previous version, take the following steps:

1. Point the symlink to the previous version

ln -sf {{ jc_projects_directory }}/{{ jc_project_name }}/<jc_app_version_to_restore> {{ jc_projects_directory }}/{{ jc_project_name }}/current
So this will look by example:
`ln -sf /var/www/jesuscommandments/version_1.0 /var/www/jesuscommandments/current`

2. Determine if there are any changes made to the database and media files in the time between the previous version and the current version.

If there are any changes, copy the last active database to the current path, by example:  
`cp /var/www/jesuscommandments/version_1.0/jesus_commandments_website/jesus_commandments_website/db.sqlite3 /var/www/jesuscommandments/current/jesus_commandments_website/jesus_commandments_website/db.sqlite3`

And rsync the latest media files to the current path, by example:  
`rsync -av /var/www/jesuscommandments/version_1.0/jesus_commandments_website/jesus_commandments_website/media/ /var/www/jesuscommandments/current/jesus_commandments_website/jesus_commandments_website/media/`

### Playbook Example

```yaml
---

- hosts: 
  - jesuscommandments
  become: true
  gather_facts: true

  vars:
    jc_install_requirements: true
    jc_update_database: true
    jc_fill_database: true
    jc_update_translation_files: true
    jc_auto_translate_files: false
    jc_app_version: 'version_1.0'
    jc_force_copy_database: false
    jc_clean_thumbnail_cache: false
    jc_git_force: yes
    jc_git_clone: yes
    jc_git_update: yes
    jc_domain_names:
      - acc.jesuscommandments.org
    jc_production: False
    jc_settings_debug: True
    jc_developer_ips:
      # Developer1
      - 1.2.3.4
      # Developer2
      - 2.3.4.5
    jc_superuser_username: myadmin
    jc_superuser_email: info@jesuscommandments.org

    # These settings should be stored in a vault
    jc_secretkey: <random secret key for django>
    jc_superuser_password: somepassword1
    mysql_user_password: somepassword2
    mysql_root_password: somepassword3
    mysql_user_jc_password: somepassword4


  roles:
    - role: jesus_commandments_server
```
