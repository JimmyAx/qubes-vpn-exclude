# qubes-vpn-exclude

Hooks for [Qubes-vpn-support](https://github.com/tasket/Qubes-vpn-support) to
exclude specific hosts from the VPN tunnel (implementing [inverse split 
tunneling](https://en.wikipedia.org/wiki/Split_tunneling)). Useful for
when the VPN network won't forward requests to the public internet as this
configuration allows AppVMs behind a VPN ProxyVM to directly connect to the
internet instead of going through the VPN.

Only requests over HTTP (port 80) and HTTPS (port 443) are excluded from the
VPN.

## Requirements

 * Requires Qubes-vpn-support to be installed and working.
 * dnsmasq must be installed in the ProxyVM (or template).
 * ipset must be installed in the ProxyVM (or template).

## Installation and configuration

 1. Enable qubes-vpn-exclude in the ProxyVM by adding `vpn-exclude-domains` to
    the services tab in Qube Manager.

 2. Copy the `qubes-vpn-exclude` folder to the ProxyVM.

 3. Execute `sudo bash ./install` in the `qubes-vpn-exclude` folder in the
    ProxyVM.

 4. Edit `/rw/config/rc.local` and add a call to 
    `/rw/config/qubes-vpn-exclude/rc.local-hook` right before
    `# Start tunnel service`.

 5. Edit `/rw/config/qubes-vpn-ns` and add a call to 
    `systemctl restart dnsmasq.service` right after 
    `do_notify "LINK IS UP." "network-idle"`.

 6. Restart your ProxyVM.

Now you should have a new file `/rw/config/qubes-vpn-exclude.list` in your
ProxyVM. Edit it and add the domain names that you want to exclude. After that
run `systemctl restart dnsmasq.service` for the changes to take effect (or 
reboot your ProxyVM). Wildcards for all subdomains can be specified using the 
syntax `.example.com`. Comments are supported using `#`.

## Technical notes

qubes-vpn-exclude is implemented using iptables, ipsets and dnsmasq.

Upon startup of the ProxyVM iptable rules are added to exclude the ipsets
`qubes-vpn-exclude-4` (for IPv4) and `qubes-vpn-exclude-6` (for IPv6) from the
forced VPN tunnel. By default these ipsets are empty and therefore no domains
are excluded from the VPN tunnel.

When a DNS request is performed from an AppVM the ProxyVM will force redirect 
them to the local DNS server provided by dnsmasq. If the domain matches the 
list of excluded domains dnsmasq will use the local Qubes DNS server to resolve 
them. The IPs for the domain will then be added to either the
`qubes-vpn-exclude-4` or `qubes-vpn-exclude-6` ipsets.

Any domain not matching the exclude list will be forwarded to the DNS in the 
VPN.

qubes-vpn-exclude has been tested with Fedora 30 but should work with any
Linux-based ProxyVM supported by Qubes OS.
