## Scripts

Reset all email accounts under a cPanel with randomized passwords:

```bash
resettispaghetti()
{
  ACCOUNTS="$(uapi --user="${1}" Email list_pops | grep -i "email:" | grep -i "@" | awk '{print $2}')"
  for ACCOUNT in ${ACCOUNTS}; do
    PASS="$(openssl rand -base64 16)"
    uapi --user="${1}" Email passwd_pop email="${ACCOUNT}" password="${PASS}" >/dev/null
  done
}
```

Usage:

```
resettispaghetti username
```

And reset a cPanel password with something random:

```bash
rotatopotato()
{
  PASS="$(openssl rand -base64 16)"
  whmapi1 passwd user="${1}" password="${PASS}" >/dev/null
}
```

---

Backup MySQL and cPanel user database details before an upgrade:

```
backup_mysql()
{
  BACKUP_DIR="/root/mysql_upgrade.$(date -I)"

  mkdir "${BACKUP_DIR}" || return
  cd "${BACKUP_DIR}" || return

  echo "Dumping databases to disk..."

  for DATABASE in $(mysql -Ne 'SHOW DATABASES;' | grep -Ev 'information_schema|performance_schema'); do
    mysqldump "${DATABASE}" > "${DATABASE}".sql
    
    echo "${DATABASE}" >> databases.txt
    echo "Done: ${DATABASE}"
  done

  echo "Backing up /var/lib/mysql/..."
  
  whmapi1 configureservice service=mysql enabled=1 monitored=0 >/dev/null
  /scripts/restartsrv_mysql --stop >/dev/null

  mkdir -p "${BACKUP_DIR}/var/lib"
  rsync -a /var/lib/mysql "${BACKUP_DIR}/var/lib/"

  whmapi1 configureservice service=mysql enabled=1 monitored=1 >/dev/null
  /scripts/restartsrv_mysql --start >/dev/null
  
  echo "Done."

  echo "Backing up cPanel user database information..."
  
  mkdir -p "${BACKUP_DIR}/var/cpanel"
  rsync -a /var/cpanel/databases "${BACKUP_DIR}/var/cpanel/"
  
  echo "Done."
}
```
