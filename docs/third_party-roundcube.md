Download Roundcube 1.3.x to the web htdocs directory and extract it (here `rc/`):
```
cd data/web
wget -O - https://github.com/roundcube/roundcubemail/releases/download/1.3.x/roundcubemail-1.3.x-complete.tar.gz | tar xfvz -
# Change folder name
mv roundcubemail-1.3.x rc
# Change permissions
chown -R root: rc/
```

Create a file `data/web/rc/config/config.inc.php` with the following content.

**Change the `des_key` parameter to a random value.** It is used to temporarily store your IMAP password.

```
<?php
error_reporting(0);
if (!file_exists('/tmp/mime.types')) {
file_put_contents("/tmp/mime.types", fopen("http://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types", 'r'));
}
$config = array();
$config['db_dsnw'] = 'mysql://' . getenv('DBUSER') . ':' . getenv('DBPASS') . '@mysql/' . getenv('DBNAME');
$config['default_host'] = 'tls://dovecot';
$config['default_port'] = '143';
$config['smtp_server'] = 'tls://postfix';
$config['smtp_port'] = 587;
$config['smtp_user'] = '%u';
$config['smtp_pass'] = '%p';
$config['support_url'] = '';
$config['product_name'] = 'Roundcube Webmail';
$config['des_key'] = 'yourrandomstring_changeme';
$config['log_dir'] = '/dev/null';
$config['temp_dir'] = '/tmp';
$config['plugins'] = array(
  'archive',
  'managesieve'
);
$config['skin'] = 'larry';
$config['mime_types'] = '/tmp/mime.types';
$config['imap_conn_options'] = array(
  'ssl' => array('verify_peer' => false, 'verify_peer_name' => false, 'allow_self_signed' => true)
);
$config['enable_installer'] = true;
$config['smtp_conn_options'] = array(
  'ssl' => array('verify_peer' => false, 'verify_peer_name' => false, 'allow_self_signed' => true)
);

$config['managesieve_port'] = 4190;
$config['managesieve_host'] = 'tls://dovecot';
$config['managesieve_conn_options'] = array(
  'ssl' => array('verify_peer' => false, 'verify_peer_name' => false, 'allow_self_signed' => true)
);
// Enables separate management interface for vacation responses (out-of-office)
// 0 - no separate section (default),
// 1 - add Vacation section,
// 2 - add Vacation section, but hide Filters section
$config['managesieve_vacation'] = 1;

```

Point your browser to `https://myserver/rc/installer` and follow the instructions.
Initialize the database and leave the installer.

**Delete the directory `data/web/rc/installer` after a successful installation!**

### Enable change password function in Roundcube

Open `data/web/rc/config/config.inc.php` and enable the password plugin:

```
...
$config['plugins'] = array(
    'archive',
    'password',
);
...
```

Open `data/web/rc/plugins/password/password.php`, search for `case 'ssha':` and add above:

```
        case 'ssha256':
            $salt = rcube_utils::random_bytes(8);
            $crypted = base64_encode( hash('sha256', $password . $salt, TRUE ) . $salt );
            $prefix  = '{SSHA256}';
            break;
```

Open `data/web/rc/plugins/password/config.inc.php` and change the following parameters (or add them at the bottom of that file):

```
$config['password_driver'] = 'sql';
$config['password_algorithm'] = 'ssha256';
$config['password_algorithm_prefix'] = '{SSHA256}';
$config['password_query'] = "UPDATE mailbox SET password = %P WHERE username = %u";
```

### Enable OpenPGP encryption/decryption with Mailvelope and Roundcube

Roundcube supports the [Mailvelope](https://www.mailvelope.com/en/help) browser plugin for Firefox and Chromium/Chrome.
With this plugin you can easily encrypt and decrypt messages from within Roundcube.

To install:
- download the plugin from the [Mailvelope website](https://www.mailvelope.com/en/help)
- install [GnuPG for your platform](https://www.gnupg.org/download/index.html)
- open the plugin options and import or create your keys for your email address(es)
- go to your Roundcube webpage ( https://example.org/rc ) and click on the Mailvelope plugin icon in the browser toolbar.  We need to authorise the current domain to work with Mailvelope.  Go to "Advanced Options" and "Activate on current tab" and select OK when presented with the options for the current domain.
- then reload the Roundcube page and you should see an "Encrypt" toolbar icon when composing messages and encrypted messages will also show a small lock icon and be decrypted (if you have the appropriate key)

---

Optionally, you can add Roundcube's link to the mailcow Apps list.
To do this, open or create `data/web/inc/vars.local.inc.php` and add the following code-block:

*NOTE: Don't forget to add the `<?php` delimiter on the first line*

````
...
$MAILCOW_APPS = array(
  array(
    'name' => 'SOGo',
    'link' => '/SOGo/'
  ),
  array(
    'name' => 'Roundcube',
    'link' => '/rc/'
   )
);
...
````
