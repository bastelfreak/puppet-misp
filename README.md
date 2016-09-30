# MISP MODULE

#### Table of Contents

1. [Overview](#overview)
2. [Module Description - What the module does and why it is useful](#module-description)
3. [Setup - Getting started with the misp module](#setup)
    * [What misp affects](#what-misp-affects)
4. [Usage - Configuration options and additional functionality](#usage)
    * [Basic usage](#basic-usage)
    * [Another usage example](#another-usage-example)
5. [Parameters of the MISP Class](#parameters-of-the-misp-class)
    * [MISP installation](#misp-installation)
    * [Database configuration](#database-configuration)
    * [MISP configuration](#misp-configuration)
        * [Site configuration](#site-configuration)
        * [Security](#security)
        * [MISP](#misp)
        * [GPG](#gpg)
        * [SMIME](#smime)
        * [Proxy](#proxy)
        * [Secure Authentication](#secure-auth)
        * [Plugin](#plugin)
        * [Apache Shibboleth Authentication](#apacheshibbauth)          
    * [Services](#services)
    

This module installs and configures MISP (Malware Information Sharing Platform) on CentOS 7. 
It has been tested on Puppet 3.8.7 and with MISP versions 2.4.50 and 2.4.51.

## Module Description

This module installs and configures MISP on CentOS 7. It installs all the needed dependencies, configures MISP and 
starts the services. However it does not set up the database nor the GPG key, that is up to the administrator to do. 
In addition it does not set up the web server on top of which MISP would run, meaning that Apache, Nginx or another 
web server of your choice would be needed (nevertheless the module need to know to know the name of the service of the 
web server (e.g. httpd)).

As mentioned before the database would need to be set up, the schema imported and then a user with rights to access the 
'misp' database created. If GPG would be used, the GPG key would need to be created and placed in the configured 
directory (by default '/var/www/MISP/').

The module follows the installation instructions that can be found [here](https://github.com/MISP/MISP/tree/2.4/INSTALL). 
Also details about the database and GPG key creation and set up can be found there.

NOTE: the configuration and database files of MISP are used as templates on the module, therefore if the are major 
changes on the version of MISP the template might cause troubles and need to be updated.

## Setup

### What MISP affects

The MISP module will not alter any OS files, all the configuration will happen in '/config_dir/' (by default 
/install_dir/app/Config/') where the *core.php*, *bootstrap.php*, *database.php* and *config.php* files will be deployed 
with the established values.

This module needs the following packages:

    * gcc: Needed for compiling Python modules
    * git: Needed for pulling the MISP code and other git repositories which MISP depends on
    * zip, redis, haveged and maria db
    * python-devel, python-pip, python-lxml, python-dateutil, python-six, python-lm, importlib [pip], Crypt_GPG [pear]: Python related packages
    * rh-php56, rh-php56-php-fpm, rh-php56-php-devel, rh-php56-php-mysqlnd, rh-php56-php-mbstring, php-pecl-redis, php-pear: PHP 5.6 related packages
    * php-mbstring: Python package required by Crypt_GPG
    * libxslt-devel', 'zlib-devel

The services needed by MISP are: 

    * rh-php56-php-fpm
    * haveged
    * redis: This package installs the redis server, therefore it would only be installed if the 'redis' parameter is 
    set to true. 
    * The 4 workers and the scheduler [CakeResque]

## Usage

### Basic usage

In order to use the module it would be enough to include the module:
```puppet
include ::misp,
```

Or the class:
```puppet
class{ ::misp:}
```

And the module will use all parameters with default values, these values are specified later on.

### Another usage example

```puppet
class {'::misp':
    git_tag          => 'v2.4.52',
    email            => 'someone.someother@somewhere.ch',
    contact          => 'someone.someother@somewhere.ch',
    salt             => 'Rooraenietu8Eeyo<Qu2eeNfterd-dd+',
    cipherseed       => '9999999999999999999999999999999999999999999999999999999999999999',
  }

```

## Parameters of the MIPS Class

The MISP class can take every parameter needed to change the configuration of MISP. However, they all have the default value 
set to the recommended value so there is no need to change it. The parameters can be classified in the ones needed for 
the installation of MISP itself, for the database, for the configuration and for the services. The parameters are the following:

### MISP installation
  
* `misp_git_tag` - Version of MISP that will be installed. By default "v2.4.51".
* `install_dir` - Directory in which MISP will be installed. By default "/var/www/MISP/".
* `config_dir` - Directory in which the configuration of MISP should be located. By default "install_dir/app/Config/".
* `stix_git_repo`- Git url of the STIX module. By default "git://github.com/STIXProject/python-stix.git".
* `stix_git_tag`- Version of the STIX module. By default "v1.1.1.4".
* `cybox_git_repo`- Git url of the CyBox repository. By default "git://github.com/CybOXProject/python-cybox.git".
* `cybox_git_tag`- Version of the CyBox module. By default "v2.1.0.12".
* `timezone`- Timezone where the instance has been placed. By default "UTC".
* `default_user`- User as which to run the installation of MISP. By default apache.
* `default_group`- Group as which to run the installation of MISP. By default apache.
* `default_high_user`- In some cases root permissions are need in the installation, this user will be used in 
those cases. By default root.
* `default_high_group`- In some cases root permissions are need in the installation, this group will be used in 
those cases. By default apache.

### Database configuration

* `db_name` - Name of the database. By default "misp".
* `db_user` - Name of the user with rights on the database. By defeault "misp".
* `db_host` - Name of the host in which the database is located. By default "misp.com".
* `db_port` - Port to connect to the database in the specified host. By default 3306.
* `db_password` - Password used to access the database. By default is empty.

### MISP configuration

#### Site Configuration
* `debug` - Debug mode for the full instance. By default set to 0 (false).
* `site_admin_debug` - The debug level of the instance for site admins. This feature allows site admins to run debug mode on a live instance 
without exposing it to other users. The most verbose option of debug and site_admin_debug is used for site admins. 
By default false.

This two parameters are recommended to be set to 0 and false respectively. However, if needed they can be set to tru to find 
errors with names/tables in the database. In this case it would be better to just set to true 'site_admin_debug' instead of 
the whole instance.

#### Security
* `security_level` = 'medium'
* `salt` - The salt used for the hashed passwords. Keep in mind, this will invalidate all passwords in the database.
By default set to "Rooraenietu8Eeyo<Qu2eeNfterd-dd+".
* `cipherseed` - Seed for the cipher. Empty by default.
* `auth_method` - Authentication method used for the instance. Empty means default user-password login method. Empty by default.
* `password_policy_length` - Password length requirement. By default set to 6.
* `password_policy_complexity` - Password complexity requirement. By default set to '/((?=.*\\d)|(?=.*\\W+))(?![\\n])(?=.*[A-Z])(?=.*[a-z]).*$/'.

#### MISP
* `footermidleft` = '',
* `footermidright` = '',
* `host_org_id` - The hosting organisation of this instance. If this is not selected then replication instances cannot be added. 
By default is set to 1, meaning the first Organisation in the system.
* `email_subject_org` - The organisation tag of the hosting organisation. This is used in the e-mail subjects. By defualt set to 'ORGNAME'.
* `showorg` - Setting this setting to 'false' will hide all organisation names / logos. By default set to true.
* `background_jobs` - Enables the use of MISP's background processing. By default set to true.
* `cached_attachments` - Allow the XML caches to include the encoded attachments. By default set to true.
* `email` - The e-mail address that MISP should use for all notifications. By default "root@localhost".
* `contact` - The e-mail address that MISP should include as a contact address for the instance's support team. By default "root@localhost"
* `cveurl` - Turn Vulnerability type attributes into links linking to the provided CVE lookup. By default set to 'http://cve.circl.lu/cve/',
* `disablerestalert` - When enabled notification e-mails will not be sent when an event is created via the REST interface. 
By defualt set to false.
* `default_event_distribution` - The default distribution setting for events (0-3). 0 meanse your organisation only, 1 means this
community only, 2 means contacted communities and 3 is all communities. By defualt set to 1.
* `default_attribute_distribution` - The default distribution setting for attributes, set it to 'event' if you would like 
the attributes to default to the event distribution level. (0-3 or "event"). By default set to 'event'.
* `tagging` - Enable the tagging feature of MISP. By default set to true.
* `full_tags_on_event_index` = true,
* `footer_logo` = '',
* `take_ownership_xml_import` = false,
* `unpublishedprivate` = false,
* `disable_emailing` - When enabled, no outgoing e-mails will be sent by MISP. By default set to false.
* `live` - If set to false the instance will only be accessible by site admins. By default true.
* `extended_alert_subject` - Enabling this flag will allow the event description to be transmitted in the alert e-mail's subject. 
By default set to false.
* `default_event_threat_level` - The default threat level setting when creating events. By default set to 1.
* `newUserText` - The message sent to the user after an account creation. By default set to "Dear new MISP user,\\n\\nWe 
would hereby like to welcome you to the $org MISP community.\\n\\n Use the credentials below to log into MISP at $misp, 
where you will be prompted to manually change your password to something of your own choice.\\n\\nUsername: $username\\n
Password: $password\\n\\nIf you have any questions, don\'t hesitate to contact us at: $contact.\\n\\nBest regards,\\nYour 
$org MISP support team".
* `passwordResetText` - The message sent to the users when a password reset is triggered. By default set to "Dear MISP user,
\\n\\nA password reset has been triggered for your account. Use the below provided temporary password to log into MISP 
at $misp, where you will be prompted to manually change your password to something of your own choice.\\n\\nUsername: 
$username\\nYour temporary password: $password\\n\\nIf you have any questions, don\'t hesitate to contact us at: 
$contact.\\n\\nBest regards,\\nYour $org MISP support team".
* `enableEventBlacklisting` - Enable the blacklisting of event UUIDs to prevent them from being pushed to your instance. 
By default set to false.
* `enableOrgBlacklisting` - Enable blacklisting of organisation UUIDs to prevent them from creating events. 
By default set to false.
* `log_client_ip` - All log entries will include the IP address of the user. By default set to false.
* `log_auth` - MISP will log all successful authentications using API keys. By default set to false.
* `disableUserSelfManagement` - When enabled only Org and Site admins can edit a user's profile. By default set to false.
* `block_old_event_alert` - Enable this setting to start blocking alert e-mails for old events. By default set to false.
* `block_old_event_alert_age` = 30,
* `maintenance_message` = 'Great things are happening! MISP is undergoing maintenance, but will return shortly. You can contact the administration at $email.',
* `email_subject_TLP_string` = 'TLP Amber',
* `terms_download` = false,
* `showorgalternate` = false,

#### GPG
* `gpg_onlyencrypted` = false,
* `gpg_email` - By default "no-reply@localhost"
* `gpg_homedir` - By default "/var/www/html"
* `gpg_password` = '',
* `gpg_bodyonlyencrypted` = false,

#### SMIME
* `smime_enabled` = false,
* `smime_email` = '',
* `smime_cert_public_sign` = '',
* `smime_key_sign` = '',
* `smime_password` = '',

#### Proxy
* `proxy_host` = '',
* `proxy_port` = '',
* `proxy_method` = '',
* `proxy_user` = '',
* `proxy_password` = '',

#### SecureAuth
* `secure_auth_amount` - The number of tries a user can try to login and fail before the bruteforce protection kicks in. 
By default set to 5.
* `secure_auth_expire` - The duration (in seconds) of how long the user will be locked out when the allowed number of login 
attempts are exhausted. By default set to 300.

#### Plugin
* `customAuth_disable_logout` = true,
* `ZeroMQ_enable` = false,
* `ZeroMQ_port` = 50000,
* `ZeroMQ_redis_host`= 'localhost',
* `ZeroMQ_redis_port` = 6379,
* `ZeroMQ_redis_password` = '',
* `ZeroMQ_redis_database` = '1',
* `ZeroMQ_redis_namespace` = 'mispq',
* `RPZ_policy` = 0,
* `RPZ_walled_garden` = '127.0.0.1',
* `RPZ_serial` = '$date00',
* `RPZ_refresh` = '2h',
* `RPZ_retry` = '30m',
* `RPZ_expiry` = '30d',
* `RPZ_minimum_ttl` = '1h',
* `RPZ_ttl` = '1w',
* `RPZ_ns` = 'localhost',
* `RPZ_email` = 'root.localhost',
* `sightings_anonymise` = false,
* `sightings_policy` = 0,
* `sightings_enable` = false,
* `export_services_enable` = true,
* `export_services_url` = 'http://127.0.0.1',
* `export_services_port` = 6666,
* `export_timeout` = 10,
* `import_services_enable` = true,
* `import_services_url` = 'http://127.0.0.1',
* `import_services_port` = 6666,
* `import_timeout` = 10,
* `enrichment_services_enable` = true,
* `enrichment_services_url` = 'http://127.0.0.1',
* `enrichment_services_port` = 6666,
* `enrichment_timeout` = 10,
* `enrichment_hover_enable` = true,
* `enrichment_hover_timeout` = 5,

#### ApacheShibbAuth
* `shib_default_org` = '1',
* `egroup_role_match` = {},

### Services

* `webservername` = The name of the service of the web server on top of which MISP is running. By default httpd.
* `redis_server` = If the redis database will be installed locally or not, meaning that the redis server will be installed. By default true.


 
  
The *email* variable specifies the email address that will be used for as sender for the notifications, the *contact* email address is the 
one that will be shown in the displayed messages.
