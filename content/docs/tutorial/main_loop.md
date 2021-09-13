+++
title = "Main Loop"
description = ""
date = 2021-09-13T00:00:00+00:00
updated = 2021-09-13T00:00:00+00:00
draft = false
weight = 3
sort_by = "weight"
template = "docs/page.html"

[extra]
lead = ""
toc = true
top = false
+++

In your main game loop, you should call `advance_frame(...)` in fixed intervals. How to do that exactly depends heavily on your software stack. You can also check out 👉[this article](https://medium.com/@tglaiel/how-to-make-your-game-run-at-60fps-24c61210fe75) or 👉[this article](https://gafferongames.com/post/fix_your_timestep/) to learn more about running your own gameloop.

For full code examples, take a look at:

- 👉[BoxGame P2P](https://github.com/gschup/ggrs/tree/main/examples/box_game/box_game_p2p.rs),
- 👉[BoxGame Spectator](https://github.com/gschup/ggrs/tree/main/examples/box_game/box_game_spectator.rs),
- 👉[BoxGame SyncTest](https://github.com/gschup/ggrs/tree/main/examples/box_game/box_game_synctest.rs),
- 👉[Bevy variants](https://github.com/gschup/bevy_ggrs/tree/main/examples/box_game)

## Polling Remote Clients

If you have spare time between rendering and updating your game, you can always call:

```rust
sess.poll_remote_clients();
```

This will receive and handle incoming UDP packets and send queued packets to other remote clients. GGRS should work without explicitly calling this method, but frequent polling leads to timely communication between sessions.
