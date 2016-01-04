# Magento cache regeneration lock
Avoid DOS when Magento regenerates its cache

## The problem
Let's say you reset Magento's cache, on the next run the cache gets recreated but... what if you're website has a lot of traffic and, before the cache regeneration finishes, you got more requests?

What happens is that all the concurrent requests will trigger cache regeneration, sometimes these requests will create a spike in the server(s) load making your website unavailable.

## Why is it happening?

Magento, by default, doesn't use a locking mechanism to prevent multiple concurrent cache regenerations. You could check it in the Mage_Core_Model_App class.

## Solution

Overriding Mage_Core_Model_App class in your "local" codepool implementing a locking mechanism.

## Why can't we use a standard Magento model rewrite?

Cause Mage_Core_Model_App is a special class that's not instanced (by the Magento core) using the Mage::getModel(), it's instead instanced with a "new Mage_Core_Model_App()" in app/Mage.php.

This is why I couldn't create a Magento module for this purpose.

## Side benefit

This solution should also avoid multiple concurrent runs of the resource setup scripts.

## Installation

If you're on Linux or Mac OSX use these lines of code:
```
cd magento-root-directory
mkdir -p app/code/local/Mage/Core/Model
cp app/code/core/Mage/Core/Model/App.php app/code/local/Mage/Core/Model/App.php
wget https://raw.githubusercontent.com/fballiano/magento-cache-regeneration-lock/master/fballiano-magento-cache-regeneration-lock.patch
patch -p1 < fballiano-magento-cache-regeneration-lock.patch
rm fballiano-magento-cache-regeneration-lock.patch app/code/local/Mage/Core/Model/App.php.orig
```
If you're on Windows simply copy core/Mage/Core/Model/App.php to the local pool and apply the patch in this repository.

## Why a patch and now directly the patched file?

To have a better compatibility across multiple Magento (1.x) versions, both CE and EE.

## Why are you directly instancing the Mage_Index_Model_Resource_Helper_Mysql4 class?
At that point in the run time it's not possible to successfully call Mage::getModel() or Mage::getResourceModel() methods, only a limited set of configurations are loaded and the Magento's class name resolution can't work.

##Compatibility
Tested on Magento CE 1.9 and EE 1.13.

The file based flock version can only work on singe server projects or on multiserver projects where the document_root (actually the var directory) is shared via NFS. This solution will not work on a truly separated multiserver environment. I'm working on a different approach for that situation.

##History
The first version of the patch (available at http://bit.ly/1INibTw) used file based flock and could work only on a single server environment (or on multiserver with document root shared via NFS).

The newer version uses Magento's Mage_Index_Model_Resource_Helper_Mysql4 core class which locks on the database, solving the multiserver environment issue with much less custom code.

##Support
If you have any issues with this extension, open an issue on GitHub.

##Contribution
Any contributions are highly appreciated. The best way to contribute code is to open a
[pull request on GitHub](https://help.github.com/articles/using-pull-requests).

##Developer
Fabrizio Balliano  
[http://fabrizioballiano.com](http://fabrizioballiano.com)  
[@fballiano](https://twitter.com/fballiano)

##Licence
[OSL - Open Software Licence 3.0](http://opensource.org/licenses/osl-3.0.php)

##Copyright
(c) Fabrizio Balliano

## Credits

This code was inspired by Made_Cache by Made:
https://github.com/madepeople/Made_Cache
