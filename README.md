# offlineimap-xoauth2
Using offlineimap with MS Office 365 and oauth2 authentication

This repo is a set of instructions to get offlineimap working with MS 365 account using only oauth2 athentication.

# file: .offlineimaprc
These lines must be in the offlineimaprc file:

[Repository remoteOutlook]
type = IMAP
ssl = yes
sslcacertfile = /etc/ssl/certs/ca-certificates.crt
remotepasseval = get_authinfo_password("outlook.office365.com", "j.jenkins@fsspx.email", 993)
remoteuser = j.jenkins@fsspx.email
remotehost = outlook.office365.com
remoteport = 993
auth_mechanisms = XOAUTH2
oauth2_request_url = https://login.microsoftonline.com/common/oauth2/v2.0/token
oauth2_access_token_eval = get_token()
oauth2_tenant_id = 5f4af3ad-8646-414b-83d8-ef95a0f39e42
oauth2_client_id = 08162f7c-0fd2-4200-a84a-f25a4db0b584
oauth2_client_secret = TxRBilcHdC6WGBee]fs?QR:SJ8nI[g82
realdelete = no
folderfilter = lambda folder: folder in ['Sent','INBOX','Archive']

The values oauth2_access_token_eval is found in the offlineimap.py file. It reads in the value of the access_token created by the get_token.py file.

The oauth2_tenant_id is found with the m365 cli tool:
m365 tenant id get

Note that the m365 tool is simply installed with the following:
sudo npm install -g @pnp/cli-microsoft365
(sudo for global install)

the oauth2_client_id and oauth2_client_secret is taken from the https://github.com/UvA-FNWI/M365-IMAP config.py file

