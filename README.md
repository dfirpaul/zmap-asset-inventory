# zmap-asset-inventory
Python script which takes internal asset inventory at scale using zmap.  Outputs to a nicely-formatted CSV for delivery to the customer.


## Features:
* Scans entire private IP space (by default)
    * Bandwidth, by default, is capped at 500Kbps
* Automatic reverse-DNS lookups
* Ability to calculate delta between scan results and another list
    * Great for finding stray hosts
* Outputs to CSV
* Can check for EternalBlue, default SSH creds, and open VNC (optional)
* Automatic caching of scan results
    * Run additional port scans without waiting for host discovery or DNS lookups 
    * Saves lots of time if scanning > thousands of hosts


## Typical usage scenario:

**zmap ping scan &rarr; zmap port scan(s) &rarr; modules (optional, e.g. EternalBlue scan) &rarr; CSV**

1. **Host Discovery**
    1. Ensure your /etc/hosts contains the correct DNS information for reverse lookups
    1. Run a ping sweep (defaults to entire private IP range):
        - `$ ./asset_inventory.py`
    1. Tip #1: You can specify a `--blacklist`
    1. Tip #2: All raw output is saved in `~/.asset_inventory`
1. **Port Scans**
    - Multiple ports can be scanned in one go:
        - `$ ./asset_inventory.py -p 21 22 23 80 443 445`
    - Note: Only alive hosts (discovered during the ping sweep) will be scanned unless `--skip-ping` is specified
1. **Service Enumeration**
    - Useful for enumerating host-based controls such as AV on Windows systems
    - Note: Requires an account which can execute code on target systems (e.g. a Domain Admin)
    - To enumerate services:
        1. Edit `services.config` and ensure credentials are valid
            - Note: Impacket's wmiexec is used for execution, and it must be in your path:
                - `$ export PATH=/root/Downloads/impacket/examples:$PATH`
            - Tip: You can pass the hash or use a golden ticket.  A password or hash is recommended; golden tickets can be a bit buggy, and only work on systems with a resolvable hostname.
        1. Ensure credentials are valid (seriously)
        1. Dew it.  All systems with 445 open are scanned by default:
            - `$ ./asset_inventory.py -M enum-services`
            - Tip: You can specify a whitelist 
1. **Additional Checks**
    - To check for EternalBlue:
        - `$ ./asset_inventory.py -M eternalblue`
    - To check for default SSH creds:
        - `$ ./asset_inventory.py -M default-ssh`
    - To check for default open VNC:
        - `$ ./asset_inventory.py -M open-vnc`
    - To check for open fileshares (SMB, FTP, and NFS):
        - `$ ./asset_inventory.py -M open-shares`
    - Multiple modules can be run at once, e.g.:
        - `$ ./asset_inventory.py -M eternalblue open-vnc`
        - `$ ./asset_inventory.py -M all`
1. **Generate CSV**
    - A report is automatically generated after each run
        - They are saved in the working directory (default: ~/.asset_inventory)
    - To combine all past reports:
        - `$ ./asset_inventory.py --combine`
        - A combined CSV file will be created in the current directory


## Usage:
~~~
# ./asset_inventory.py --help
usage: asset_inventory.py [-h] [-t STR [STR ...]] [-p PORTS [PORTS ...]] [-n]
                          [--force-dns] [-B STR] [-i IFC] [-G MAC]
                          [--blacklist FILE] [--whitelist FILE] [-w CSV_FILE]
                          [-f] [-Pn] [--force-ping]
                          [-M [MODULES [MODULES ...]]] [--work-dir DIR]
                          [-d FILE] [--netmask NETMASK] [--combine-all-assets]

Assess the security posture of an internal network

optional arguments:
  -h, --help            show this help message and exit
  -t STR [STR ...], --targets STR [STR ...]
                        target network(s) to scan
  -p PORTS [PORTS ...], --ports PORTS [PORTS ...]
                        port-scan online hosts
  -n, --no-dns          do not perform reverse DNS lookups
  --force-dns           force dns lookups while loading cache
  -B STR, --bandwidth STR
                        max egress bandwidth (default 500K)
  -i IFC, --interface IFC
                        interface from which to scan (e.g. eth0)
  -G MAC, --gateway-mac MAC
                        MAC address of default gateway
  --blacklist FILE      a file containing hosts to exclude from scanning
  --whitelist FILE      only these hosts (those which overlap with targets)
                        will be scanned
  -w CSV_FILE, --csv-file CSV_FILE
                        output CSV file
  -f, --start-fresh     don't load results from previous scans
  -Pn, --skip-ping      skip zmap host-discovery
  --force-ping          force a new zmap ping sweep
  -M [MODULES [MODULES ...]], --modules [MODULES [MODULES ...]]
                        Module for additional checks such as EternalBlue (pick
                        from eternalblue, open-vnc, open-shares, default-ssh,
                        enum-services, all, *)
  --work-dir DIR        custom working directory (default
                        /home/bls/.asset_inventory)
  -d FILE, --diff FILE  show differences between scan results and IPs/networks
                        from file
  --netmask NETMASK     summarize networks with this CIDR mask (default 16)
  --combine-all-assets  combine all previous results and save in current
                        directory
~~~