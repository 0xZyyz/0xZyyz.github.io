---
layout: post
title: "USB Power Leak — exfiltrating a flag through the VBUS line"
date: 2026-09-22 21:00:00 +0100
categories: [ctf, hardware, forensics, writeups]
role: author
---

This is the author write-up for **TX**, a hardware / signal-processing challenge I made. The idea: a "keyboard" that leaks data not over USB, but through tiny pulses on the **5 V power line** of the USB port. I'll go through the story, the traps I set, how the signal is built, and how to get it back. I'll also show why the obvious approach doesn't work.

---

## 1. The scenario

> A suspicious USB device was found plugged into a secure workstation. Externally it looked like a standard keyboard, but it contained extra electronics including a large capacitor bank. Your task: recover the data that was exfiltrated via a clever side-channel.

This isn't made up out of nothing. Air-gap and power side-channels are a real research topic: devices that modulate their current draw so that something else on the same supply line can read the data. The capacitor bank in the story is the "transmitter". It dumps small bursts of charge onto VBUS, and each burst shows up as a short voltage pulse.

Players got two files:

- `capture.sal`: a **Saleae Logic 2** capture with 4 unlabeled channels (2 digital @ 1 MS/s, 2 analog @ 10 kS/s)
- `usb_traffic.pcap`: USB bus traffic from a "USB analyser"

A `.sal` file is just a ZIP. Rename it and you get `meta.json`, `digital-0.bin`, `digital-1.bin`, `analog-2.bin`, `analog-3.bin`. The analog samples are `int16` and map to volts using the ranges in `meta.json` (`fullScaleVoltageRanges`). You can open it in Logic 2 directly, or parse it yourself.

---

## 2. The rabbit holes (on purpose)

Two of the three data sources are there to waste your time.

**The pcap.** It's a real-looking HID keyboard capture. If you decode the key reports you get:

```
the quick brown fox jumps over the lazy dog
all your base are belong to us
```

No flag. The keyboard really *is* just a keyboard. There's one useful detail though: the host polls the device **every 10 ms**. Keep that number in mind.

**The digital channels (Ch0/Ch1).** They're D+ and D−, and they look like USB full-speed traffic: SYNC, SOF packets, an IN / DATA0 / ACK every 16 frames. The DATA0 payload is all zeros. It's filler that looks like USB, and it has nothing to find.

The data is in the analog channels.

---

## 3. Finding the signal

Ch2 sits around **5.00 V**, so it's **VBUS**. Ch3 sits around **0 V**, so it's **GND**. The challenge text says one of them "shows unusual fluctuations".

![raw analog channels](/assets/posts/usb-power-leak/raw-channels.png)

Ch2 and Ch3 over the whole capture, then VBUS zoomed to 100 ms
{: .img-caption}

Over the whole 10 s it looks like noise on both channels. Zoomed in, VBUS has a slow wave (50 Hz mains hum), random noise, and some short spikes that show up at a regular rhythm. The spikes are only about **30 mV** tall on a 5 V rail, and the noise is about the same size. That's the problem this challenge is built around.

Two more clues:

- GND has the **same 50 Hz hum** in it. VBUS and GND are correlated (r ≈ 0.55), so the hum is common-mode interference that you can subtract.
- The spikes repeat on a **10 ms** grid, the same as the pcap's poll interval.

---

## 4. How the data is encoded

There are three layers, from the voltage up to the flag.

### Layer 1: pulse-position modulation (PPM)

Time is split into **10 ms slots**. Every slot gets exactly one pulse. The information is *where* the pulse is inside the slot:

- pulse centred at **2 ms**, symbol **0**
- pulse centred at **7 ms**, symbol **1**

![PPM symbols](/assets/posts/usb-power-leak/ppm-symbols.png)

One symbol per 10 ms slot. The pulse is a Ricker wavelet (τ = 1 ms, 30 mV)
{: .img-caption}

The pulse is a **Ricker wavelet** (the "Mexican hat"): a positive peak with two small negative lobes on the sides. This matters later.

```python
def ricker_pulse(t, t_centre):
    dt = (t - t_centre) / PULSE_TAU          # PULSE_TAU = 1 ms
    return PULSE_DROP * (1 - dt*dt) * math.exp(-dt*dt / 2)   # PULSE_DROP = 30 mV
```

### Layer 2: Manchester

Each data bit becomes two PPM symbols:

| bit | symbols |
|-----|---------|
| 0   | `1 0`   |
| 1   | `0 1`   |

Manchester guarantees a transition in every bit, and it lets you check yourself: a pair like `00` or `11` means a symbol was read wrong.

### Layer 3: the frame

```
[ 64-bit preamble 1010… ][ sync 0xEB 0x90 ][ flag bytes ][ CRC-8/MAXIM ]
```

The preamble gives you a clean alternating pattern to lock onto. `0xEB90` is a classic telemetry sync word. The CRC tells you whether your decode is actually right.

That's 64 + 16 + 39·8 + 8 = 400 bits, so **800 symbols**, so **8 seconds** of signal. This matches the full-capture plot: the noise band gets quieter after the 8 s mark, where the transmission ends.

---

## 5. Why the obvious approach fails

The first thing most people try: for each 10 ms slot, find the extreme sample and check which half of the slot it's in. With the pulses described as "fluctuations" or "drops", that usually means `argmin`:

```python
def window_min_decode(signal):
    syms = []
    for i in range(len(signal) // SYM_N):
        win = signal[i * SYM_N: (i + 1) * SYM_N]
        syms.append(0 if np.argmin(win) < SYM_N // 2 else 1)
    return syms
```

It gives garbage. Even after cleaning the signal (next section), it gets **252 of the 1000 symbols wrong**. The histogram shows why:

![offset histogram](/assets/posts/usb-power-leak/offset-histogram.png)

Where the "pulse" lands inside each slot: argmin on the raw signal vs. matched filter on the cleaned signal
{: .img-caption}

On the left, the minima are spread across the whole slot. Two things work against it:

1. **The pulse is a bump, not a dip.** The only negative parts of a Ricker wavelet are its side lobes, about 13 mV deep, which is close to the noise level. So `argmin` mostly lands on noise and on the hum.
2. **One sample is a weak decision.** At 10 kS/s a pulse is only ~20 samples wide. Choosing a single sample throws away the other 19.

On the right, after proper processing, you get two sharp clusters at **2 ms** and **7 ms**. This histogram is also how a player finds the PPM positions without knowing them in advance.

---

## 6. The solve: clean up, then use a matched filter

![signal chain](/assets/posts/usb-power-leak/signal-chain.png)

The same 100 ms window at each stage of processing
{: .img-caption}

### Step 1: common-mode rejection with GND

The hum is on both channels, so estimate how much of GND is in VBUS (least squares) and subtract it:

```python
def apply_cmr(vbus, gnd):
    v = vbus.astype(np.float64); g = gnd.astype(np.float64)
    v -= np.mean(v); g -= np.mean(g)
    alpha = np.mean(v * g) / np.mean(g * g)   # ≈ 0.40 here
    return v - alpha * g
```

This is why the GND channel is in the capture: it's the reference. It's the same idea as a differential probe.

### Step 2: a 50 Hz notch

CMR removes most of the hum, but GND has its own noise, so some hum stays. A narrow IIR notch at 50 Hz removes the rest without touching the ~1 ms pulses:

```python
def notch_50hz(signal):
    w0 = 2.0 * math.pi * 50 / A_SR
    alpha = math.sin(w0) / 60.0          # Q = 30, very narrow
    b = np.array([1.0, -2.0 * math.cos(w0), 1.0])
    a = np.array([1.0 + alpha, -2.0 * math.cos(w0), 1.0 - alpha])
    ...
```

After these two steps you can see the pulses by eye (panel 3), but single samples are still noisy.

### Step 3: matched filter

When you know the shape of the pulse, the best linear detector in white noise is to **correlate with that shape**. It uses every sample of the pulse, not just one. Measure the pulse width from a zoomed plot (~1 ms), build the template, and correlate:

```python
def make_ricker_template():
    half = int(3 * PULSE_TAU * A_SR)
    n = np.arange(-half, half + 1) / (PULSE_TAU * A_SR)
    h = (1 - n * n) * np.exp(-n * n / 2)
    return h - np.mean(h)

mf = np.correlate(signal - np.mean(signal), templ, mode='same')
peaks = [t0 + np.argmax(mf[t0:t0 + SYM_N]) for t0 in range(0, n_syms * SYM_N, SYM_N)]
```

Panel 4 shows the result: one clear peak per slot. Then compare each peak's offset to a **4.5 ms** boundary (halfway between 2 and 7 ms, a bit early to allow for jitter), decode the Manchester pairs, find `0xEB90`, and check the CRC.

### Result

```
window-min:
  FAILED
matched filter:
  Securinets{usb_p0w3r_l1n3_3xf1ltr4t10n}
kalman+MF:
  Securinets{usb_p0w3r_l1n3_3xf1ltr4t10n}
```

The raw bytes after the sync word:

```
eb 90  Securinets{usb_p0w3r_l1n3_3xf1ltr4t10n}  a4
                                               └─ CRC-8/MAXIM, matches
```

### Alternative: Kalman + matched filter

My solver also has a second path. A small 2-state Kalman filter tracks the slow baseline (hum and drift), and its normalised innovation, meaning "what the model didn't expect", goes into the same matched filter. It removes the baseline adaptively instead of with a fixed notch. It gets the same flag. It's more than this challenge needs, but it's useful when the interference isn't a clean 50 Hz.

---

## 7. Takeaways

- **Look at every channel, but decide which one matters.** The pcap and the digital lines are there so you look at the wrong layer. The physical layer is the one that carries the data.
- **Know the pulse shape before choosing a detector.** Min/max per window is fine when the signal is strong. When the pulse is about the same size as the noise, a matched filter is what makes it work.
- **Use the reference channel you're given.** GND in the capture is there to be subtracted.
- **Use the error checks.** Manchester pairs and the CRC tell you when a decode is right, so you're not guessing.

Thanks to everyone who played. If you solved it a different way, message me, I'd like to see it.
