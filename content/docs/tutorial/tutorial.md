+++
title = "Setup"
description = ""
date = 2021-08-04T19:17:23+00:00
updated = 2021-08-04T19:17:23+00:00
draft = false
weight = 1
sort_by = "weight"
template = "docs/page.html"

[extra]
lead = ""
toc = true
top = false
+++

To start, add GGRS to your dependencies:

```toml
[dependencies]
ggrs = "0.5"
```

## Assumptions

GGRS assumes that you have addresses and ports of every client you want to synchronize with. It does not handle NAT traversal nor does does it provide a signalling server. Check out 👉[this article](https://keithjohnston.wordpress.com/2014/02/17/nat-punch-through-for-multiplayer-games/) or 👉[this article](https://bford.info/pub/net/p2pnat/) for more information on the topic.

TODO determinism requirements
