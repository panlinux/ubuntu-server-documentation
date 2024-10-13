# DNSSEC

DNSSEC stands for Domain Name System (DNS) Security Extensions (SEC). It's an extension to DNS that introduces digital signatures so that each DNS response can be verified for:

 * integrity: The answer was not tampered with and did not change during transit.
 * authenticity: The data came from the true source, and not another entity impersonating the source.

It's important to note that DNSSEC, however, will NOT encrypt the data: it is still sent in the clear.

DNSSEC is based on public key cryptography, meaning that every DNS zone has a public/private key pair. The private key is used to sign the zone's DNS records, and the corresponding public key can be used to very those signatures. This public key is also published in the zone, and anyone fetching records can also fetch the public key to verify the signature of the data.

The question then becomes, how to trust that this public key is authentic? Turns out the key is also signed: it's signed by the parent zone's key, which is also signed by its parent, and so on, all the way to the top: the root DNS zone. Those last keys are trusted implicitly, and all DNS resolvers have it as an anchor. This sequence of keys and signatures all the way to the top is called the chain of trust.

The public key cryptography behind SSL/TLS is similar, but there we have the Certificate Authority entity (CA) that issues the certificates and vouches for them. Every single web browser or other web client needs to have a "bootstrap" list of CAs that it will trust by default, and there are dozens. In DNSSEC, the only bootstrap public key the resolver needs is the root zone one. It's as if there was only one trusted CA.

DNSSEC suddenly made it more attractive to store other types of information in DNS zones. Although it has always been possible to store SRV, TXT and other generic records in DNS, now these can be signed, and can thus be relied upon to be true. A well known initiative that leverages DNSSEC for this purpose is DANE: DNS-based Authentication of Named Entities (RFC 6698, RFC 7671, RFC 7672, RFC 7673).


# References

 * DNSSEC - What Is It and Why Is It Important: https://www.icann.org/resources/pages/dnssec-what-is-it-why-important-2019-03-05-en
 * Tool to visualize the DNSSEC chain of trust of a domain: https://dnsviz.net/
 * DANE: https://en.wikipedia.org/wiki/DNS-based_Authentication_of_Named_Entities

