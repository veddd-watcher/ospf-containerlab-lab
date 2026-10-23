# OSPF Lab: 3 Routers with Containerlab + FRR

Three FRRouting routers running single-area OSPF, built with Containerlab on an M2 Mac (OrbStack, arm64).

## Topology

```
r1 (1.1.1.1) --10.0.12.0/30-- r2 (2.2.2.2) --10.0.23.0/30-- r3 (3.3.3.3)
```

| Router | Loopback | eth1 | eth2 |
|---|---|---|---|
| r1 | 1.1.1.1/32 | 10.0.12.1/30 | - |
| r2 | 2.2.2.2/32 | 10.0.12.2/30 | 10.0.23.1/30 |
| r3 | 3.3.3.3/32 | 10.0.23.2/30 | - |

All interfaces are in area 0, with point-to-point network type on the links (no DR/BDR election).

## Reproduce

```bash
sudo containerlab deploy -t ospf.clab.yml
docker exec -it clab-ospf-lab-r1 vtysh
```

Then on each router: set ospf router-id, and put lo and the link interfaces in ip ospf area 0 with ip ospf network point-to-point on the links.

## Results

See verification.txt: r1 reaches Full with r2, learns 3.3.3.3/32 (cost 20) and 10.0.23.0/30 via 10.0.12.2, and pings 3.3.3.3 with TTL 63 (one routed hop).

## Failure test

I shut eth2 on r2. r2 re-originated its router LSA without that link and flooded it, r1 re-ran SPF, and 3.3.3.3/32 and 10.0.23.0/30 were withdrawn. After bringing the link back up, the adjacency re-formed and the routes returned.

## Next

Bake the router configs into bind-mounted files so the lab comes up pre-configured in one command.

