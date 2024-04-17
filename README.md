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
