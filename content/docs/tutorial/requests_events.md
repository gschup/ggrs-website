+++
title = "Requests and Events"
description = ""
date = 2021-09-13T00:00:00+00:00
updated = 2021-09-13T00:00:00+00:00
draft = false
weight = 4
sort_by = "weight"
template = "docs/page.html"

[extra]
lead = ""
toc = true
top = false
+++

## Handling Requests

If calling `advance_frame(...)` succeeds, GGRS will return a `Vec<GGRSRequest>`. Handling requests is mandatory. This sequence of requests is order-sensitive! You need to fulfill all requests in order. There are three types of requests: AdvanceFrame, LoadGameState and SaveGameState. Please see 👉[BoxGame](https://github.com/gschup/ggrs/tree/main/examples/box_game/box_game.rs) for a full code example.

### AdvanceFrame

```rust
AdvanceFrame { inputs: Vec<GameInput> }
```

Advance the frame with the provided inputs.

### LoadGameState

```rust
LoadGameState { cell: GameStateCell }
```

Load a previous gamestate by calling `your_state = cell.load()`. The provided `frame` defines from which frame this gamestate is from.

### SaveGameState

```rust
SaveGameState { cell: GameStateCell, frame: Frame }
```

Save the current gamestate by calling `cell.save(your_state)`.

## Handling Events

Events are notifications from the session for the user. It is recommended to fetch events after every update. Most events are simply of informative nature, requiring no special action from the user. Please see the examples or refer to the documentation what kind of `GGRSEvent` can occur.

One exception to this is the `WaitRecommendation` event, which GGRS gives out when your local clients runs too far ahead of remote clients, leading to a lot of one-sided rollbacks on your end. A simple way to mitigate this discrepancy by skipping the indicated amount of frames. More elaborate means to synchronize the clients are described 👉[here](../time_sync/).
