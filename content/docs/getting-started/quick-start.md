+++
title = "Quick Start"
description = "One page summary of how to start developing with GGRS."
date = 2021-08-04T19:17:23+00:00
updated = 2021-08-04T19:17:23+00:00
draft = false
weight = 20
sort_by = "weight"
template = "docs/page.html"

[extra]
lead = "One page summary of how to start developing with GGRS."
toc = true
top = false
+++

```toml
[dependencies]
ggrs = "0.4"
```

GGRS mainly operates through one of three sessions; each providing different functionalities:

- `P2PSession`: Communicate with other remote sessions; send and receive inputs to synchronize your game between clients. All Clients participating in the game create their own session and connect to each other in a peer-to-peer fashion.
- `P2PSpectatorSession`: Connects to another `P2PSession` in order to receive confirmed game inputs without contributing to the game input itself. If you want clients to spectate games, this is the session to use.
- `SyncTestSession`: Used mainly for debugging purposes, this session simulates a configurable amount of rollbacks each frame. This is a great way to test if your game updates deterministically. The `SyncTestSession` will compare checksums between original and resimulated gamestates and raise an error if something went wrong.

## Setup

For all sessions, you will have to define the number of active players contributing to the game input via `num_players` as well as the size of such inputs via `input_size`. The `local_port` is where your client will receive packets of remote clients.

GGRS does not handle NAT hole punching. Check out 👉[this article](https://keithjohnston.wordpress.com/2014/02/17/nat-punch-through-for-multiplayer-games/) for more information on the topic.

### P2PSession

First, create a new session.

```rust
let local_port: u16 = 7000;
let num_players : u32 = 2;
let input_size : usize = std::mem::size_of::<u32>();

let mut sess = ggrs::start_p2p_session(num_players, input_size, local_port)?;
```

Then, you should specify the players participating in the game, exactly as many as you defined through `num_players`. You can add exactly one `PlayerType::Local`, the rest should be `PlayerType::Remote(addr)`.

```rust
let local_handle = 0;
let remote_handle = 1;
let remote_addr: SocketAddr = "127.0.0.1:7001".parse()?;

sess.add_player(PlayerType::Local, local_handle)?;
sess.add_player(PlayerType::Remote(remote_addr), remote_handle)?;
```

Optionally, define any spectators you wish to add. The `spectator_handle` should be greater or equal to `num_players`. Internally, GGRS will add 1000 to the provided handle to identify the spectator client, so the resulting internal handle will be 1002 in this example.

```rust
let spectator_handle = 2;
let spec_addr: SocketAddr = "127.0.0.1:7002".parse()?;
sess.add_player(PlayerType::Spectator(spec_addr), spectator_handle)?;
```

You can set input delay for the local player. You do not need to do this for remote players.

```rust
let desired_delay = 2;
sess.set_frame_delay(desired_delay, local_handle)?;
```

Per default, GGRS requests to save the gamestate after every update step in order to minimize rollbacks as much as possible. If saving your game state takes much longer than performing multiple update steps, GGRS has an alternative sparse saving mode where instead the amount of save requests are minimized at the cost of some additional state update requests.

```rust
sess.set_sparse_saving(true)?;
```

Finally, you can start the session. GGRS will then make an effort to establish communication between all clients. Only if the connection to all remotes (including spectators) has been established, the session will be able to advance the gamestate and send inputs.

```rust
sess.start_session()?;
```

### P2PSpectatorSession

First, create a new session. For the spectator, you will have to define a `host_addr` from which this session will receive inputs from all players. The defined host needs to add this client as a spectator, as well.

```rust
let local_port: u16 = 7002;
let num_players : u32 = 2;
let input_size : usize = std::mem::size_of::<u32>();
let host_addr: SocketAddr = "127.0.0.1:7000".parse()?;

let mut sess = ggrs::start_p2p_spectator_session(num_players, input_size, local_port, host_addr)?;
```

Optionally, you can change the default catch-up parameters. Setting the catch-up speed to 1 will prevent any catch-up measures.

```rust
sess.set_max_frames_behind(5)?; 
sess.set_catchup_speed(2)?;
```

Then, you can start the session. GGRS will then make an effort to establish communication to the host. Only then will the session be able to advance the gamestate.

```rust
sess.start_session()?;
```

### SyncTestSession

First, create a new session. `check_distance` specifies how many frames of rollback you want to induce every frame. This value cannot exceed the maximum predicition frames allowed. Currently, 7 is the maximum distance of rollback that can occur.

```rust
let check_distance : u32 = 7;
let num_players : u32 = 2;
let input_size : usize = std::mem::size_of::<u32>();

let mut sess = ggrs::start_synctest_session(num_players, input_size, check_distance)?;
```

Afterwards, you can optionally define input delays for any player you wish.

```rust
let input_delay = 2;

sess.set_frame_delay(input_delay, 0)?;
sess.set_frame_delay(input_delay, 1)?;
```

You don't need to define players or start the session. Because there are no remote clients to synchronize with, a `SyncTestSession` will immediately be ready to advance the game.

## Main Loop

In your main game loop, you should call `advance_frame(...)` in fixed intervals. How to do that exactly depends heavily on your software stack. You can also check out 👉[this article](https://medium.com/@tglaiel/how-to-make-your-game-run-at-60fps-24c61210fe75) or 👉[this article](https://gafferongames.com/post/fix_your_timestep/) to learn more about running your own gameloop.

Please see 👉[BoxGame P2P](https://github.com/gschup/ggrs/tree/main/examples/box_game/box_game_p2p.rs), 👉[BoxGame Spectator](https://github.com/gschup/ggrs/tree/main/examples/box_game/box_game_spectator.rs) or 👉[BoxGame SyncTest](https://github.com/gschup/ggrs/tree/main/examples/box_game/box_game_synctest.rs) for a full code example.

### Handling Requests

If calling `advance_frame(...)` succeeds, it will return a `Vec<GGRSRequest>`. Handling requests is mandatory. This sequence of requests is order-sensitive! You need to fulfill all requests in order. There are three types of requests:

- `AdvanceFrame { inputs: Vec<GameInput> }`: Advance the frame with the provided inputs
- `LoadGameState { cell: GameStateCell }`: Save the current gamestate by calling `cell.save(your_state)`.
- `SaveGameState { cell: GameStateCell, frame: Frame }`: Load a previous gamestate by calling `your_state = cell.load()`. The provided `frame` defines frame from which this gamestate is at.

Please see 👉[BoxGame](https://github.com/gschup/ggrs/tree/main/examples/box_game/box_game.rs) for a full code example.

### Polling Remote Clients

If you have spare time between rendering and updating your game, you can always call:

```rust
sess.poll_remote_clients();
```

This will receive and handle incoming UDP packets and send queued packets to other remote clients. GGRS should work without explicitly calling this method, but frequent polling leads to timely communication between sessions.

### Handling Events

Events are notifications from the session for the user. Please see the examples or refer to the documentation what kind of `GGRSEvent` can occur.
