# Posit Configuration Defaults

The goal of this repository is to capture a new default set of configuration files that exemplify the most commonly needed/used configuration options, following the 80/20 rule. This, in turn, will hopefully prevent customers from needing to round-trip to docs site in order to speed installation and configuration of Posit products while exposing common configurations and features without needing admin intervention to enable features they've already paid for.

## Guiding Principles:
- Each option should include at minimum, a direct link to the configuration/definition page for that option and a short description of the option.
- The configurations must be valid on a new system with no other configurations and no uncommented values
- Value verbosity and real-world examples over technical specifics. The target audience for these comments and links is the over-worked sysadmin who hasn't checked on Connect in a few months and is trying to fix an issue quickly without having to employ too much Google-Fu


## Product Changes

### Connect

Connect currently defaults enabling Python/Quarto/NodeJS to false. We should automatically enable these runtimes and enable version scanning for those three languages only against installations of Python/Quarto/NodeJS in the `/opt`. Warn if they runtimes aren't installed or found at startup, and if a customers license doesn't include them, ie. NodeJS. If we can remove the baseline items in teh configuration file, it would be best just to enable the appropriate runtimes at startup based on the license tier, then we don't need a warn or log message about NodeJS for Basic/Enhanced licenses.

### Package Manager

A vast majority of customers at all license tiers create a `cran` and `pypi` repo the first thing they do. This is counter-intuitive and those "basic" repos should be enabled and configured at startup. More complex customers can remove those or replace them with different sources/repos. This will have the additional side benefit of not requiring pod access for PPM when deployed in Kubernetes for some POC/Simpler customer environments. 


>**Warning**
>These configuration files include references to external files that won't exist on your systems by default, such as your organizations certificate and key files, which you will need to generate internally, place on the server, and then edit in the configuration files for your product. Additionally, you'll need to customize authentication parameters, and file paths across the configuration files

