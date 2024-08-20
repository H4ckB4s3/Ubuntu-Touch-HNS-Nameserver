# Ubuntu-Touch-HNS-Nameserver
Select a HNS Nameserver and change the /etc/resolv.conf file on Ubuntu Touch Devices.

## Installation

Follow these steps.

# Step 1: Create the hns.sh file
```bash
nano hns.sh
```

# Step 2: The script
```bash
#!/bin/bash


# Ensure the script is run with root privileges
if [ "$(id -u)" != "0" ]; then
   echo "You must use 'sudo'!" 1>&2
   exit 1
fi


echo -e '\n- Set HNS servers / Restore default -\n'


RESOLV_CONF="/etc/resolv.conf"


# HNSDNS servers
HNSDNS_SERVERS=("139.144.68.241" "139.144.68.242")


# HDNS servers
HDNS_SERVERS=("103.196.38.38" "103.196.38.39")


# HNSDOH servers
HNSDOH_SERVERS=("194.50.5.26" "194.50.5.27")


PS3='
Please enter your choice: '
options=("HNSDNS" "HDNS" "HNSDOH" "Custom DNS" "Restore OS" "Quit")
select opt in "${options[@]}"; do
    case $opt in
        "HNSDNS")
            DNS_SERVERS=("${HNSDNS_SERVERS[@]}")
            ;;
        "HDNS")
            DNS_SERVERS=("${HDNS_SERVERS[@]}")
            ;;
        "HNSDOH")
            DNS_SERVERS=("${HNSDOH_SERVERS[@]}")
            ;;
        "Custom DNS")
            read -p "Type primary server: " priDN
            read -p "Type secondary server: " secDN
            DNS_SERVERS=("$priDN" "$secDN")
            ;;
        "Restore OS")
            # Remount root as read/write
            mount -o remount,rw /
            # Delete the resolv.conf file
            rm -f "$RESOLV_CONF"
            # Recreate symbolic link to systemd stub-resolv.conf
            ln -s /run/systemd/resolve/stub-resolv.conf "$RESOLV_CONF"
            # Remount root as read-only
            mount -o remount,ro /
            echo -e '\nDefault settings have been restored!\n'
            exit 0
            ;;
        "Quit")
            echo -e '\nOS unchanged!\n'
            break
            ;;
        *)
            echo "Invalid option $REPLY"
            continue
            ;;
    esac


    # If a valid DNS choice was made
    if [[ -n "${DNS_SERVERS[*]}" ]]; then
        # Remount root as read/write
        mount -o remount,rw /
        # Handle symbolic link or regular file case
        if [ -L "$RESOLV_CONF" ]; then
            rm "$RESOLV_CONF"
            touch "$RESOLV_CONF"
        fi
        # Write new DNS settings
        {
            echo "# Custom DNS servers"
            for dns in "${DNS_SERVERS[@]}"; do
                echo "nameserver $dns"
            done
        } > "$RESOLV_CONF"
        # Remount root as read-only
        mount -o remount,ro /
        echo -e '\nDNS servers have been set up!\n'
        exit 0
    fi
done
```
Save and exit
# Step 3: Make the hns.sh script executable
```bash
chmod +x hns.sh
```
# Step 4: Execute the script
```bash
sudo ./hns.sh
```
# Step 4: Execute the script
Choose your HNS nameserver or enter custom nameservers

## License

No license, no limits. Free to use, abuse and improve this code however you see fit.
