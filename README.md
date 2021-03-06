<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
## Table of Contents

- [Required software](#required-software)
- [First time setup](#first-time-setup)
- [Vagrant as local MAMP/WAMP/XAMP/EasyPHP/... replacement](#vagrant-as-local-mampwampxampeasyphp-replacement)
  - [Let's develop something](#lets-develop-something)
  - [Let's call it a day](#lets-call-it-a-day)
- [Grunt tasks](#grunt-tasks)
  - [Combined (aka. most useful) tasks available for all plugins](#combined-aka-most-useful-tasks-available-for-all-plugins)
    - [`grunt prepare-languages`](#grunt-prepare-languages)
    - [`grunt prepare-assets`](#grunt-prepare-assets)
    - [`grunt prepare-archives`](#grunt-prepare-archives)
    - [`grunt prepare-dev-assets`](#grunt-prepare-dev-assets)
    - [`grunt dev-skin:frontend-{skin-slug}`](#grunt-dev-skinfrontend-skin-slug)
    - [`grunt gitpull:{plugin-slug}`](#grunt-gitpull-plugin-slug)
    - [`grunt start-dev`](#grunt-start-dev)
  - [Tasks available for all plugins](#tasks-available-for-all-plugins)
    - [`grunt checktextdomain`](#grunt-checktextdomain)
    - [`grunt makepot`](#grunt-makepot)
    - [`grunt potomo`](#grunt-potomo)
    - [`grunt sync-cuar-commons`](#grunt-sync-cuar-commons)
    - [`grunt less`](#grunt-less)
    - [`grunt autoprefixer`](#grunt-autoprefixer)
    - [`grunt uglify`](#grunt-uglify)
    - [`grunt bump-version`](#grunt-bump-version)
    - [`grunt wp_readme_to_markdown`](#grunt-wp_readme_to_markdown)
    - [`grunt compress`](#grunt-compress)
    - [`grunt watch`](#grunt-watch)
  - [Tasks specific to the main WP Customer Area plugin](#tasks-specific-to-the-main-wp-customer-area-plugin)
    - [`grunt update-libs`](#grunt-update-libs)
    - [`grunt tx-push`](#grunt-tx-push)
    - [`grunt tx-pull`](#grunt-tx-pull)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Required software

- Install NPM and Grunt
- Install Vagrant (that requires Ruby 1.9.x too)
- Install xgettext ([Installer for Windows](http://mlocati.github.io/gettext-iconv-windows/))

## First time setup

- Checkout this build-environment repository to a folder (e.g. c:/wpca/)
- Install some build-tools dependencies, run `npm install --global --production windows-build-tools`, then
  `npm install --global node-gyp` and add the PYTHON user environment variable `C:\Users\conta\.windows-build-tools\python27\python.exe`
- Run `npm install` in the build environment directory
- Checkout the Customer Area plugin to the wp-plugins folder from [its github repository](https://github.com/marvinlabs/customer-area/)
- Open `vagrant/www/wordpress-default/public_html/wp-config.php` and add the following constants

    ```php
    /**
     * Custom config for WPCA
     * @see https://github.com/marvinlabs/wpca-products-build-environment/issues/5
     */
    // Environment related
    define('WP_ENV', 'development');
    define('WP_CONTENT_DIR', __DIR__ . '/wp-content');
    define('WP_CONTENT_URL', 'http://' . $_SERVER['HTTP_HOST'] . '/wp-content/');
    define('WP_PLUGIN_DIR', dirname(dirname(dirname(__DIR__))) . '/wpca-plugins');
    define('WP_PLUGIN_URL', 'http://' . $_SERVER['HTTP_HOST'] . '/wp-content/plugins/');
    
    // Errors display
    define( 'WP_DEBUG', true );
    define('SAVEQUERIES', true);
    ini_set('display_errors', 1);
    define('WP_DEBUG_DISPLAY', true);
    define('SCRIPT_DEBUG', true);
    ```
- Open `vagrant/config/nginx-config/nginx-wp-common.conf` and comment out the following lines or you will get an error
  like `Multisite support not enabled` when browsing the `My files page`.
  
  ```
  # Pass uploaded files to wp-includes/ms-files.php.
  # rewrite /files/$ /index.php last;
  
  # if ($uri !~ wp-content/plugins) {
  #    rewrite /files/(.+)$ /wp-includes/ms-files.php?file=$1 last;
  # }
  ```
  

## Vagrant as local MAMP/WAMP/XAMP/EasyPHP/... replacement

This build environment makes use of Vagrant (and more specifically of 
[VVV](https://github.com/Varying-Vagrant-Vagrants/VVV)) to provide a web server contained in a virtual machine.

Please refer to the documentation of [VVV](https://github.com/Varying-Vagrant-Vagrants/VVV) to know how to install and 
start this virtual machine. Basically, all files for your webserver will be shared in the folder `vagrant/www`. 

*Hint: if you still want to use *AMP or any other local server, you can edit the `grunt/config/sync.json` file and
add your web server's plugins folder to that list*

*Hint: to debug you can use xdebug. Here is some [tutorial to setup PhpStorm to work with that VM](http://devincharge.com/debug-cli-remote-server/)*

### Let's develop something

When you want to start developing, run `. vagrant-start.sh` which will:
  
- start the vagrant VM (the first time you do it can take a few minutes)
- Open the `vvv.test` URL in your favorite browser so you can access the sites easily
- TODO : watch for changes in the `wp-plugins` for LESS, JS, PHP.

### Let's call it a day

Once you are finished, just run `. vagrant-stop.sh` to halt gracefully the VM.

### Let's re-provision the VM

Sometimes, you may pull the last environment-build from this repo, and see that some changes have been made in the
`vagrant-config` folder. In this case, you can re-provision the VM by running `. vagrant-provision.sh`.
> Note :
> This command run a `vagrant up --provision` and also copy some config files before that.
> If the VM is already running, do not forget to stop it before by running `. vagrant-stop.sh`.

You will notice a folder called `vagrant-config`. All the directories and files within this folder will be recursively
copied to `vagrant/` before provisioning. This folder contains some VVV customisations for our build-environment,
so the files should not be edited there.

## Grunt tasks

All grunt tasks shall be run in the build environment directory directly

### Combined (aka. most useful) tasks available for all plugins

#### `grunt prepare-languages`

When translations need a refresh.

Runs sequentially: `checktextdomain`, then `makepot`, then `potomo`

#### `grunt prepare-vendors`

When you are updating the vendors (via bower or composer). Some of them are copied and prepared for use in WP Customer
Area projects (for instance, Bootstrap classes are prefixed - prefixed bootstrap is no more used).

Runs sequentially: `copy:copy-bootstrap`, then `copy:prefix-bootstrap`

#### `grunt prepare-assets`

When asset sources have changed and we need to compile them  

Runs sequentially: `copy:libs-assets-extras`, then `prepare-dev-assets`, then `less`, then `postcss`, then `uglify`, then `update-cuar-versions`

#### `grunt prepare-archives`

When you are ready to release a new version and want to build a zip file for publishing your add-on

Runs sequentially: `compress`

#### `grunt prepare-dev-assets`

When you need to update the nuancier CSS file for development purpose.
It means that when developping on local.wordpress.test, you'll get a "open" button on the bottom left corner of the
screen that will open a nuancier including almost all bootstrap variables and their values.

Basically, the `dev-vars` task creates a CSS file from Bootstrap variables parsed values, and the second one compile it
so we can import it. The CSS import of this file is done into `customer-area/skins/frontend/master/cuar-functions.php`

Runs sequentially, for each skin: `dev-vars:{skin-slug}`, then `less:cuar-skin-{skin-slug}-less-vars`

#### `grunt dev-skin:frontend-{skin-slug}`

Everytime a skin is registered in `grunt/config/skins.json`, and that you want to work on it, you can compile it's style
by running this command. It will also allow you to use sourcemaps, and it will parse the bootstrap variables and their
values so you will get a nuancier showing you all the colors from your skins while developping.
This will also compile the JS files and allow sourcemaps when the `{skin-slug}` is `master`.

#### `grunt gitpull:{plugin-slug}`

When you need to pull updates from a plugin repository

> Note that this task can be run as a standalone task without a {plugin-slug} so you can pull all repositories at once.

#### `grunt start-dev`

When you want to start working with your local web server

Runs sequentially: `watch`

### Tasks available for all plugins

#### `grunt checktextdomain`

Checks that all the source code uses the proper text domain in internationalization functions

#### `grunt makepot`

Create the POT files from the source code

#### `grunt potomo`

Compile all `.po` files to `.mo` files 

#### `grunt sync-cuar-commons`

Use this when you want to copy all files required by an add-on from the main plugin.

- Copies customer-area/libs/cuar/** to each addon's folder

#### `grunt less`

Compile the LESS source code into CSS stylesheets. 

LESS files are taken from:

- Base plugin skins' `src/less` folder and compiled to the skin's `assets/css` folder
- 3rd party add-ons' `src/less` and compiled to the add-on's `assets/css` folder

#### `grunt autoprefixer`

Remove/add prefixes for CSS properties. This is usually run as post-processing of the `less` task. This task is run on
the CSS assets of the base plugin skins as well as 3rd party add-ons' CSS assets.

#### `grunt uglify`

Combines and compresses Javascript files into a unique file. 

JS files are taken from each plugin's `src/js` folder and compiled to the plugin's `assets/admin/js` and/or 
`assets/frontend/js` folder

#### `grunt bump-version`

Update the version number for a given plugin. Usage: `grunt bump-version:plugin:mode`

Examples:

- `grunt bump-version:customer-area:minor`
- `grunt bump-version:customer-area-login-form:major`
- `grunt bump-version:customer-area-notifications:patch`

#### `grunt wp_readme_to_markdown`

Convert each `readme.txt` file to a `README.md` file more appropriate for git repositories 

#### `grunt compress`

Make a zip file of each plugin and place it in the releases folder. File inclusion/exclusion can be adjusted in the file
named `grunt/config/build.json`.
  
#### `grunt watch`

Watch for file changes in:

- the `wp-plugins` folder to start a synchronisation task with the local web server
  TODO : watch for JS, CSS LESS files

### Tasks specific to the main WP Customer Area plugin

#### `grunt update-libs`

Use this when you need to update all the libs for WPCA. This will run updates from bower, composer, and custom libs,
copy the required vendors files to WPCA, and recompile LESS and JS assets.

#### `grunt tx-push`

Use this when you have finished developing something and need to update the translation repository for translators.

*Note: You need to create the .transifex file with your credentials to wp-translations.org before you can use this task*

- Run the `prepare-languages:customer-area` task
- Push POT file to wp-translations.org
 
#### `grunt tx-pull`  

Use this when translators have finished working and you need to update the PO/MO files in the plugin folder. 

*Note: You need to create the .transifex file with your credentials to wp-translations.org before you can use this task*

- Push latest PO files from wp-translations.org
- Compile PO files to MO files

## Debugging JS

For instance, for Switch Users, in Chrome.

- In the panel "sources", we see a yellow folder in the section NETWORK (not Workspace). 
- The JS file appears. When clicking on it, we see the right source file, which means that sourcemaps are correct. 
- We then need to add that file to the workspace: right clic > add folder to workspace > pick the folder `wp-plugins/customer-area-switch-users/src/js/frontend`
- Authorize access to Chrome and reload the page
- Then clic on the workspace file and pick `map to network resource`
- You can then select `switch-users.js` in the scripts loaded from network
