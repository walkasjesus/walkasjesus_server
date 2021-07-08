# jesus_commandments_server

This repository contains IT Automation tools for [Ansible](https://docs.ansible.com/ansible/latest/index.html) to install & configure all the server components which are required to serve the [Walk as Jesus Framework](https://github.com/walkasjesus/walkasjesus_framework)

## Requirements

Thise Ansible role is made for the following setup:  

- Debian 10 (You should have a bare Debian 10 installation ready)
- Nginx as webserver
- uWSGI as Application server for serving the Python Django Framework 
- We assume that there is a Transparant proxy server in front of our Nginx installation (That one is supposed to offload the SSL traffic and forward the request to this Nginx server). However you can easily edit the Nginx configuration in this role and configure it to your own needs.
- In order to synchronize the latest version of the Jesus Commandments App, you need a private key to the user {{ git_user }} which is allowed to synchronize the repository. (Ex. place the private key in /home/walkasjesus/.ssh/id_rsa)

## Installation & Configuration

We will Install & Configure:  

- Basic OS Hardening
- Nginx
- uWSGI
- Mariadb
- Jesus Commandment App
  - Initial installation
  - Update to newer versions

### Mandatory Variables

1. Please provide a `waj_secretkey` variable for your django project. We suggest to set this variable in a vault.
`waj_secretkey: <secretkey of the project>`

2. Configure the domain names in a list in the variable `waj_domain_names`:

```yaml
waj_domain_names:
  - example.com
  - www.example.com
```

3. Create a superuser by settings the following variables
```yaml
waj_superuser_username: <username>
waj_superuser_email: <email>
```

we suggest to set the password variable in a vault  
`waj_superuser_password: <password>`

4. Set the API key variable for the scripture.api.bible. You need to get your license @api.bible.
The API key to scripture.api.bible  
`waj_bible_api_key: xxxxxxx`

5. Set the users who get access to the protected awstats user statistics on `https://<domain>/awstats`
```yaml
waj_developer_users:
  - { user: 'username1', password: 'password1' }
```

### Usage

To Update and install the Jesus Commandment Application itself, use the following variables:  

Will install all requirements  
`waj_install_requirements: false`
Will update the current database structure
`waj_update_database: false`
Will create the admin user for the django application
`waj_create_admin_user: false`
Will import a static CSV file with all the Commandments and corresponding Bible References from its upstream Github repository.
`waj_import_commandments: false`
Will import a static CSV file with all the Media files from its upstream Github repository.
`waj_import_media: false`
Update translation files
`waj_update_translation_files: false`
Auto translate all files
`waj_auto_translate_files: false`
Clean Thumbnail Cache
`waj_clean_thumbnail_cache: false`

If you running a non-production environment, you can set the following variable to false
`waj_production: false`

Add your own WAN ip addresses to allow the developer toolbar on your machine for performance and debug statistics 
```yaml
waj_developer_ips:
  - 1.1.1.1
  - 2.2.2.2
```

Will clone the configured origin repository to the {{ waj_git_project_path }}
`waj_git_clone: yes`
What version of the repository to check out (default HEAD).  This can be the literal string `HEAD', a branch name, a tag name. 
`waj_git_version: <branchname|tagname>`
Will update the current git repository to the {{ waj_git_project_path }}
`waj_git_update: yes`
Will force a git clone, even if there is no up-to-date and clean local repository
`waj_git_force: yes`

Configure the waj_app_version you would like to give this.
`waj_app_version: 'version_1.0'`
In some situations you would like to force a copy of the previous database to the current one.
`waj_force_copy_database: false`

### Rollback

If you want to rollback to the previous version, take the following steps:

1. Point the symlink to the previous version

ln -sf {{ waj_projects_directory }}/{{ waj_project_name }}/<waj_app_version_to_restore> {{ waj_projects_directory }}/{{ waj_project_name }}/current
So this will look by example:
`ln -sf /var/www/jesuscommandments/version_1.0 /var/www/jesuscommandments/current`

2. Determine if there are any changes made to the database and media files in the time between the previous version and the current version.

If there are any changes, copy the last active database to the current path, by example:  
`cp /var/www/jesuscommandments/version_1.0/jesus_commandments_website/jesus_commandments_website/db.sqlite3 /var/www/jesuscommandments/current/jesus_commandments_website/jesus_commandments_website/db.sqlite3`

And rsync the latest media files to the current path, by example:  
`rsync -av /var/www/jesuscommandments/version_1.0/jesus_commandments_website/jesus_commandments_website/media/ /var/www/jesuscommandments/current/jesus_commandments_website/jesus_commandments_website/media/`

### Playbook Example


#### Configuring the ansible server
To configure the ansible server in order to install the JesusCommandments application you can take the following steps:

1. Install some required packages:
`apt-get install ansible`

2. Configure an inventory
`vi /etc/ansible/hosts`

```
[jesuscommandments]
jc-acc ansible_host=<internal ip>
jc-prod1 ansible_host=<internal ip>
```

3. Configure a playbook

En example would look something like this:
```yaml
---

- hosts: 
  - jesuscommandments
  become: true
  gather_facts: true

  vars:
    waj_install_requirements: true
    waj_update_database: true
    waj_import_commandments: true
    waj_import_media: true
    waj_update_translation_files: true
    waj_auto_translate_files: false
    waj_app_version: 'version_1.0'
    waj_force_copy_database: false
    waj_clean_thumbnail_cache: false
    waj_git_force: yes
    waj_git_clone: yes
    waj_git_update: yes
    waj_domain_names:
      - acc.jesuscommandments.org
    waj_production: False
    waj_settings_debug: True
    waj_developer_ips:
      # Developer1
      - 1.2.3.4
      # Developer2
      - 2.3.4.5
    waj_superuser_username: myadmin
    waj_superuser_email: info@jesuscommandments.org

    # These settings should be stored in a vault
    waj_secretkey: <random secret key for django>
    waj_superuser_password: somepassword1
    mysql_user_password: somepassword2
    mysql_root_password: somepassword3
    mysql_user_waj_password: somepassword4


  roles:
    - role: jesus_commandments_server
```

4. Configure some group_vars

Set some global group_vars variables which will apply for acc and prod
`group_vars/jesuscommandments/main.yml`
```yaml
waj_media_files: "{{ playbook_dir }}/group_vars/jesuscommandments/media"
```

Set some global group_vars vault variables which will apply for acc and prod
`group_vars/jesuscommandments/vault.yml`
```yaml
# The API key to scripture.api.bible
waj_bible_api_key: xxxxxxxxx 
```

Encrypt the vault:
`ansible-vault encrypt group_vars/jesuscommandments/vault.ym`

5. Configure some host_vars

Set some host specific host_vars variables which will apply for this host only
`host_vars/jc-acc/main.yml`
```yaml
waj_domain_names:
  - acc.jesuscommandments.org

waj_production: False

waj_settings_debug: True

waj_developer_ips:
  - 1.1.1.1
  - 2.2.2.2

# Superuser settings
waj_superuser_username: admin
waj_superuser_email: info@jesuscommandments.org
```

Set some host specific vault variables which will apply for this host only
`host_vars/jc-acc/vault.yml`
```yaml
waj_secretkey: xxxxx

waj_superuser_password: xxxxx

mysql_user_password: xxxxx
mysql_root_password: xxxxx
mysql_user_waj_password: xxxxx

waj_developer_users: 
  - { username: 'user1', password: 'xxxxx' }
  - { username: 'user2', password: 'xxxxx' }
```

Encrypt the vault:
`ansible-vault encrypt group_vars/walkasjesus/vault.ym`

## Related projects and repositories

The following projects are related to this repository.

### walkasjesus_framework

This repository is the heart of the application which will show all the different components of the Jesus Commandments Application in a fancy Python Framework [Django](https://www.djangoproject.com/)

### walkasjesus_biblereferences

Repository for the Walk as Jesus Framework where all the commandments with all their related Bible References are stored in a CSV. This CSV can be imported/exported with the [Walk as Jesus Framework](https://github.com/walkasjesus/walkasjesus_framework) and the [Walk as Jesus Server](https://github.com/walkasjesus/jesus_commandments_server)

### walkasjesus_media

Repository for the [Walk as Jesus Framework](https://github.com/walkasjesus/walkasjesus_framework) where all the resources (movies, songs, blogs, sermons, testimonies, etc) in all languages are stored in a CSV. This CSV can be imported/exported with the [Walk as Jesus Framework](https://github.com/walkasjesus/walkasjesus_framework) and the [Walk as Jesus Server](https://github.com/walkasjesus/jesus_commandments_server)

### walkasjesus_translations

This repository contains all translation files from English to other languages. We use Google translate to make a first automated translation. The next step for the translator is to review all translated items and acknowledge them through the admin panel of the website.
