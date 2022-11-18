# offlineimap-xoauth2
Using offlineimap with MS Office 365 and oauth2 authentication

This repo is a set of instructions to get offlineimap working with MS 365 account using only oauth2 athentication.

You will need the project https://github.com/UvA-FNWI/M365-IMAP  also installed. I have put it in the ~/Source directory but it can go anywhere. You will need to at least call the function
```
python3 get_token.py
```
At least once to produce the tokens. It will need to be called each time you change your password.

I had to moodify the config.py file with the following line:

```
Authority = "https://login.microsoftonline.com/5f4af3ad-8646-414b-83d8-ef95a0f39e42"
````

Note the last part of this url is simply the tenant_id, which you can find below. 

Change the .offlineimap.py file to represent where you have put the M365-IMAP project.

# file: .offlineimaprc
These lines must be in the offlineimaprc file:

```
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
oauth2_access_token_eval = get_access_token()
oauth2_tenant_id = 5f4af3ad-8646-414b-83d8-ef95a0f39e42
oauth2_client_id = 08162f7c-0fd2-4200-a84a-f25a4db0b584
oauth2_client_secret = TxRBilcHdC6WGBee]fs?QR:SJ8nI[g82
realdelete = no
folderfilter = lambda folder: folder in ['Sent','INBOX','Archive']
```

# file: .offlineimap.py
```
def get_authinfo_password(machine, login, port):
    s = "machine %s login %s password ([^ ]*) port %s" % (machine, login, port)
    p = re.compile(s)
    authinfo = os.popen("gpg -q --no-tty -d ~/.authinfo.gpg").read()
    return p.search(authinfo).group(1)

def get_access_token():
    token_file = open("/home/jenkins/Source/M365-IMAP/imap_smtp_access_token", "r")
    data = token_file.read()
    token_file.close()
    return data

def get_refresh_token():
    refresh_token = os.popen("cd /home/jenkins/Source/M365-IMAP/; python3 refresh_token.py")
    token_file = open("/home/jenkins/Source/M365-IMAP/imap_smtp_refresh_token", "r")
    data = token_file.read()
    token_file.close
    return data
```
The values oauth2_access_token_eval is found in the offlineimap.py file. It reads in the value of the access_token created by the get_token.py file. This token is saved in the same directory as the M365-IMAP project folder. Similarly the refresh token function, which first calls the script to refresh the token from the server.

Note also that the offlineimap.py file also contains a script to get the password from an encrypted .authinfo.gpg file (function get_authinfo_password). Theoretically this can be used to access also the token, but that is left as an exercize to the reader.

# oauth2_tenant_id

The oauth2_tenant_id is found with the m365 cli tool:
```
m365 tenant id get
```
Note that the m365 tool is simply installed with the following:
```
sudo npm install -g @pnp/cli-microsoft365
```
(sudo for global install)

the oauth2_client_id and oauth2_client_secret is taken from the https://github.com/UvA-FNWI/M365-IMAP config.py file. If you have your own Azure app you can replace these values.

# repeat offlineimap
Sometimes the authentication just fails on the first attempt and then needs to simply be repeated. It works on the second try. Most likely this is due to the refresh_token taking more time before it proceeds to authentication.
