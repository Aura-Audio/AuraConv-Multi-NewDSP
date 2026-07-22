# AuraConv-Multi-NewDSP

### AuraConv Multi-Tap Resonator 🌊

![Platform](https://img.shields.io/badge/Platform-Web%20Audio%20API-blue?style=flat-square)
![Category](https://img.shields.io/badge/Category-Interactive%20DSP-orange?style=flat-square)
![Version](https://img.shields.io/badge/Version-1.0.0-success?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)

AuraConv Multi-Tap Resonator is a browser-native, high-performance sound design tool that translates the highly optimized AuraConv moving-average filter into an interactive single-page web application. 

By splitting a live microphone input across **25 parallel DSP tracks**—ranging from 25 to 100,001 taps—it allows users to sculpt transients, create dense acoustic blurs, and generate ethereal ambient washes directly in any modern web browser.

- Main - https://auraconv-multi-newdsp.pages.dev/
- v1 - https://0fa4310b.auraconv-multi-newdsp.pages.dev/
- v2 - https://fda0262b.auraconv-multi-newdsp.pages.dev/
- v3 - https://77beef2d.auraconv-multi-newdsp.pages.dev/
- v4 - https://26c55732.auraconv-multi-newdsp.pages.dev/
- v5 - https://4d0c2bb1.auraconv-multi-newdsp.pages.dev/
- v6 - https://682ffc51.auraconv-multi-newdsp.pages.dev/ - WASM
- v7 - https://3a865b9d.auraconv-multi-newdsp.pages.dev/ - WASM
---


# AuraConv Multi-Tap Resonator — v8.0.0 🌊

![Platform](https://img.shields.io/badge/Platform-Web%20Audio%20API-blue?style=flat-square)
![Category](https://img.shields.io/badge/Category-DSP%20Audio%20Effect-orange?style=flat-square)
![Version](https://img.shields.io/badge/Version-8.0.0-success?style=flat-square)
![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)

**AuraConv Multi-Tap** is a browser-native, highly optimized parallel-path acoustic resonator and sound design engine written in native JavaScript. 

Version 8.0.0 represents a complete paradigm shift, transforming the core algorithm from a static temporal blurring filter into an **active IIR Modal Resonator**. By introducing a real-time feedback matrix and dynamic spectral weighting across **150 independent parallel channels** (scaling up to **1,000,001 taps**), AuraConv can turn simple transients—like claps, whispers, or room noise—into lush, tuned acoustic chambers, physical-modeling string decays, and cavernous ambient drones.

---

## ✨ Key Features

* **Active Feedback Matrix (Modal Resonance):** Feeds a portion of each channel's output back into its own delay line. This converts the traditional FIR moving average filter into an **IIR string-model resonator** (similar to Karplus-Strong synthesis), producing organic, organ-like ringing chords.
* **Timbre Tilt (Spectral Balance):** Dynamically applies a real-time linear weighting slope across the 150 channels. Shifting to **"Bright"** accentuates shorter, transient-softening tap sizes; shifting to **"Dark"** emphasizes deep, cavernous abyssal washes.
* **150-Channel High-Resolution Grid:** Features a highly dense, responsive 5-column mixer board allowing independent gain, dry/wet, and active routing control over 150 logarithmically spaced channels.
* **1,000,001 Tap Ceiling:** Scales up to **20.8 seconds of linear-phase group delay** (at 48kHz), working as an organic, slowly evolving drone generator.
* **Dynamic Buffer Allocation:** Allocates RAM strictly based on each channel's individual tap size (rounded up to the nearest power of two), reducing the overall memory footprint of the 150-channel grid to **~234 MB** [1].
* **Dynamic Audio Graph Routing:** Nodes are physically disconnected (`node.disconnect()`) from the Web Audio thread when muted, ensuring inactive channels consume **0% CPU processing overhead**.
* **Mobile loudspeaker Rerouting Fix:** Implements modern `AudioSession` play-and-record profiles paired with a Web Audio asynchronous "jolt" (`suspend` / `resume` loop) to bypass earpiece routing, forcing output directly to the loud device speakers on iOS and Android.

---

## 🔬 DSP Architecture Under the Hood

### 1. The Active IIR Feedback Loop
To achieve stable, highly musical physical-modeling resonance, AuraConv v8.0.0 implements an in-loop decay matrix within the custom `AudioWorkletProcessor`:

```javascript
// Feed back previous sample's processed wet signal into the current input
const inL = inputL[i] + (this.prevWetL * feedback);
const inR = inputR[i] + (this.prevWetR * feedback);

this.memL[this.writePos] = inL;
this.memR[this.writePos] = inR;

this.sumL += inL;
this.sumR += inR;

// [Standard Moving Average Math]
const wetL = (this.sumL / size) * gain;
const wetR = (this.sumR / size) * gain;

// Cache current wet output for the next feedback calculation
this.prevWetL = wetL;
this.prevWetR = wetR;
```
Because the feedback parameters are bounded strictly to `0.95`, the loop gain remains safe and acoustically stable across all 150 parallel pipelines.

### 2. Linear Spectral Weighting (Timbre Tilt)
Adjusting 150 channels individually for tonal balance is resolved through a zero-CPU overhead weighting curve applied dynamically during master gain calculations:
```javascript
const progress = idx / 149; // Range: 0 (shortest) to 1 (longest)
if (tiltVal > 50) {
  // Bright tilt: attenuate long taps, preserve short taps
  const factor = (100 - tiltVal) / 50; 
  weight = 1 - progress + (progress * factor);
} else {
  // Dark tilt: attenuate short taps, preserve long taps
  const factor = tiltVal / 50; 
  weight = progress + ((1 - progress) * factor);
}
const finalGain = rawGain * weight;
```

---

## 🎛️ Acoustic Classifications & Grid Layout

The 150 parallel channels are categorized into five distinct color-coded bands to streamline the sound design process:

* **Soften (25 – 250 Taps / Blue):** Sharp, immediate windows designed to soften harsh transients on percussion or vocal sibilance.
* **Warmth (251 – 2,000 Taps / Amber):** Smooth, mid-frequency muffling that highlights fundamental harmonics and vocal warmth.
* **Blur (2,001 – 20,000 Taps / Purple):** Moderate time-smearing, blending distinct sounds into cloud-like textures.
* **Wash (20,001 – 150,000 Taps / Emerald):** Massive temporal blur, turning sound into an acoustic ambient chamber.
* **Abyss (150,001 – 1,000,001 Taps / Cyan):** Cavernous, slow-building linear delay structures that create infinite drone beds.

---

## 🚀 Getting Started

1. Download the `auraconv_abyss.html` file.
2. Double-click the file to open it in any modern web browser (Chrome, Edge, Safari, Firefox).
3. Connect headphones (highly recommended to prevent microphone feedback loops).
4. Click **Activate Microphone** and begin exploring.

---

## 📜 License & Contributing
Licensed under the MIT License. Pull requests and DSP optimizations are welcome!
