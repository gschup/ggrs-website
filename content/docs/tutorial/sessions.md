+++
title = "Sessions"
description = ""
date = 2021-09-13T00:00:00+00:00
updated = 2021-09-13T00:00:00+00:00
draft = false
weight = 2
sort_by = "weight"
template = "docs/page.html"

[extra]
lead = ""
toc = true
top = false
+++

GGRS mainly operates through one of three sessions; each providing different functionalities:

- `P2PSession`: Communicate with other remote sessions; send and receive inputs to synchronize your game between clients. All Clients participating in the game create their own session and connect to each other in a peer-to-peer fashion.
- `P2PSpectatorSession`: Connects to another `P2PSession` in order to receive confirmed game inputs without contributing to the game input itself. If you want clients to spectate games, this is the session to use.
- `SyncTestSession`: Used mainly for debugging purposes, this session simulates a configurable amount of rollbacks each frame. This is a great way to test if your game updates deterministically. The `SyncTestSession` will compare checksums between original and resimulated gamestates and raise an error if something went wrong.

## Setup

For all sessions, you will have to define the number of active players contributing to the game input via `num_players` as well as the size of such inputs via `input_size`. The `local_port` is where your client will receive packets of remote clients.

## P2PSession

A P2PSession communicates with other P2PSessions and P2PSpectatorSessions. They send and receive inputs to synchronize your game between clients. All Clients participating in the game create their own session and connect to each other in a peer-to-peer fashion.

```rust
let local_port: u16 = 7000;
let num_players : u32 = 2;
let input_size : usize = std::mem::size_of::<u32>();

let mut sess = P2PSession::new(num_players, input_size, local_port)?;
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

## P2PSpectatorSession

The `P2PSpectatorSession` connects to another `P2PSession` in order to receive confirmed game inputs without contributing to the game input itself. If you want clients to spectate games, this is the session to use. For the spectator, you will have to define a `host_addr` from which this session will receive inputs from all players. The defined host needs to add this client as a spectator, as well.

```rust
let local_port: u16 = 7002;
let num_players : u32 = 2;
let input_size : usize = std::mem::size_of::<u32>();
let host_addr: SocketAddr = "127.0.0.1:7000".parse()?;

let mut sess = P2PSpectatorSession::new(num_players, input_size, local_port, host_addr)?;
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

## SyncTestSession

Used mainly for debugging purposes, this session simulates a configurable amount of rollbacks each frame. This is a great way to test if your game updates deterministically. The `SyncTestSession` will compare checksums between original and resimulated gamestates and raise an error if something went wrong. `check_distance` specifies how many frames of rollback you want to induce every frame. This value cannot exceed the maximum predicition frames allowed. Currently, 7 is the maximum distance of rollback that can occur.

```rust
let check_distance : u32 = 7;
let num_players : u32 = 2;
let input_size : usize = std::mem::size_of::<u32>();

let mut sess = SyncTestSession::new(num_players, input_size, check_distance)?;
```

Afterwards, you can optionally define input delays for any player you wish.

```rust
let input_delay = 2;

sess.set_frame_delay(input_delay, 0)?;
sess.set_frame_delay(input_delay, 1)?;
```

You don't need to define players or start the session. Because there are no remote clients to synchronize with, a `SyncTestSession` will immediately be ready to advance the game.
