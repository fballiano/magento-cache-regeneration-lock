# Magento cache regeneration lock
Avoid DOS when Magento regenerates its cache.

<table><tr><td align=center>
<strong>If you find my work valuable, please consider sponsoring</strong><br />
<a href="https://github.com/sponsors/fballiano" target=_blank title="Sponsor me on GitHub"><img src="https://img.shields.io/badge/sponsor-30363D?style=for-the-badge&logo=GitHub-Sponsors&logoColor=#white" alt="Sponsor me on GitHub" /></a>
<a href="https://www.buymeacoffee.com/fballiano" target=_blank title="Buy me a coffee"><img src="https://img.shields.io/badge/Buy_Me_A_Coffee-FFDD00?style=for-the-badge&logo=buy-me-a-coffee&logoColor=black" alt="Buy me a coffee" /></a>
<a href="https://www.paypal.com/paypalme/fabrizioballiano" target=_blank title="Donate via PayPal"><img src="https://img.shields.io/badge/PayPal-00457C?style=for-the-badge&logo=paypal&logoColor=white" alt="Donate via PayPal" /></a>
</td></tr></table>

## Important

**This patch is not needed starting from OpenMage 20.3.0 because it's beeing merged in the core.**

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

## Why a patch and not directly the patched file?

To have a better compatibility across multiple Magento (1.x) versions, both CE and EE.

## Compatibility
To do the use of very basic Magento's classes/objects it should work on almost every version.

## Support
If you have any issues with this extension, open an issue on GitHub.

## Contribution
Any contributions are highly appreciated. The best way to contribute code is to open a
[pull request on GitHub](https://help.github.com/articles/using-pull-requests).

## Developer
Fabrizio Balliano  
[http://fabrizioballiano.com](http://fabrizioballiano.com)  
[@fballiano](https://twitter.com/fballiano)

## Licence
[OSL - Open Software Licence 3.0](http://opensource.org/licenses/osl-3.0.php)

## Copyright
(c) Fabrizio Balliano

## Similar solutions

https://github.com/madepeople/Made_Cache  
https://github.com/ctidigital/Configuration-Cache-Lock
