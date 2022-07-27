Reset all email accounts under a cPanel account with randomized passwords. Useful if the account was hacked.

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

The reset account and new password will be returned on a new line. The API also returns the cPanel user _itself_, so this will strip out the error that the account doesn't exist. You'll need to change the actual cPanel user password to change that one.

---
