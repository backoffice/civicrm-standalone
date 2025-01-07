# CiviCRM Standalone (Project Template)

This is a project-template for [CiviCRM Standalone](https://lab.civicrm.org/dev/core/-/wikis/standalone). You may use this to initialize a new web-site running [CiviCRM](https://civicrm.org) -- *without* requiring any other content management system (such as Drupal or WordPress).

This template is based on [composer](https://getcomposer.org/). It is geared toward site-builders. You may use `composer` for site-building operations -- such as upgrading the application and managing third-party add-ons.

__TIP__: If you are planning to _develop patches for CiviCRM Standalone_, you should use a different template: [civibuild's `standalone-dev`](https://github.com/civicrm/civicrm-buildkit/tree/master/app/config/standalone-dev). In that template, it is easier to manage `git` (branches, commits, etc) for the canonical repositories.

__See also__: [(dev/core#2998) Epic: Re-implement CiviCRM Standalone (CMS-less CiviCRM)](https://lab.civicrm.org/dev/core/-/issues/2998)

<!--
It is still in the early development stages. Eventually we aim to provide a zip/tar archive that can be uploaded to a web server. Currently it requires [composer](https://getcomposer.org/) or [buildkit](), which are fairly technical tools used mostly by (respectively) PHP and CiviCRM developers.

## Development

We are using CiviCRM's Gitlab for issues/discussion, and Github pull-requests for tracking code changes.

* CiviCRM Standalone meta-issue: https://lab.civicrm.org/dev/core/-/issues/2998
* CiviCRM Standalone github repo: https://github.com/civicrm/civicrm-standalone

We also have a [CiviCRM Standalone channel](https://chat.civicrm.org/civicrm/channels/standalone) on CiviCRM's chat.

There are four key repositories:

1. [Buildkit](https://github.com/civicrm/civicrm-buildkit/) *The standalone work is now included in the main branch.*
2. [cv](https://github.com/civicrm/cv) *The standalone work is now included in the main branch.*
3. [This repo (civicrm-standalone)](https://github.com/civicrm/civicrm-standalone)
4. CiviCRM Core, of course. Standalone support and the Standalone Users extension (which was in a [separate repo](https://lab.civicrm.org/extensions/standaloneusers/)) are all in core now.

![Diagram showing how repositories relate](images/repos.excalidraw.png)

In words:

- Buildkit provides the `civibuild` command for creating local instances of 
  CiviCRM. It includes instructions for fetching this repo.

- Those instances include the `cv` command/tool, and have this repo as the project's 
  webroot.

- The composer.json file in this repo pulls in the master branch of CiviCRM core.

- The standalone users core extension is then installed on the instance using 
  the command line (it will be hidden from the extensions UI), to provide the access restrictions.
-->

## Project layout

This repo as top dir of project:

- `private/` holds non-web-accessible files including logs, Smarty 
 templates, settings, temporary files (compiler cache and tmp) and private uploads.
- `public/` holds web-accessible and web-writable files such as images and public assets genereated dynamically by CiviCRM (packages and core).
- `index.php` - the main router/request handler which must be in the webroot.
- `ext/` store CiviCRM extensions. This folder is usually web-writeable to allow installing extensions through the UI, but need not be if extensions are managed as part of the site build process.
- `vendor/civicrm/` holds all the composer-sourced code, notably including:
  - `civicrm-core` The core files
- `civicrm.standalone.php` - provides a common codepath for `index.php`, `cv`, and so on, to locate and run the CiviCRM autoloader, classloader and settingsloader.

Note: the file structure can be updated to have a nested webroot, (for example, `web`). In which case, place the `public/`, `index.php`, `.htaccess` and optionally the `ext/` files and directories under the nested webroot. Then update `civicrm.standalone.php` and `index.php` and `composer.json` (paths to the public assest) to reflect the new structure. You may also need to update the paths in `private/civicrm.settings.php`.

## Install with buildkit

```
civibuild create mytest1 --type standalone-clean
```

Note that this will always install the latest master/main branch.

You should now be able to see CiviCRM up and running. Jump to Next Steps.

## Install with composer

If you don't want the buildkit environment and you want to test the web-based installer, you can do it this way. It assumes you have setup your own httpd/php/sql services and configured them.

Clone this repo as the root of your project and pull in dependencies:

```
cd /var/www/
git clone git@github.com:civicrm/civicrm-standalone.git standalone
cd standalone
composer install
```

Unless your php worker runs as your own user, you may need to configure permissions (adjust for your set-up). e.g.

```
PHPWORKERUSER=www-data
mkdir -m2770 -p web/upload data
chmod g+rwX -R web/upload data
chgrp $PHPWORKERUSER web/upload data
```

Now we need to create some SQL and the database:

```
# The brackets here save you a cd - command after!
( cd vendor/civicrm/civicrm-core/xml && php GenCode.php 0 0 Standalone; )

# create the database - you may need to add your credentials if not stored in ~/.my.cnf
mysql -e "CREATE DATABASE standalone_civicrm;"
```

Now visit the site in your browser. You should get the installer page, but with lots of red notices about database stuff. That's ok, it's just because it doesn't know the DSN for the database. Edit the DSN yourself at the bottom of the page and hit Apply.

At the end you should be able to access /civicrm

## Next steps

You need authx enabled and configured, and the standaloneusers extension installed.

You can do this at the command line with

```
cv en authx standaloneusers && cv vset +l authx_login_cred+=pass
```

...or from **Administer » System Settings » Authentication**. You'll need to add *User Password* to the **Acceptable credentials (HTTP Session Login) select. And hit **Save**. Then install the the standaloneusers extension by `cv en standaloneusers` (it is a core extension, so will already be part of your docroot) - you can't do this from the UI as it's hidden.

Read the [standaloneusers/README.md](https://github.com/civicrm/civicrm-core/blob/master/ext/standaloneusers/README.md)

## About CiviCRM

CiviCRM is web-based, open source, Constituent Relationship Management (CRM) software geared toward meeting the needs of non-profit and other civic-sector organizations.

As a non profit committed to the public good itself, CiviCRM understands that forging and growing strong relationships with constituents is about more than collecting and tracking constituent data - it is about sustaining relationships with supporters over time.

To this end, CiviCRM has created a robust web-based, open source, highly customizable, CRM to meet organizations’ highest expectations right out-of-the box. Each new release of this open source software reflects the very real needs of its users as enhancements are continually given back to the community.

With CiviCRM's robust feature set, organizations can further their mission through contact management, fundraising, event management, member management, mass e-mail marketing, peer-to-peer campaigns, case management, and much more.

CiviCRM is localized in over 20 languages including: Chinese (Taiwan, China), Dutch, English (Australia, Canada, U.S., UK), French (France, Canada), German, Italian, Japanese, Russian, and Swedish.

For more information, visit the [CiviCRM website](https://civicrm.org).
