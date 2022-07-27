Reset all email accounts under a cPanel account. Useful if the account was hacked.
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
