# Verification runbook

Run these checks after opening the Packet Tracer project and waiting for all intended links to turn green.

## 1. Interface status

On each router:

```text
show ip interface brief
```

Expected result: G0/0, G0/1, and G0/2 are `up/up` with the addresses in the addressing plan.

## 2. DHCP allocation

On each PC, open **Desktop → Command Prompt**:

```text
ipconfig /all
```

Expected result: the client receives an address at or above `.21` in its local `/24`, the local router as its gateway, and `192.168.20.10` as its DNS server.

On each router:

```text
show ip dhcp pool
show ip dhcp binding
```

Expected result: the local pool is active and the two local clients have leases.

## 3. Local and remote reachability

From a Site 1 client, test in increasing scope:

```text
ping 192.168.10.1
ping 10.0.12.2
ping 192.168.20.10
ping 192.168.30.1
```

The first attempt may lose a packet while ARP resolves. Repeat it before treating that as a failure.

## 4. DNS resolution

From clients at each site:

```text
ping dns.practice.lab
ping r1.practice.lab
ping sw3.practice.lab
```

Expected result: each name resolves to the address listed in `addressing-plan.md` and receives replies.

## 5. Preferred routing paths

On each router:

```text
show ip route
show ip route static
```

Expected result: the routing table contains one active static route for each remote LAN. The floating route is configured but absent while the preferred next hop is reachable.

From a Site 1 client:

```text
tracert 192.168.30.21
```

Expected result: the preferred path uses R1 then R3.

## 6. Floating-route failover

On R1, disable the direct R1–R2 link:

```text
enable
configure terminal
interface gigabitEthernet0/1
shutdown
end
```

Then confirm the backup route and service continuity:

```text
show ip route static
ping 192.168.20.10
```

Expected result: R1 installs the route to `192.168.20.0/24` through `10.0.13.2` with administrative distance `10`; DNS remains reachable through R3.

Restore the direct link:

```text
configure terminal
interface gigabitEthernet0/1
no shutdown
end
```

Expected result: the lower-distance primary route through `10.0.12.2` returns.

## Troubleshooting order

1. Check cabling and interface state.
2. Check the client address, mask, gateway, and DNS server.
3. Ping the local gateway.
4. Ping the remote destination by IP address.
5. Inspect the router's static routes.
6. Test the destination by DNS name.
7. Inspect the DNS service and A record if IP connectivity works but name resolution fails.

