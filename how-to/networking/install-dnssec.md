# DNS Security Extensions (DNSSEC)

DNSSEC is a set of security extensions to DNS which allow DNS data to be verified for authenticity and integrity.

This how-to will show how to enable DNSSEC for an existing zone in your bind9 DNS server deployment.

## Starting point
The starting point for this how-to is an existing bind9 DNS server deployed with an authoritative zone. For details on how to deploy bind9 in this fashion, please see the [DNS How-To](https://github.com/panlinux/ubuntu-server-documentation/blob/bind9-dnssec/how-to/networking/install-dns.md).

Nevertheless, here is a quick set of steps to reach that state for an example domain called `example.internal`.

 * First, install the bind9 package:

  sudo apt install bind9 -y

 * edit `/etc/bind/named.conf.local` and add this *zone* block:

    zone "example.internal" {
        type master;
        file "/etc/bind/db.example.internal";
    };

 * Create the file `/etc/bind/db.example.internal` with these contents:

    $TTL 86400      ; 1 day
    example.internal.       IN SOA  example.internal. root.example.internal. (
                                    7          ; serial
                                    43200      ; refresh (12 hours)
                                    900        ; retry (15 minutes)
                                    1814400    ; expire (3 weeks)
                                    7200       ; minimum (2 hours)
                                    )
    example.internal.       IN  NS      ns.example.internal.
    ns                      IN  A       192.168.1.10
    noble                   IN  A       192.168.1.11

 * restart the service:

    sudo systemctl restart named

 * Check if the service can resolve the name `noble.example.internal`:

    $ dig @127.0.0.1 +short noble.example.internal
    192.168.1.11

