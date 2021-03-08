# Red vs Blue Project

## Notes

### Network Topology

Linux web server is probably on 192.168.1.105.

### Web Server

* Secret folder location is `company_folders/secret_folder`.
* Message on the page indicates username may be `ashton`.
* Crack with hydra, using command:
```bash
hydra -l ashton -P /usr/share/wordlists/rockyou.txt -s 80 -f -vV 192.168.1.105 http-get /company_folders/secret_folder
```
* Hydra crack successful. Credentials:
  - username: `ashton`
  - password: `leopoldo`

* Note in secret folder indicates webdav can be accessed with user `ryan`, and
  provides password hash.

* Cracked password hash using `john`. Password is `linux4u`.

* Webdav credentials:
  - username: `ryan`
  - password: `linux4u`

* Connected to webdav via `dav://192.168.1.105/webdav` and used above creds.

## Kibana Logs

1. Offensive Traffic

2. Request for the hidden directory
  * 12,150 requests were made for the secret directory (`/company_folders/secret_folder`)
  * 2 requests were made for the file `/company_folders/secret_folder/connect_to_corp_server`
  * These requests were made on February 27, between 18:40:51 and 18:43:41.
  * Files requested were the directory itself, and the file `/company_folders/secret_folder/connect_to_corp_server`
  * An alarm should be set to trigger if more than 100 requests are made for the
    secret folder, or if more than 50% of these requests result in HTTP 401,
    which would be an indicator of a brute-force attack.
  * To harden this machine, we should lock-out specific IP addresses if they
    continually request `/company_folders/secret_folder`, and receive HTTP 401.

3. Identify the brute force attack
  * Traffic specifically from `hydra` can be identified by the user_agent.original `Mozilla/4.0 (Hydra)`
  * 12,142 requests were made during the brute-force attack.
  * The attacker made 12,141 requests before successfully guessing the password
  * An alarm should be set to trigger if more than 100 sequential requests are
    made to the secret folder which result in HTTP 401. This is indicative of
    a brute-force attack.
  * To harden this machine, we should lock-out specific IP addresses if they
    continually request `/company_folders/secret_folder`, and receive HTTP 401.

4. Find the Webdav connection
  * 60 requests were made to the webdav directory, breaking down as follows:
    * 36 for `/webdav`
    * 4 for `/webdav/`
    * 20 for `/webdav/shell.php`
  * The only file requested was `/webdav/shell.php`
  * Since access to `/webdav` is so limited, all traffic requesting `/webdav`
    should alert. Especially traffic that tries to upload files (i.e. PUT
    requests).
