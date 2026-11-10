# CCNP-ENCOR-Labs

Hands-on BGP and CCNP ENCOR labs built in EVE-NG (hosted on my Proxmox server), with configs, verification output, and Wireshark captures.

## Labs

| Lab | What I proved |
|---|---|
| [01 - eBGP/iBGP fundamentals](labs/bgp-01-ebgp-ibgp-fundamentals/) | iBGP doesn't relay iBGP-learned routes; eBGP rewrites next hop, iBGP doesn't |
| [02 - eBGP/iBGP with Dual Route Reflectors](labs/bgp-02-dual-route-reflectors/) | Dual route reflectors keep clients' routes when one fails; next-hop-self on the RRs fixes inaccessible eBGP next hops; LocPref on the edge router steers the whole AS |
