# Posit Configuration Defaults

The goal of this repository is to capture a recommended/best practice set of configuration files for all of our professional products. These configurations are *opinionated* meaning that they represent the options we'd like to see our customers utilize, but ultimately we recognize that there is no one size fits all approach to configuring our products. With that in mind, I believe that these configurations will be appropriate for 80% of our customers and can serve as an asset to guide a conversation with the remaining 20% of customers. Included in each configuration is a set of options that are related to security and can enhance product security, but may result in access issues if applied incorrectly.

## Guiding Principles:
- Each option should include at minimum, a direct link to the configuration/definition page for that option and a short description of the option and potential impacts
- The configurations must be valid on a new system with no other configurations and no uncommented values
- Value verbosity and real-world examples over technical specifics. The target audience for these comments and links is the over-worked sysadmin who hasn't checked on Connect in a few months and is trying to fix an issue quickly without having to employ too much Google-Fu




>**Warning**
>These configuration files include references to external files that won't exist on your systems by default, such as your organizations certificate and key files, which you will need to generate internally, place on the server, and then edit in the configuration files for your product. Additionally, you'll need to customize authentication parameters, and file paths across the configuration files

