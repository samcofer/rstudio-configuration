# Posit Configuration Defaults

The goal of this repository is to capture a recommended/best practice set of configuration files for all of our professional products. These configurations are *opinionated* meaning that they represent the options we'd like to see our customers utilize, but ultimately we recognize that there is no one size fits all approach to configuring our products. With that in mind, I believe that these configurations will be appropriate for 80% of our customers and can serve as an asset to guide a conversation with the remaining 20% of customers. Included in each configuration is a set of options that are related to security and can enhance product security, but may result in access issues if applied incorrectly.

Repository Styling Guide:
- For each option that you add/modify, please include both a direct link to the configuration/definition page for that option and a short description of the option and potential impacts
- If sections get long enough that they are difficult to distinguish from the section above and below please use commented line separators to provide visual separation of sections
- Value verbosity and real-world examples over technical specifics. The target audience for these comments and links is the over-worked sysadmin who hasn't checked on Connect in a few months and is trying to fix an issue quickly without having to employ too much Google-Fu
- Workbench Example:
```
#-----------------------------------------------------------------------------------------#
# Admin Dashboard Configuration
#
# https://docs.rstudio.com/ide/server-pro/server_management/administrative_dashboard.html
# The Admin Dashboard is not enabled by default (http://<server-address>/admin)
# Admin-superuser-group can: 1.Suspend or terminate active sessions, 2.Assume control of active sessions
# (e.g. for troubleshooting), and 3.Login to RStudio as any other server user
admin-enabled=1
admin-group=rstudio_admin
admin-superuser-group=rstudio_superuser
admin-monitor-log-use-server-time-zone=1
#-----------------------------------------------------------------------------------------#
```
- Connect Example:
```
; -------------- High-Level Server Configuration -------------- ;
[Server]

; Address is a public URL for this RStudio Connect server. Must be configured
; to enable features like including links to your content in emails. If
; Connect is deployed behind an HTTP proxy, this should be the URL for Connect
; in terms of that proxy.
; https://docs.posit.co/connect/admin/appendix/configuration/#Server.Address
Address = "http://rstudio-connect.example.com"
```
- Package Manager Example: 
```
[HTTPS]
; Path to a TLS certificate file. If the certificate is signed by a certificate authority, the
; certificate file should be the concatenation of the server's certificate followed by the CA's
; certificate. Must be paired with `HTTPS.Key`.
;https://docs.posit.co/rspm/admin/appendix/configuration/#HTTPS.Settings

Certificate = "/path/to/certificate/file"

; Path to a private key file corresponding to the certificate specified with `HTTPS.Certificate`.
; Required when `HTTPS.Certificate` is specified.

Key = "/path/to/key"
```

>**Warning**
>These configuration files include references to external files that won't exist on your systems by default, such as your organizations certificate and key files, which you will need to generate internally, place on the server, and then edit in the configuration files for your product. Additionally, you'll need to customize authentication parameters, and file paths across the configuration files

