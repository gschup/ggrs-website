+++
title = "Time Synchronization"
description = ""
date = 2021-09-13T00:00:00+00:00
updated = 2021-09-13T00:00:00+00:00
draft = false
weight = 5
sort_by = "weight"
template = "docs/page.html"

[extra]
lead = ""
toc = true
top = false
+++

When running a P2PSession, one client may over time get ahead of another. This can happen due to numerous reasons, such as one machine running slow or one session starting slightly early. Running ahead like this is a disadvantage, as the other clients receive inputs well in time, while the client ahead has to rollback much more frequently. There are multiple ways to mitigate this and bring the clients back to running in sync. GGRS exposes two main ways to handle time rift: The `WaitRecommendation` Event and `P2PSession::frames_ahead`.

## Synchronizing via Events

The easiest way is to fetch GGRSEvents via `session.events()`:

```rust
for event in sess.events() {
    if let GGRSEvent::WaitRecommendation { skip_frames } = event {
        frames_to_skip += skip_frames
    }
    println!("Event: {:?}", event);
}
```

When executing frame updates, simply skip a number of frames in your game loop:

```rust
[...]
if frames_to_skip > 0 {
    frames_to_skip -= 1;
    println!("Frame {} skipped: WaitRecommendation", game.current_frame());
    continue;
}
[...]
```

GGRS will send `WaitRecommendation` events no more than once per second and only if the local client has consistenly been three or more frames ahead for at least 30 frames.
This method is a simple way to keep clients running in sync, but is has drawbacks, since skipping frames may be jarring to the user.
Additionally, even with only a two-frame rift, one client will experience more rollbacks than the other.

You can check out this approach in 👉[BoxGame P2P](https://github.com/gschup/ggrs/tree/main/examples/box_game/box_game_p2p.rs).

## Manual Time Rift Synchronization

In addition to the event-based recommendations, GGRS exposes `P2PSession::frames_ahead()`, which will give you a number indicating how many frames the client believes to be ahead
based on the last 30 frames. If this number is negative, your local client is behind. If positive, your local client is ahead. If you have control over exact frame timings, you can run the client who's ahead slightly slower than usual, allowing for remote clients to catch up.

```rust
let mut fps_delta = 1. / self.fps as f64;
if self.run_slow {
    fps_delta *= 1.1;
}

[...]

self.run_slow = session.frames_ahead() > 0;
```

This way, the user will not experience stutters due to skipped frames. Additionally, the time rift will consistently be smaller. Optionally, you could think about running the client behind slightly faster or having a scalable slowdown or speedup based on the discrepancy size. When handling time rift via `session.frames_ahead()`, you might want to disregard `WaitRecommendation` events, based on your implementation.

You can check out this approach in 👉[Bevy GGRS](https://github.com/gschup/bevy_ggrs/blob/main/src/ggrs_stage.rs).
