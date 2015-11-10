# Magento Update and Deployment Tutorial
Useful Overview for Magento Core and Extenstion Update

## Magento Core Update
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

## 3rd party Module Update
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
* Remove old module files: `git rm -r .modman/YOUR-MODULE-PATH`
* Copy the new module files to the path .modman/YOUR-MODULE-PATH
* Update modman file, e.g. with this [quick command](https://gist.github.com/jhoelzl/08d0c7f4edeece4584bf) `gm` that is placed in your `.bashrc`.
* `git add -u` and `git add .modman/YOUR-MODULE-PATH`
* `git status` gives you back all the changes
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

## Perfect Magento Remote Deployment
A lot of people are using the `maintenance.flag` in the Magento root folder to ensure a maintenance page for the users.

* Enable maintenance mode: `n98-magerun.phar sys:maintenance`
* Remove all remote files (except `media` and `var` folder)
* Deploy Magento Core: `modman deploy magento_core --force --copy`
* Deploy Magento custom core files:  `modman deploy magento_core_custom --force --copy`
* Deploy all other modules through symlinks: `modman deploy-all --force`
* Symlink your custom environment files, such as `robots.txt`, `local.xml`, etc. that are different for each git branch / Magento instance
* Run setup scripts: `n98-magerun.phar sys:run:setup`
* Reindex: `n98-magerun.phar index:reindex:all`
* Flush cache: `n98-magerun.phar cache:flush`
* Disable maintenance mode: `n98-magerun.phar sys:maintenance`
