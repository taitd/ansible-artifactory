artifactory
===========

Ansible role which helps to install and configure Artifactory.

The configuration of the role is done in such way that it should not be
necessary to change the role for any kind of configuration. All can be
done either by changing role parameters or by declaring completely new
configuration as a variable. That makes this role absolutely
universal. See the examples below for more details.

Please report any issues or send PR.


Examples
--------

```
---

- name: Example of how to install Artifactory OSS with default settings
  hosts: all
  roles:
    - artifactory

- name: Example of how to install Artifactory PRO with default settings
  hosts: all
  vars:
    # When license is specified, it will automatically use PRO version
    artifactory_license: "{{ lookup('file', '/path/to/artifactory.lic') }}"
  roles:
    - artifactory

- name: Example of how to install Artifactory with PostgreSQL DB
  hosts: all
  vars:
    # Specify DB engine to be PostgreSQL
    artifactory_db_engine: postgresql
  roles:
    - postgresql
    - artifactory

- name: Example of how to install Artifactory with MySQL DB
  hosts: all
  vars:
    # Specify DB engine to be MySQL
    artifactory_db_engine: mysql
  roles:
    - mysql
    - artifactory
```


Role variables
--------------

```
# YUM repo URL
artifactory_yumrepo_url: "{{
  'http://jfrog.bintray.com/artifactory-pro-rpms'
    if artifactory_license != None
    else
  'http://jfrog.bintray.com/artifactory-rpms' }}"

# Additional YUM repo pararms
artifactory_yumrepo_params: {}

# Package to be installed (explicit version can be specified here)
artifactory_pkg: "{{
  'jfrog-artifactory-pro'
    if artifactory_license != None
    else
  'jfrog-artifactory-oss' }}"

# License content for the PRO version
artifactory_license: null
# Examples
#artifactory_license: |
#  Fygu3RmK9savpgN/ZXb3QnDgi07+i4gY9D+eWJ6mVfd3vtUSirZKuaeLQjGZ77HPlc91ToaqEIKd
#  ...
#artifactory_license: "{{ lookup('file', '/path/to/the/lic/file') }}"

# Packages required for the PostgreSQL DB creation
artifactory_db_pgsql_pkgs:
    - python-psycopg2
    - postgresql-jdbc

# Packages required for the MySQL DB creation
artifactory_db_mysql_pkgs:
    - MySQL-python
    - mysql-connector-java

# Default list of extra packages
artifactory_deps_pkgs__default:
  - java

# Custom list of extra packages
artifactory_deps_pkgs__custom: []

# Final list of extra packages
artifactory_deps_pkgs: "{{
  (
    artifactory_db_pgsql_pkgs
      if artifactory_db_engine == 'postgresql'
      else
    artifactory_db_mysql_pkgs
      if artifactory_db_engine == 'mysql'
      else
    []
  ) +
  artifactory_deps_pkgs__default +
  artifactory_deps_pkgs__custom }}"

# Name of the service
artifactory_service: artifactory

# Config directory
artifactory_conf_dir: /etc/opt/jfrog/artifactory/

# DB engine [derby|postgresql|mysql]
artifactory_db_engine: derby

# User used to create DB and user
artifactory_db_login_user: "{{
  'postgres'
    if artifactory_db_engine == 'postgresql'
    else
  'root' }}"

# Password used to create DB and user
artifactory_db_login_password: "{{
  'postgres'
     if artifactory_db_engine == 'postgresql'
     else
  None }}"

# Password used to create DB and user
artifactory_db_driver: "{{
  'org.postgresql.Driver'
     if artifactory_db_engine == 'postgresql'
     else
  'com.mysql.jdbc.Driver' }}"

# Location of the JDBC driver
artifactory_db_driver_location: "{{
  '/usr/share/java/postgresql-jdbc.jar'
     if artifactory_db_engine == 'postgresql'
     else
  '/usr/share/java/mysql-connector-java.jar' }}"

# Destination symlink for the JDBC driver
artifactory_db_driver_destination: /opt/jfrog/artifactory/tomcat/lib/{{ artifactory_db_engine }}-jdbc.jar

# DB server configuration options
artifactory_db_host: localhost
artifactory_db_port: "{{
  5432
    if artifactory_db_engine == 'postgresql'
    else
  3306 }}"
artifactory_db_name: artifactory
artifactory_db_user: artifactory
artifactory_db_password: artifactory

# DB property file
artifactory_db_conf:
  type: "{{ artifactory_db_engine }}"
  driver: "{{ artifactory_db_driver }}"
  url: jdbc:{{ artifactory_db_engine }}://{{ artifactory_db_host }}:{{ artifactory_db_port }}/{{ artifactory_db_name }}
  username: "{{ artifactory_db_user }}"
  password: "{{ artifactory_db_password }}"
```


Dependencies
------------

- [`config_encoder_filters`](https://github.com/jtyr/ansible-config_encoder_filters)
- [`postgresql`](https://github.com/jtyr/ansible-postgresql) (optional)
- [`oracle_java`](https://github.com/jtyr/ansible-oracle_java) (optional)


License
-------

MIT


Authors
-------

Dennis Tait, Jiri Tyr
