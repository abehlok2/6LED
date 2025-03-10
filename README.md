# DIY Audio-Visual Brainwave Entrainment

This project aims to create an open-source audio-visual brainwave entrainment system using a small single-board computer (SBC) such as a Raspberry Pi.  
It drives 6 high-power LEDs via a PCA9685 PWM board and MOSFETs, and optionally generates and plays synchronized audio entrainment tracks (e.g. binaural, isochronic).  
Users can design custom multi-step “sequences” (lighting + audio parameters) with a PyQt-based GUI editor, then run them on hardware.

---

## Table of Contents
1. [Overview](#overview)
2. [Features](#features)
3. [Hardware Components](#hardware-components)
4. [Tools Required](#tools-required)
5. [System Architecture](#system-architecture)
6. [Installation](#installation)
7. [Usage](#usage)
   1. [Running the GUI Editor](#running-the-gui-editor)
   2. [Generating & Running Sequences](#generating--running-sequences)
8. [GUI Overview](#gui-overview)
9. [Code Structure](#code-structure)
10. [Disclaimer](#disclaimer)
11. [License](#license)

---

## Overview
Brainwave entrainment (BWE) involves the use of pulsing lights or audio tones at specific frequencies in an attempt to influence brain states. This project allows you to experiment with various stimulation patterns:
- **Visual**: 6 strobing LEDs with adjustable waveforms, duty cycles, frequency ramps, and more.
- **Audio**: Up to 3 carriers that can be individually enabled, each with optional frequency ramps, random frequency modulation (RFM), binaural or isochronic modes, and pink noise.

The result is a flexible platform for exploring entrainment principles. The system is intended for DIY research and is **not** a medical device. Use at your own discretion.

---

## Features
- **PyQt5-based GUI** (in `sequence_editor.py`) for creating multi-step LED + audio sequences.
- **Advanced Oscillator Patterns (work in progress)**: Sine, square, and configurable “phase/brightness/timing” secondary patterns (e.g., “Sacred Geometry,” “Fractal Arc,” etc.).
- **Random Frequency Generation**: Configurable, slightly changes to the binaural to reduce habituation to a steady-state stimulus. 
- **Binaural Generation**: Can generate a matching, stepped binaural .wav file, synced to the LED "track" with up to 3 simultaneous binaurals. Supports isochronic tones + pink noise. Plays audio automatically when run on-device, if present. 
- **Linear Ramps**: Frequencies or duty cycles can transition over the duration of each step.
- **Strobe Intensity**: Per-step strobe intensity or crossfade.
- **JSON File Storage**: Save or load complete sequences, including audio settings, to a `.json` file.
- **Stepwise Audio Generation**: Automatic `.wav` generation matching the duration and frequency ramps of each step.

---

## Hardware Components
Below is the minimal parts list used for the LED driver side:

1. **SBC computer** (Raspberry Pi or comparable)
2. **PCA9685** 16-channel PWM driver
3. **3× High-power (~0.3 A × 3.3 V) cool white LEDs**
4. **3× High-power warm white LEDs**
5. **6× Switching MOSFETs** (e.g., IRFZ44N logic-level; or similar)  
   *(If they are not fully logic-level at 3.3 V, a 5 V level shifter or gate driver may be required)*
6. **1× External 5 V power supply** (capable of driving the 6 LEDs + the Raspberry Pi if desired)
7. **6× 2 W, 5 Ω resistors** (one per LED for current limiting)
8. **6x 10KΩ resistors** (pull-downs for MOSFET gates)
9. **6× 220 Ω resistors** (protects PWM pins)
10. **Wiring** (jumper wires, hookup wire, etc.)

### Typical Connections
- The PCA9685 connects via I²C (SDA, SCL) to the SBC (Raspberry Pi pins).
- PCA9685 outputs go to the MOSFET gates (with pull-down resistors to ensure they remain off when idle).
- MOSFET drains connect to each LED + current-limiting resistor in series, then to the 5 V supply rail.
- MOSFET sources go to ground.
- Make sure to share a common ground between the SBC, PCA9685, and the external 5 V supply.

---

## Tools Required
1. **Soldering iron + flux**
2. **Wire strippers**
3. **Multimeter** (optional but recommended for verifying current/voltage)
4. **Breadboard** (optional for prototyping)
5. **3D printer** (optional if you want a custom LED holder or enclosure)

##  Circuit Diagram
![test](https://github.com/user-attachments/assets/24c5af34-a95f-4e9e-9783-57645dcb86a8)

---

## System Architecture
1. **`sequence_editor.py` (PyQt5 GUI)**:
   - Allows you to add/duplicate/remove steps and configure oscillator waveforms, frequencies, random modulation (RFM), and more for each LED group or individual LED.
   - Manages global audio settings such as binaural mode, isochronic pulses, multiple carriers, pink noise, etc.
   - Saves the sequence + settings to JSON.
   - Optionally generates a `.wav` audio file matching the final sequence’s durations and frequencies.

2. **`sequence_model.py`**:
   - Contains the data classes (`Step`, `Oscillator`, `StrobeSet`, `AudioSettings`, etc.) that define how the LED and audio steps are stored, serialized, and reconstructed from JSON.

3. **`audio_generator.py`**:
   - Functions that generate mono or stereo waveforms based on your audio carrier setups.
   - Handles random frequency modulation, pink noise, binaural or isochronic calculations, etc.
   - Capable of stitching multiple step waveforms into a single `.wav` file for the entire sequence.

4. **`run_on_device_6.py`**:
   - A command-line runner that loads a JSON sequence (created by the editor) and physically drives the PCA9685 + LEDs, optionally playing the generated `.wav` file in sync.
   - Applies real-time oscillator logic: waveforms, frequency ramps, RFM, brightness patterns, etc.

---

## Installation
1. **Clone or download** this repository onto your development machine or Raspberry Pi.
2. Install required Python packages: 
   numpy – for audio waveform generation
   simpleaudio – for real-time audio playback
   adafruit-circuitpython-pca9685 – for driving the PCA9685 over I²C
   PyQt5 – for the GUI editor
   You may also need PyQt5-sip and other standard libraries for your Python environment.
   Enable I²C on your Raspberry Pi (if using Pi OS), then wire the PCA9685 accordingly.

GUI Overview
Below is a screenshot of the Sequence Editor GUI in "Split" mode (PyQt5):
![image](https://github.com/user-attachments/assets/db44b9f4-8f38-4098-b52d-18616c8a6409)


Left Panel: List of steps in the sequence. You can Add, Duplicate, Remove, or Reorder steps.
Step Info: Set the duration of the step and a short description.
Oscillator Mode:
Combined: 1 oscillator controlling all 6 LEDs.
Split: 2 oscillators (e.g., one for even-indexed LEDs, one for odd-indexed).
Independent: Each LED has its own oscillator.
Oscillator Settings: Choose waveform (Sine, Square, Off), frequency start/end, duty cycle start/end, random frequency modulation (RFM) parameters, and advanced pattern modulation.
Strobe Intensities: Overall intensity ramp for each group of LEDs.
Audio Tabs (right side): Up to 3 carriers, each with optional frequency ramps, RFM, volume control. The Global Settings tab includes enabling audio, binaural or isochronic mode, pink noise, etc.
