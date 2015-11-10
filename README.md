# Magento Update Tutorial
Useful Tipps for Magento Core and Module Update

## Magento Core Update
### Prearrangement
t.b.d.
### Inspection
t.b.d.
### Update Process
t.b.d.

### Testing
* Test on your local VM if you have one
* If everything is fine, push to Test or Dev environment and test again

### Deployment
* Make backup of SQL and all magento related files of your production environment
* git push
* Run deployment process (if you have one)

## 3rd party Module Update
### Prearrangement
* Install git, modman, n98-magerun
* Move the existing module files to modman structure for better decoupling of module files
* Download latest version of 3rd party module
* 
### Inspection
* Make a diff (e.g. Winmerge, Kaleidoscope) of your current module version and an original module files with the same version. If you have custom changes, move them to a separate module.
* Make a diff (e.g. Winmerge (Win), Kaleidoscope (Mac)) of your current module version and the latest one. Investigate changes. Check mysql update scripts in order to have a quick overview what is going on in the database when updating this module.
* If you have custom template and skin files for your module, you have to update them as well

### Update Process

* git rm -r .modman/YOUR-MODULE-PATH
* Copy the new module files to the path .modman/YOUR-MODULE-PATH
* git status gives you back the changes
* git commit -m "Updated Module from 1.xx to 2.xx"

### Testing
* Test on your local VM if you have one
* Inspect backend settings of module and adjust values
* If everything is fine, push to Test or Dev environment and test again

### Deployment
* Make backup of SQL and all magento related files of your production environment
* git push
* Run deployment process (if you have one)
