---
layout: default
title: Statefull docker swarm
categories: [docker, swarm, persistence]
category: docker
date:   2015-11-17 16:16:01 -0600
---
# Statefull docker swarm

[Persistence Alternatives](https://opensource.ncsa.illinois.edu/confluence/display/NDS/Gluster+Alternatives+and+Cloud+Provider+Alternatives)

## Tips

- Trigger the failure on the active Swarm Node

```sh
docker node update --availability drain <machine>
```