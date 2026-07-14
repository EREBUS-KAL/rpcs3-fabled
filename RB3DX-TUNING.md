# Rock Band 3 Deluxe tuning guide (rpcs3-fabled)

This fork carries latency- and stability-oriented changes aimed at rhythm games,
with Rock Band 3 Deluxe (RB3DX) as the primary target. This page documents the
fork-specific options and collects the community-recommended configuration in
one place.

## Fork-specific changes

### Audio: Low Latency Streaming (`Audio` tab, or `Audio: Low Latency Streaming` in config.yml)

RPCS3 normally requests audio stream periods of at least 512 samples (~10.7 ms)
from the audio backend. Modern audio stacks (WASAPI, PipeWire, CoreAudio) handle
smaller periods without problems on most hardware. With this option enabled the
Cubeb backend requests periods down to 256 samples (~5.3 ms, still clamped to
the minimum the backend reports as supported).

This matters twice:

1. The stream period itself is part of the output latency.
2. cellAudio's *effective* minimum for **Desired Audio Buffer Duration** is
   `stream period + ~10.7 ms`. With the standard period the real floor is
   ~21 ms no matter how low the slider goes; with Low Latency Streaming the
   floor drops to ~16 ms, so low slider values actually take effect.

Lower total audio latency reduces the vocal-monitoring echo and shrinks the
calibration offset the game has to compensate for. If you hear crackling that
**Enable Buffering** does not fix, turn this back off.

Default: **off** (identical behavior to upstream RPCS3).

### Input: USB transfer wake-up fix (always on)

Emulated USB instruments respond to the game's interrupt polls after a fixed
response time. Previously the USB handler thread only checked for due responses
on a free-running ~1 ms tick, so every single poll picked up a random extra
0–1 ms of latency depending on tick phase. The handler thread is now woken when
a transfer is submitted, so responses complete on time. This removes up to 1 ms
of *jitter* per poll for all emulated USB devices (and reduces average input
latency by ~0.5 ms).

### Input: Emulated USB instrument response time (`Input/Output: Emulated USB instrument response time (microseconds)` in config.yml)

The emulated Rock Band 3 MIDI Pro Adapter (guitar/drums/keys), Guitar Hero Live
guitar, and DJ Hero turntable complete each interrupt poll after a fixed
response time, 1000 µs by default (the upstream value). This option lets you
lower it to 100 µs, cutting up to ~0.9 ms of input latency. The real hardware
takes several milliseconds, so even the default is faster than a physical
adapter — games do not depend on the exact value.

Range: 100–1000, default 1000. Not exposed in the GUI; edit the config.yml or a
custom per-game configuration.

## Recommended RB3DX configuration

The settings below follow the [MiloHax custom configuration
guide](https://guides.milohax.org/) and the [RPCS3 wiki page for Rock Band
3](https://wiki.rpcs3.net/index.php?title=Rock_Band_3), plus the fork options
above. Use a *custom configuration* for the game (right click the game →
Create Custom Configuration From Global Settings) so other titles keep their
defaults.

### CPU

| Setting | Value | Why |
| --- | --- | --- |
| PPU / SPU Decoder | Recompiler (LLVM) | Default, fastest |
| SPU Block Size | Mega | Fewer, larger SPU compilations; helps lower-core-count CPUs |
| Preferred SPU Threads | 1–4 (only if you see stutter) | Limits simultaneous SPU work on CPUs with few cores |
| Thread Scheduler | RPCS3 Scheduler / Alternative | Reduces microstutter on 12+ thread CPUs |

### GPU

| Setting | Value | Why |
| --- | --- | --- |
| Renderer | Vulkan | Fastest |
| Write Color Buffers | **On** | Required — characters render incorrectly without it |
| VSync | Off (use in-game calibration instead) | Lower display latency |
| Frame limit | Auto or Off | Avoid double limiting |

### Audio

| Setting | Value | Why |
| --- | --- | --- |
| Enable Buffering | On | Prevents crackle |
| **Low Latency Streaming** | **On** (fork option) | Lower output floor, see above |
| Desired Audio Buffer Duration | Start at 32 ms, lower until crackle | Latency vs. stability |
| Enable Time Stretching | Off | Adds latency, hurts rhythm feel |
| Microphone Type | Standard (for vocals) | RB3 expects a standard USB mic |

### Advanced / Debug

| Setting | Value | Why |
| --- | --- | --- |
| Debug Console Mode | On | With RB3DX's "Large Heap" option: more memory, up to 16k songs, better stability |
| Driver Wake-up Delay | 20 µs (increase by 20 if crashing persists) | Known workaround for crashes after several songs, required when using RSX FIFO Accuracy = Fast |
| RSX FIFO Accuracy | Fast *only together with* Driver Wake-up Delay ≥ 20 µs | Small latency/perf gain, unstable without the delay |
| Sleep Timers Accuracy | As Host (Linux/macOS) / Usleep (Windows) | Default; governs timing jitter |

### Network

| Setting | Value | Why |
| --- | --- | --- |
| Network Status | Connected | Prevents freezing in the song library |
| PSN Status | RPCN (optional) | Online features |

## Sources

- [MiloHax Guides — Rock Band 3 on PC](https://guides.milohax.org/)
- [RPCS3 Wiki — Rock Band 3](https://wiki.rpcs3.net/index.php?title=Rock_Band_3)
- [RPCS3 issue #18463 — overdrive crackle regression (fixed upstream, included in this fork's base)](https://github.com/RPCS3/rpcs3/issues/18463)
