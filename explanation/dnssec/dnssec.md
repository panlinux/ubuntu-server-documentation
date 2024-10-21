# DNSSEC

DNS (Domain Name System) is a mapping system between names and IP (Internet Protocol) addresses. Besides this main purpose that allows us to use friendly names instead of a sequence of numbers to reach a web site, it also stores a lot of information about a particular domain, such as:
 * who owns it
 * when it was last updated
 * what are the authoritative name servers for the domain
 * what are the mail exchangers for the domain (i.e., which systems are responsible for email for this domain)
 * mapping of host names to IP addresses, and vice-versa
 * and more

When DNS was first conceived, security wasn't a top priority yet. At its origins, DNS is susceptible to multiple vulnerabilities, and has many weaknesses. Most of them are a consequence of spoofing: there is no guarantee that the reply you got to a DNS query a) was not tampered with; b) came from the true source.

This is not news, and other mechanisms on top of DNS and around it are in place to counteract that weakness. For example, the famous https padlock that can be seen when accessing most websites nowadays, which uses the TLS protocol to both authenticate the website, and encrypt the connection. It doesn't prevent DNS spoofing, and your web browser might still be tricked into attempting a connection with a fraudulent website, but the moment the TLS certificate is inspected, a warning will be issued to the user. Depending on local policies, the connection might be even immediately blocked. Still, DNS spoofing is a real problem, and TLS itself is subject to other types of attacks.

## What is it?

DNSSEC stands for Domain Name System (DNS) Security Extensions (SEC). It's an extension to DNS that introduces digital signatures so that each DNS response can be verified for:

 * integrity: The answer was not tampered with and did not change during transit.
 * authenticity: The data came from the true source, and not another entity impersonating the source.

It's important to note that DNSSEC, however, will NOT encrypt the data: it is still sent in the clear.

DNSSEC is based on public key cryptography, meaning that every DNS zone has a public/private key pair. The private key is used to sign the zone's DNS records, and the corresponding public key can be used to very those signatures. This public key is also published in the zone, and anyone fetching records can also fetch the public key to verify the signature of the data.

The question then becomes, how to trust that this public key is authentic? Turns out the key is also signed: it's signed by the parent zone's key, which is also signed by its parent, and so on, all the way to the top: the root DNS zone. Those last keys are trusted implicitly, and all DNS resolvers have it as an anchor. This sequence of keys and signatures all the way to the top is called the chain of trust.

The public key cryptography behind SSL/TLS is similar, but there we have the Certificate Authority entity (CA) that issues the certificates and vouches for them. Every single web browser or other SSL/TLS client or operating system needs to have a "bootstrap" list of CAs that it will trust by default, and there are dozens. In DNSSEC, the only bootstrap public key the resolver needs is the root zone one. It's as if there was only one trusted CA.

## Other uses

DNSSEC suddenly made it more attractive and secure to store other types of information in DNS zones. Although it has always been possible to store SRV, TXT and other generic records in DNS, now these can be signed, and can thus be relied upon to be true. A well known initiative that leverages DNSSEC for this purpose is DANE: DNS-based Authentication of Named Entities (RFC 6394, RFC 6698, RFC 7671, RFC 7672, RFC 7673).

For example, imagine if the SSH fingerprints for a host you are logging into for the first time were also stored in DNS, and secured via DNSSEC. Instead of being prompted if you recognize the fingerprint, and wish to proceed, all this verification could happen in the background via DNSSEC and the SSHFP resource record published in the DNS zone for that host. OpenSSH is already capable of this via the configuration option `VerifyHostKeyDNS`.


## Where does the DNSSEC validation happen?

DNSSEC validation is the act of, besides fetching the DNS records that were requested, their signature is also obtained, and verified. Who is responsible for that?

It depends.

Let's analyze the simple scenario of a system on a local network performing a DNS query for a domain.

(picture)
laptop, LAN, recursive DNS server in LAN which queries internet DNS servers
(/picture)

Here we have:

 * An Ubuntu system, like a desktop, configured to use a local DNS server.
 * A DNS server configured to perform recursive queries on behalf of the clients from the local network

Let's zoom in a little bit on that Ubuntu system:

(picture)
app -> stub resolver -> queries sent out to recursive DNS servers
(/picture)

When an application needs to translate a hostname to an IP address, it uses standard glibc calls for that job. That is called a stub resolver, or simply a "dns client". This is a very simple client in the sense that it does not perform recursive queries: it expects to dispatch the DNS query to a recursive DNS server, which will do all the hard work.

In Ubuntu, the default stub resolver is systemd-resolved. That's a daemon, running locally, and listening on port 53/udp on IP 127.0.0.53, and the system is configured to use that as its nameserver via `/etc/resolv.conf`:

    nameserver 127.0.0.53
    options edns0 trust-ad

This stub resolver has its own configuration for which recursive DNS servers to use. That can be seen with the command `resolvectl`. For example:

    Global
             Protocols: -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
      resolv.conf mode: stub

    Link 12 (eth0)
        Current Scopes: DNS
             Protocols: +DefaultRoute -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
    Current DNS Server: 10.10.17.1
           DNS Servers: 10.10.17.1
            DNS Domain: lxd

This configuration is usually provided via DHCP, but could also be set via other means. In this particular example, the DNS server that the stub resolver (systemd-resolved) will use for all queries that go out on that network interface is 10.10.17.1. The output above also has `DNSSEC=no/unsupported`: we will get back to that in a moment.

Given what we have:
 * application
 * stub resolver ("dns client")
 * recursive DNS server in the local network
 * authoritative DNS servers in the internet

Where does the DNSSEC validation happen? Who is responsible?

Well, any DNS server can perform the validation. The more, the better. Let's look at two scenarios, and what it means in each case.

### Validating Resolver

When a recursive DNS server is also performing DNSSEC validation, it's called a Validating Resolver. That will typically be the DNS server on your local network.


    (picture)
    laptop, LAN, recursive DNS server in LAN querying external DNS servers
    the local DNS server is also marked as being a Validating Resolver
    (/picture)



# References

 * DNSSEC - What Is It and Why Is It Important: https://www.icann.org/resources/pages/dnssec-what-is-it-why-important-2019-03-05-en
 * Tool to visualize the DNSSEC chain of trust of a domain: https://dnsviz.net/
 * DANE: https://en.wikipedia.org/wiki/DNS-based_Authentication_of_Named_Entities
 * RFC 4255 - Using DNS to Securely Publish Secure Shell (SSH) Key Fingerprints: https://datatracker.ietf.org/doc/html/rfc4255

