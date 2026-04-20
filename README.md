```bash
#!/bin/bash
# Usage: bash log.sh
# OR: cat /var/log/messages | bash log.sh

set -euo pipefail

# Configuration
filters=("wpa_supplicant" "ACPI Warning" "pulseaudio") # Add your full list here
exclude_regex=$(printf "|%s" "${filters[@]}")
exclude_regex=${exclude_regex:1}

files=( /var/log/syslog /var/log/messages )

process() {
    # Clean input, filter, then colorize.
    tr -cd '\11\12\15\40-\176' | \
    grep -aEi "warn|error|crit|fatal" | \
    grep -avE "$exclude_regex" | \
    sort -u | \
    grep --color=always -Ei "warn|error|crit|fatal|$" || true
}

for file in "${files[@]}"; do
    if [[ -f "$file" && -r "$file" ]]; then
        tail -n 2000 "$file" 2>/dev/null | sed "s|^|$file: |" | process
    fi
done
```
