# AuraConv-Multi-NewDSP

### AuraConv Multi-Tap Resonator 🌊

![Platform](https://img.shields.io/badge/Platform-Web%20Audio%20API-blue?style=flat-square)
![Category](https://img.shields.io/badge/Category-Interactive%20DSP-orange?style=flat-square)
![Version](https://img.shields.io/badge/Version-1.0.0-success?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)

AuraConv Multi-Tap Resonator is a browser-native, high-performance sound design tool that translates the highly optimized AuraConv moving-average filter into an interactive single-page web application. 

By splitting a live microphone input across **25 parallel DSP tracks**—ranging from 25 to 100,001 taps—it allows users to sculpt transients, create dense acoustic blurs, and generate ethereal ambient washes directly in any modern web browser.

---

## ✨ Core Features

* **Browser-Native DSP:** Leverages the high-performance `AudioWorklet` API to run complex filter calculations in a dedicated background audio thread, separate from the UI.
* **25-Channel Parallel Matrix:** Features a modular mixer panel to independently control, blend, and mute 25 distinct odd-numbered tap lengths.
* **Phase-Aligned Dry/Wet Pathing:** Every channel automatically calculates its own group delay, delaying its dry signal internally to prevent comb-filtering and phase cancellation during parallel mixing.
* **Zero-Latency O(1) Complexity:** Implements a running-sum accumulator. Processing 100,000 taps requires the exact same CPU load during active playback as processing 25 taps.
* **Dynamic Parameter Smoothing:** Individual channels and master faders employ single-pole parameter smoothing to eliminate digital "zipper noise" and click artifacts.
* **Fully Standalone & Offline-Ready:** Packaged entirely inside a single, zero-dependency HTML file. Uses inline Blobed JS modules to bypass browser CORS limitations when running locally.

---

## 🔬 DSP Architecture under the Hood

This application implements several professional audio techniques directly in native JavaScript:

### 1. The AudioWorklet Thread Model
While standard JavaScript runs on the main browser thread (which handles rendering, scrolling, and UI interactions), AuraConv registers a custom `AudioWorkletProcessor`. This runs on a highly prioritized, real-time background thread. The UI communicates with this thread via thread-safe `AudioParam` arrays.

### 2. Group Delay Alignment Math
A symmetric moving average (FIR) filter introduces a delay of precisely:
$$\text{Latency (samples)} = \frac{\text{Taps} - 1}{2}$$

Because all 25 channels utilize **odd numbers**, this latency is guaranteed to result in a whole integer. The dry signal for each channel is retrieved from the exact center of its delay line:
```javascript
const delaySamps = (size - 1) / 2;
const dryPos = (this.writePos - delaySamps) & this.mask;
const dryL = this.memL[dryPos];
```
This enables phase-accurate summing when blending the dry and processed signals.

### 3. Bitwise Circular Masking
Memory allocation and address wrapping are handled via a power-of-two bitwise mask (`& 131071`). This bypasses slow conditional branches (like `if` statements) in the main loop, optimizing memory performance.

---

## 🎛️ Acoustic Classifications & Grid Layout

The 25 parallel channel strips are dynamically categorized based on their physical tap length to aid in intuitive sound design:

* **Transient Softeners (25 – 201 Taps):** Sharp, punchy windows designed to soften the harshness of transients, useful for percussion and vocal consonants.
* **Mid-Range Warmth (251 – 1,501 Taps):** Smooth, mid-frequency blurs that muffle harsher frequencies and accentuate the fundamental warmth of vocals and instruments.
* **Cloudy Blurs (2,001 – 10,001 Taps):** Dense, reflective windows that smear physical separation, blending distinct sounds into singular, cloudy textures.
* **Ambient Washes (15,001 – 100,001 Taps):** Massive time-windows that act similarly to physical reverberation chambers, melting transients into seamless ambient drones and drones.

---

## 🚀 Getting Started

No installation, terminal commands, or hosting setups are required.

1. Download the `auraconv_app.html` file.
2. Double-click the file to open it in Chrome, Safari, Edge, or Firefox.
3. Plug in headphones (highly recommended to prevent microphone feedback loop).
4. Click **Activate Microphone**, allow browser mic access, and begin.

---

## 📜 License

This project is licensed under the MIT License - see the LICENSE file for details.
