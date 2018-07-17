# edmondscommerce-projectSetup (Ansible Role)

## Description

This is used to carry out various tasks related to setting up a project. At the moment it can do the following

 * Create the directory for the project
 * Clone down the repo
 * Run composer
 * Create an env file
 * Run doctrine generation commands
 * Configure nginx for the project
    * Create a basic nginx conf file
    * Add SSL support
    * Add basic auth
 * Set the file permissions

This will be updated with more features as required

## Role Variables

| Name                            | Default Value                                                     | Description                                                                                          |
| ------------------------------- | ----------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| users_name                      | `ansibleUser`                                                     | The name of the user to be created and configured                                                    |
| users_group                     | `ansibleUser`                                                     | The group the the user will use                                                                      |
| project_name                    | `my-ansible-project`                                              | The name of the project that is being configured                                                     |
| project_directory               | `/var/www/vhosts`                                                 | The path to the directory where the project will be installed                                        |
| project_path                    | `"{{ project_directory }}/{{ project_name }}"`                    | The full path where the project will be installed                                                    |
| nuke_current_project            | `true`                                                            | If the project should be removed if it already exists                                                |
| uses_git                        | `true`                                                            | If the project uses git. If true then the next two vars must be set                                  |
| git_repo                        | ``                                                                | The URL to the repo. Used to to clone it down                                                        |
| git_branch                      | `master`                                                          | The branch that will be checked out                                                                  |
| uses_composer                   | `true`                                                            | If the project uses composer                                                                         |
| composer_action                 | `install`                                                         | The command composer will use. Should be install or update                                           |
| file_permissions                | `644`                                                             | The permissions that will be set for every file in the project                                       |
| directory_permission            | `755`                                                             | The permissions that will be set for every directory in the project                                  |
| uses_env_file                   | `true`                                                            | If an .env file should be created                                                                    |
| env_template                    | `./templates/.env.j2`                                             | The path to the env file template                                                                    |
| uses_doctrine                   | `true`                                                            | If the project uses doctrine, if true various doctrine build commands will be triggered              |
| config_nginx_for_domain         | `true`                                                            | Should we build an nginx configuration file for the project                                          |
| vhost_template                  | `./templates/etc/nginx/conf.d/port80.j2`                          | The template for the non SSL Nginx config                                                            |
| secure_vhost_template           | `./templates/etc/nginx/conf.d/port443.j2`                         | The template for the SSL Nginx config                                                                |
| domain_name                     | `www.example.com`                                                 | The domain that is being used, also used in the name of the conf file                                |
| additional_domain_names         | `''`                                                              | Any other names that should be listened for, i.e. no www                                             |
| web_root                        | `"{{ project_path }}"`                                            | The root folder that nginx will use                                                                  |
| port_to_listen_on               | `80`                                                              | The port that the non SSL config will listen on                                                      |
| enable_ssl                      | `false`                                                           | Do we want to enable SSL for the domain                                                              |
| certificate_file                | `"/etc/ssl/nginx/{{ domain_name }}.crt"`                          | Where the SSL Certificate is stored                                                                  |
| key_file                        | `"/etc/ssl/nginx/{{ domain_name }}.key"`                          | Where the SSL Key is stored                                                                          |
| local_certificate_file          | `''`                                                              | The path to the local version of the SSL certificate file                                            |
| local_key_file                  | `''`                                                              | The path to the local version of the SSL certificate key file                                        |
| ssl_port_to_listen_on           | `443`                                                             | The port that the SSL config will listen on                                                          |
| force_http_to_https             | `true`                                                            | Should all non SSL traffic be redirected to HTTPS                                                    |
| enable_basic_auth               | `true`                                                            | Should we setup basic auth for the site                                                              |
| basic_auth_name                 | `"Restricted Area"`                                               | The title of the auth area                                                                           |
| basic_auth_file                 | `/etc/nginx/.htpasswd`                                            | Where the basic auth file should be stored. It will be generated automatically                       |
| basic_auth_username             | `nginx`                                                           | The basic auth Username                                                                              |
| basic_auth_password             | `changeMe`                                                        | The basic auth Password                                                                              |
| ip_restrict                     | `false`                                                           | Set to an array of IP address to only allow to access the site from them                             |
| fastcgi_params                  | `""`                                                              | An array of objects made of of `name` and `value` nodes. Used to set PHP env vars                    |
| using_cloudflare                | `false`                                                           | Should nginx be configured to use real IP address when passed through Cloudflare                     |
| enable_letsencrypt              | `false`                                                           | Should a SSL Certificate be generated using letsencrypt. Do not set this and enable_ssl both to true |
| letsencrypt_email               | `''`                                                              | The email address to use with letsencrypt                                                            |
| use_ssl_client_cert             | `false`                                                           | Should the site be protected using SSL Client Certificates                                           |
| ssl_client_cert_path            | `"/etc/ssl/intermediate/{{ project_name }}/certs/ca-chain.cert"`  | Where the client certificate will live on the sever                                                  |
| local_ssl_client_cert_file      | `''`                                                              | The path the the local version of the SSL client certificate                                         |

## SSL Certificates

There are two ways of getting the project to use SSL, uploading a certificate and key, or getting letsencrypt to generate them.

__Do not set both variables to true__ if this is done then we will configure nginx to use the uploaded ones

## Config templates

The nginx config is generated using templates for [ssl config](./templates/etc/nginx/conf.d/port443.j2) and [non ssl config](./templates/etc/nginx/conf.d/port80.j2).

These both extend off a [base template](./templates/etc/nginx/conf.d/baseServerBlock.j2) which allows almost any part of the configuration to be overwritten for a project
