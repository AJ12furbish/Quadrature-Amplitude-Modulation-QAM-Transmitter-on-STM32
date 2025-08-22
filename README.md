# Quadrature-Amplitude-Modulation-QAM-Transmitter-on-STM32
Realtime DSP Lab combining embedded C, DSP algorithms, and MATLAB post-processing, this project implements a complete digital communication chain from transmitter to receiver.

# QAM Transmitter-Receiver System

## Overview
This project implements a **Quadrature Amplitude Modulation (QAM) transmitter on an STM32 embedded system** in C, based on concepts from *Software Receiver Design: Build Your Own Digital Communication System in Five Easy Steps*. The design simulates digital baseband processing, modulation, channel effects, and demodulation. A binary data stream (representing a scrambled tree image) is mapped to in-phase (I) and quadrature (Q) symbol components, pulse-shaped using a raised cosine filter, and transmitted as an audio waveform. The transmission can be recorded and decoded in MATLAB to reconstruct the original image.

The implementation demonstrates:
- Symbol mapping (bits → constellation points)
- QAM modulation (I/Q generation)
- Pulse shaping with Root Raised Cosine filters
- Transmission through a noisy channel (AWGN)
- Receiver synchronization, matched filtering, and demodulation
- Bit error rate (BER) performance evaluation

---

# Learning Outcomes
This project demonstrates:
- Practical implementation of a **QAM transmitter on STM32**
- **Bitstream-to-I/Q** mapping and symbol synchronization
- Use of **raised cosine filtering** for pulse shaping
- Embedded DSP using the **ARM CMSIS library*
- MATLAB-based **signal capture, spectrogram analysis, and image recovery**

## Block Diagram
Below is the high-level block diagram of the system:

```
Bits → Symbol Mapper → Pulse Shaping → QAM Modulator → Channel → QAM Demodulator → Matched Filter → Symbol Decision → Bits
```

# Data Stream: Tree Image Transmission
The transmitted data stream contains: 
- 32-bit header generated from a maximal-length 5-bit LFSR (0x967c6ea1)
- 16384-bit payload representing a scrambled tree image, stored in tree[512]
The data is packed into an array of 513 words (32 bits each):
```
uint32_t data_stream[513] = {0};
```
The header initializes synchronization, while the payload encodes the tree bitmap for later recovery.

# Pulse Shaping with Raised Cosine Filter
To control bandwidth and reduce intersymbol interference, a **raised cosine filter** is applied to the symbols before transmission.

**Filter parameters:**
- Rolloff factor: **0.8**
- Number of Symbols: **4**
- Upsampling factor: **16 samples/symbol**
- Number of taps: **64**

The coefficients were generated in MATLAB using:
```
pulse_shaping_coeffs = rcosdesign(0.8, 4, 16, 'normal');
```

# Symbol Mapping and Modulation
The bitstream is unpacked into **I/Q symbol pairs**:
- Each bit is mapped to {–1, +1}
- Even bits → In-phase component (I)
- Odd bits → Quadrature component (Q)
This produces the QAM baseband signal ready for DAC output.

# MATLAB Signal Retrieval
The transmitted waveform can be recorded at **48 kHz in MATLAB**:
```
fs = 48000;
recorder = audiorecorder(fs,16,1);
recordblocking(recorder, 12);
QAM = getaudiodata(recorder)';
save QAM QAM;
```

# Post-processing steps:
1. **Spectrogram analysis**
   ```
   load QAM;
   figure; spectrogram(QAM,2^10,0,2^10,48000,'yaxis');
   ```
2. **Receiver demo**
   Run the QAM receiver in MATLAB, adjust synchronization parameters, and reconstruct the transmitted tree image.

# Key Files and Functions
- lab_init()
-   Loads the header and tree image data into data_stream.
-   Initializes FIR interpolation filters with raised cosine coefficients.
- process_input_buffer()
-   Converts bits into I/Q symbols.
-   Performs upsampling and raised cosine filtering.
-   Combines I/Q components with sine/cosine lookup for transmission.
- process_left_sample() / process_right_sample()
-   Sample-level passthrough with cycle measurement (for efficiency measurement).
- process_output_buffer()
-   Final access point for frame processing (left unchanged in this implementation).










