+++
title = "Introducing: Sparse Saving Feature"
description = "Explaining the sparse saving feature in GGRS and how it works."
date = 2021-08-06T12:00:00+00:00
updated = 2021-08-06T12:00:00+00:00
draft = false
template = "blog/page.html"

[taxonomies]
authors = ["Georg Schuppe"]

[extra]
lead = "Explaining the sparse saving feature in GGRS and how it works."

+++

GGRS just released version [0.4.1](https://github.com/gschup/ggrs/releases/tag/v0.4.1) and with it comes the sparse saving feature. In this article, I am going to briefly explain what it does and if you should consider using it.

## How things were before

If you were using GGRS before 0.4.1, updating, loading and saving the game state would occur exactly as often as they would in GGPO. Loading is done once per rollback, but saving is done after every update step. This is how a rollback looks like in GGPO:

```
[LOAD]
[UPDATE]
[SAVE]
[UPDATE]
[SAVE]
[UPDATE]
[SAVE]
```

By keeping a full history of gamestates, GGPO can always load the most recent and correct state. You could say that **GGPO minimizes *update steps*.**

## Saving and Loading Game State

Depending on your application, saving and loading the game state can look wildly different, but one often used technique is serialization, for example with Rust's popular [serde](https://serde.rs/). In fact, GGRS's own examples use `serde` and `bincode` in order to save and load the gamestate. While very convenient, this usually results in long computation times when saving multiple times in a single step, as in a rollback shown above.

## Sparse Saving Mode

As a solution, GGRS offers a *sparse saving* mode for `P2PSession`. It is easily enabled before starting the session.

```rust
sess.set_sparse_saving(true)?;
```

That's it. You don't need to change anything else in your code. **When in sparse saving mode, GGRS minimizes *save requests*** at the cost of additional update steps.

This works by only ever saving the latest confirmed frame, for which we have received and confirmed input from all clients. This state is guaranteed to be correct and can be used as a loading point for every rollback. If no *natural* rollbacks occur (due to all predictions being correct or all inputs arriving on time), we sometimes might need to artificially induce a rollback in order to resimulate the last confirmed state and save it.

With sparse saving, GGRS guarantees to save at most once per update tick, at the cost of potentially longer rollback sequences. In the [Rapier Synctest Example](https://github.com/gschup/ggrs/tree/main/examples/rapier), a single update step costs less than 0.5ms, while saving costs up to 3ms of time (due to the whole physics pipeline being serialized, which is definitely not optimal). In a hypothetical P2P example, this disparity makes the rapier example a prime candidate for the sparse saving feature.

## Conclusion

Depending on your application, the sparse saving mode might be beneficial for you. Update GGRS to the latest version 0.4.1, enable *sparse saving* and give it a try! If it made any difference for you, let me know!
