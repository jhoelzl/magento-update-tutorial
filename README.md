# Magento Update and Deployment Tutorial
Useful Overview for Magento Core and Extenstion Update

Contents:
*[Update Magento Core](#magento-core-update)
*[Revert Magento Core Update](#revert-magento-core-update)
*[Update 3rd-party Magento Extenstion](#magento-extension-update)
*[Perfect Magento Deployment Process](#perfect-magento-deployment)


## Magento Core Update <a name="magento-core-update"></a>
### Prearrangement
* Install `git`, `modman`, `composer` and `n98-magerun`
* Download latest Magento CE version

### Project Structure
* If it is possible, separate your existing Magento core files into a modman folder. This folder only contains the magento core files and a modman file.
* For your adjusted Magento core files, such as `app/etc/local.xml`, `.htaccess` or `robots.txt` you can make a separate modman module called `magento_core_custom` which overwrites the original Magento core files.
* A perfect structure would be:
```
project
│   composer.json
│   composer.lock  
│   readme.txt
└───.modman
    ├───magento_core
    ├───magento_core_custom
    ├───firegento_magesetup
    ├───...
└───magento
    │   .modman-skip
    ├───app
    ├───media
    ├───...
    ├───...
    └───var
└───vendor    
```
The magento root folder does not contain any files in `git` repository, all files are deployed from the `.modman` folder into the magento root folder. The `vendor` folder only contains cached files for `composer`. When a module is defined in `composer.json`, it is copied into `.modman/vendor_module`. Then it can be added to `git`.

### Inspection
t.b.d.

### File Update Process
t.b.d.

### Testing
* Test on your local VM if you have one
* Check class rewrite conflicts, e.g. with `n98-magerun.phar dev:module:rewrite:conflicts`
* If everything is fine, push to Test or Dev environment and test again

### Deployment
* Make backup of SQL and all magento related files of your production environment
* git push
* Run deployment process (if you have one)
* When you have the magento core as a modman module, you have to use modman copy strategy instead of symlink, because othwerwise `app/Mage.php` returns an error. In addition, you have to add a `.modman-skip` file into your magento root that includes a list of modman modules that should NOT be deployed with the command `modmand deploy-all`. This would be: `magento_core` and `magento_core_custom`.
* See "Perfect Magento Remote Deployment"

## Revert Magento Core Update <a name="revert-magento-core-update"></a>
In some rare cases it might be better to revert a prior core update on a live system, e.g. if some unwanted behaviour arrise.

## 3rd party Module Update <a name="magento-extension-update"></a>
### Prearrangement
* Install `git`, `modman`, `n98-magerun`
* Move the existing module files to modman structure for better decoupling of module files
* Download latest version of 3rd party module

### Inspection
* Make a diff (e.g. Winmerge, Kaleidoscope) of your current module version and an original module files with the same version. If you have custom changes, move them to a separate module.
* Make a diff (e.g. Winmerge (Win), Kaleidoscope (Mac)) of your current module version and the latest one. Investigate changes. Check mysql update scripts in order to have a quick overview what is going on in the database when updating this module.


### File Update Process
* If you have custom template and skin files for your module, better move them to the modman folder of your custom template or to a separate modman module called `.modman/YOUR-MODULE-PATH_custom`.
* You have to update your custom template files as well. Inspect the diff between the original current module version and the latest module version. Integrate these changes in your custom template and skin files if necessary.
* The new version can have modified, additional, or less files. Therefore just copying the new version over the exisintg one is unsafe, because files that are removed in the new version will stay in our git repository.
* Therefore ,remove all old module files: `git rm -r .modman/YOUR-MODULE-PATH`
* With your FTP client, copy the new module files to the path .modman/YOUR-MODULE-PATH
* Update modman file, e.g. with this [quick command](https://gist.github.com/jhoelzl/08d0c7f4edeece4584bf) `gm` that is placed in your `.bashrc`.
* Add to git stage: `git add -u` and `git add .modman/YOUR-MODULE-PATH`
* `git status` gives you back all the changed files
* `git diff --cached` or `git difftool --cached` gives you back all the changes
* Make a commit: `git commit -m "Updated Module from 1.xx to 2.xx"`

### Testing
* Test on your local VM if you have one
* Inspect backend settings of module and adjust values
* Check class rewrite conflicts, e.g. with `n98-magerun.phar dev:module:rewrite:conflicts`
* Check if module has cronjobs, e.g. with `n98-magerun.phar sys:cron:list`
* Check entries in `var/log/system.log` and `var/log/exception.log`
* If everything is fine, push to Test or Dev environment and test again

### Deployment
* Make backup of SQL and all magento related files of your production environment
* git push to your LIVE instance branch
* Run deployment process (if you have one)

## Perfect Magento Remote Deployment <a name="perfect-magento-deployment"></a>
The following deployment script is used in my production system and usually takes 4-10 minutes (based on number of modules and database size), with a downtime of the website of about 1-2 minutes.

### Better Maintenance Mode
A lot of people are using the `maintenance.flag` in the Magento root folder to ensure a maintenance page for the users. However, this assumes existing Magento Core files. In this deployment, we also deploy the Magento core files (so we have to remove them before), therefore the Magento maintenance mode will fail some time during the process.

For that reason we add two static files `maintenance.php` and `.htaccess.maintenance` in our magento root git repository. The content of `.htaccess.maintenance` includes a simple redirect to the `maintenance.php` page, which can be customized on your own:

````
FileETag None  
<ifModule mod_headers.c>  
Header unset ETag  
Header set Cache-Control "max-age=0, no-cache, no-store, must-revalidate"  
Header set Pragma "no-cache"  
Header set Expires "Mon, 1 Jan 2010 01:00:00 GMT"  
</ifModule>  

<IfModule mod_rewrite.c>
 RewriteEngine on
 RewriteCond %{REQUEST_URI} !/maintenance.php$ [NC]
 RewriteCond %{REQUEST_URI} !\.(jpe?g?|png|gif) [NC]
 RewriteRule .* /maintenance.php [R=503,L]
 ErrorDocument 503 /maintenance.php
</IfModule>
````

During deployment, a symlink from `.htaccess` to `.htaccess.maintenance` returns a stable maintenance page regardless of the existance of Magento core files. Moreover, this redirect preserves the URL of the user in the browser.

### Deployment Script
* Backup sql database 
* Backup all shop files (except var/cache, var/report, var/session)
* Checkout git branch into a deploy.tgz file
* Enable maintenance mode: Make a symlink from `.htaccess` to `.htaccess.maintenance` 
* Remove all remote files (except magento root folder, `media` and `var` folder)
* Unpack deployment file
* Deploy Magento Core: `modman deploy magento_core --force --copy`
* Deploy Magento custom core files:  `modman deploy magento_core_custom --force --copy`
* The file `.htaccess` gets overwritten by the Magento core files, therefore again make a symlink from `.htaccess` to `.htaccess.maintenance` 
* Deploy all other modules through symlinks: `modman deploy-all --force`
* Symlink your custom environment files, such as `robots.txt` to `robots.prod`, `local.xml` to `local.xml.prod`, etc. that are different for each git branch / Magento instance
* Run setup scripts: `n98-magerun.phar sys:run:setup`
* Reindex: `n98-magerun.phar index:reindex:all`
* Flush cache: `n98-magerun.phar cache:flush`
* Remove unsafe folders and files, such as `downloader`, `dev`, `.git`, `gitmodules`, etc.
* Disable maintenance mode: Symlink `.htaccess` to your custom file, such as `.modman/magento_core_custom/.htaccess`
