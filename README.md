# jesus_commandments_server
This repository contains IT Automation tools for (Ansible)[https://docs.ansible.com/ansible/latest/index.html] to install & configure all the server components which are required to serve the [Jesus Commandments Framework] (https://github.com/jesuscommandments/jesus_commandments_framework)

# Requirements

Thise Ansible role is made for the following setup:  

- Debian 10 (You should have a bare Debian 10 installation ready)
- Nginx as webserver
- uWSGI as Application server for serving the Python Django Framework 
- We assume that there is a Transparant proxy server in front of our Nginx installation (That one is supposed to offload the SSL traffic and forward the request to this Nginx server). However you can easily edit the Nginx configuration in this role and configure it to your own needs.
- In order to synchronize the latest version of the Jesus Commandments App, you need a private key to the user {{ git_user }} which is allowed to synchronize the repository. (Ex. place the private key in /home/jesuscommandments/.ssh/id_rsa)

# Installation & Configuration

We will Install & Configure:  

- Basic OS Hardening
- Nginx
- uWSGI
- Mariadb
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

4. Set the path to the HSV bible texts on the ansible server
`jc_hsv_bible_path_temp: "{{ playbook_dir }}/group_vars/files/hsv_bible.zip"`

5. Set the API key variable for the scripture.api.bible. You need to get your license @api.bible.
# The API key to scripture.api.bible
`jc_bible_api_key: xxxxxxx`

6. Set the key for the HSV bible:
# The key for the HSV bible
`jc_hsv_bible_key: xxxxxxx`

7. Set the users who get access to the protected awstats user statistics on `https://<domain>/awstats`
```yaml
jc_developer_users:
  - { user: 'username1', password: 'password1' }
```

## Usage

To Update and install the Jesus Commandment Application itself, use the following variables:  

Will install all requirements  
`jc_install_requirements: false`
Will update the current database structure
`jc_update_database: false`
Will create the admin user for the django application
`jc_create_admin_user: false`
Will import a static CSV file.
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


### Configuring the ansible server
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

4. Configure some group_vars

Set some global group_vars variables which will apply for acc and prod
`group_vars/jesuscommandments/main.yml`
```yaml
jc_hsv_bible_path_temp: "{{ playbook_dir }}/group_vars/jesuscommandments/files/hsv_bible.zip"
jc_media_files: "{{ playbook_dir }}/group_vars/jesuscommandments/media"
```

Set some global group_vars vault variables which will apply for acc and prod
`group_vars/jesuscommandments/vault.yml`
```yaml
# The API key to scripture.api.bible
jc_bible_api_key: xxxxxxxxx 
# The key for the HSV bible
jc_hsv_bible_key: xxxxxxxxx
```

Encrypt the vault:
`ansible-vault encrypt group_vars/jesuscommandments/vault.ym`

5. Configure some host_vars

Set some host specific host_vars variables which will apply for this host only
`host_vars/jc-acc/main.yml`
```yaml
jc_domain_names:
  - acc.jesuscommandments.org

jc_production: False

jc_settings_debug: True

jc_developer_ips:
  - 1.1.1.1
  - 2.2.2.2

# Superuser settings
jc_superuser_username: admin
jc_superuser_email: info@jesuscommandments.org
```

Set some host specific vault variables which will apply for this host only
`host_vars/jc-acc/vault.yml`
```yaml
jc_secretkey: xxxxx

jc_superuser_password: xxxxx

mysql_user_password: xxxxx
mysql_root_password: xxxxx
mysql_user_jc_password: xxxxx

jc_developer_users: 
  - { username: 'user1', password: 'xxxxx' }
  - { username: 'user2', password: 'xxxxx' }
```

Encrypt the vault:
`ansible-vault encrypt group_vars/jesuscommandments/vault.ym`

# Related projects and repositories
The following projects are related to this repository.

## jesus_commandments_resources
Repository for the [Jesus Commandments Framework](https://github.com/jesuscommandments/jesus_commandments_framework) where all the resources (movies, songs, blogs, sermons, testimonies, etc) in all languages are stored in a CSV. This CSV can be imported/exported with the [Jesus Commandments Framework](https://github.com/jesuscommandments/jesus_commandments_framework) and the (Jesus Commandments Server) (https://github.com/jesuscommandments/jesus_commandments_server)

## jesus_commandments_framework
This repository is the heart of the application which will show all the different components of the Jesus Commandments Application in a fancy Python Framework (Django)[https://www.djangoproject.com/]

## jesus_commandments_biblereferences
Repository for the Jesus Commandments Framework where all the commandments with all their related Bible References are stored in a CSV. This CSV can be imported/exported with the [Jesus Commandments Framework](https://github.com/jesuscommandments/jesus_commandments_framework) and the [Jesus Commandments Server] (https://github.com/jesuscommandments/jesus_commandments_server)

