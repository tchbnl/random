I sometime write little scripts to do my job. Sometimes they even work. Here's ~~some~~ one of them.

**A quick and dirty Apache log parser:**

Pass an access log file and it'll fetch some stats from the last 24 hours.

```bash
qndlp()
{
access_log="${1}"
dates="$(echo "$(date -d '24 hours ago' '+%d/%b/%Y:')|$(date -d 'now' '+%d/%b/%Y:')")"
echo 'Top 10 Requests:'
grep -E  "${dates}" "${access_log}" | grep -E '(GET)|(POST)' | awk -F '"' '{print $2}' | awk '{print $1, $2}' | sort | uniq -c | sort -nr | head -n 10
echo
echo 'Top 10 UAs:'
grep -E "'${dates}" "${access_log}" | awk -F '"' '{print $6}' | sort | uniq -c | sort -nr | head -n 10
echo
echo 'Top 10 IPs:'
while IFS= read -r ip; do
reverse_dns="$(echo "${ip}" | awk '{print $2}' | xargs dig +short -x)"
if [[ -z "${reverse_dns}" ]]; then
reverse_dns='No PTR record found.'
fi
echo -e "${ip}\t\t${reverse_dns}"
done< <(grep -E  "${dates}" "${access_log}" | awk '{print $1}' | sort | uniq -c | sort -nr | head -n 10)
}
```

Might add emojis to it if I'm feeling cute.

## More Scripts

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

```bash
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
