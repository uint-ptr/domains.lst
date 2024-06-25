# domains.lst

### edit /etc/init.d/getdomains
```sh
#!/bin/sh /etc/rc.common

START=99

start () {
    OUTPUT_FILE="/tmp/dnsmasq.d/domains.lst"
    > "$OUTPUT_FILE"

    DOMAINS="
        https://raw.githubusercontent.com/itdoginfo/allow-domains/main/Russia/inside-dnsmasq-nfset.lst
        https://raw.githubusercontent.com/labi-le/domains.lst/main/d.lst
    "

    count=0
    while true; do
        if curl -m 3 github.com; then
            for DOMAIN in $DOMAINS; do
                echo "Processing $DOMAIN"
                curl -f "$DOMAIN" | tee -a "$OUTPUT_FILE"
            done
            break
        else
            echo "GitHub is not available. Check the internet availability [$count]"
            count=$((count+1))
        fi
    done

    if dnsmasq --conf-file=$OUTPUT_FILE --test 2>&1 | grep -q "syntax check OK"; then
        /etc/init.d/dnsmasq restart
    fi
}
```

```sh
service getdomains restart
```
