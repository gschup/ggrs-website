+++
title = "FAQ"
description = "Answers to frequently asked questions."
date = 2021-08-04T19:17:23+00:00
updated = 2021-08-04T19:17:23+00:00
draft = false
weight = 30
sort_by = "weight"
template = "docs/page.html"

[extra]
lead = "Answers to frequently asked questions."
toc = true
top = false
+++

## What is GGRS?

GGRS (Good Game Rollback System) is a reimagination of the [GGPO](https://www.ggpo.net/) P2P rollback networking library written in 100% safe Rust.

## What is GGPO / Rollback?

Taken from [the official GGPO website](https://ggpo.net/):

>Rollback networking is designed to be integrated into a fully deterministic peer-to-peer engine.  With full determinism, the game is guaranteed to play out the same way on all players computers if we simply feed them the same inputs.  One way to achieve this is to exchange inputs for all players over the network, only execution a frame of gameplay logic when all players have received all the inputs from their peers.  This often results in sluggish, unresponsive gameplay.  The longer it takes to get inputs over the network, the slower the game becomes.

>In rollback networking, game logic is allowed to proceed with just the inputs from the local player.  If the remote inputs have not yet arrived when it's time to execute a frame, the networking code will predict what it expects the remote players to do based on previously seen inputs.  Since there's no waiting, the game feels just as responsive as it does offline.  When those inputs finally arrive over the network, they can be compared to the ones that were predicted earlier.  If they differ, the game can be re-simulated from the point of divergence to the current visible frame.

>Don't worry if that sounds like a headache.  GGPO was designed specifically to implement the rollback algorithms and low-level networking logic in a way that's easy to integrate into your existing game loop.  If you simply implement the functionality to save your game state, load it back up, and execute a frame of game state without rendering its outcome, GGPO can take care of the rest.

For more information about GGPO, check out [the official website](http://ggpo.net/) or [the official github repository](https://github.com/pond3r/ggpo). A very good pseudocode explanation for general rollback networking can be found [in this Gist](https://gist.github.com/rcmagic/f8d76bca32b5609e85ab156db38387e9).

## Even more questions?

Send *Georg Schuppe* an E-mail:

- <georg.schuppe@gmail.com>
