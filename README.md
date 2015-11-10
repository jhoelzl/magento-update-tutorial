# Magento Update Tutorial
Useful Overview for Magento Core and Module Update

## Magento Core Update
### Prearrangement
t.b.d.
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
* Update modman file, e.g. with this quick command 
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
