## Scripts

Reset all email accounts under a cPanel with randomized passwords. Useful if the account was hacked.

```bash
resettispaghetti()
{
    ACCOUNTS="$(uapi --user=$1 Email list_pops | grep -i "email:" | awk '{print $2}')"
    for ACCOUNT in $ACCOUNTS; do
        PASS="$(openssl rand -base64 16)"
        resetPass="$(uapi --user=$1 Email passwd_pop email="$ACCOUNT" password="$PASS")"
        RESPONSE="$(echo "$resetPass" | grep -i "You do not have an email account named")"
        if [[ $RESPONSE = "" ]]; then
            echo "${ACCOUNT}: ${PASS}"
        fi
    done
}
```

Usage:

```
resettispaghetti username
```

The reset account and new password will be returned on a new line. The API also returns the cPanel user _itself_, so there's code to strip out the error that the account doesn't exist.

## Notes

### Enable WebP support in ImageMagick and PHP on cPanel

The official cPanel article for this is broken.

1. Remove ImageMagick packages if already installed.

```
yum remove ImageMagick ImageMagick-devel
```

2. Add the Remi repo (and optionally disable the remi-safe repo that's enabled by default).

```
yum install https://rpms.remirepo.net/enterprise/remi-release-7.rpm
yum-config-manager --disable remi-safe
```

3. Install the ImageMagick packages from Remi. For some reason `libzip5` isn't a listed dependency, but it's needed.

```
yum install --enablerepo=remi ImageMagick7 ImageMagick7-devel libzip5
```

4. Install the `imagick` PECL for all PHP versions.

```
find /opt/cpanel/ -iname pecl | grep bin | while read pecl; do $pecl install imagick; done
```

### Missing Message-Id: header when sending to Gmail

From WHM => Service Configuration => Exim Configuration Manager, switch to the Advanced Editor tab and look for "custom_begin_mail_pre". Make sure it's checked and add:

```
accept hosts = :

  warn hosts = 127.0.0.1
       control = submission/sender_retain
```
