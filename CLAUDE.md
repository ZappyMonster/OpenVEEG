# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

OpenVEEG is a multi-language neuroscience data acquisition system for synchronous EEG and video recording in seizure-induced animal models. The system uses LabStreamingLayer (LSL) for cross-module synchronization.

## Architecture

```
Video Input (BlackMagic DeckLink)     EEG Input (NI DAQmx)
         ↓                                    ↓
   OpenCV Processing                   PyDAQmx/MATLAB DAQ
         ↓                                    ↓
   AVI Encoding → Disk                 LSL Outlet Stream
         ↓                                    ↓
   LSL Outlet (timestamps)             LSL Inlets (consumers)
                                              ↓
                                    Real-time Visualization
```

### Module Structure

- **OpenVEEG_Video/** - C++ video capture from BlackMagic DeckLink cards, encodes to AVI with LSL timestamp sync
- **OpenVEEG_Enceph/Read_LSL_EEG_python-master/** - Python NI DAQmx to LSL bridge for EEG acquisition
- **OpenVEEG_Enceph/Read_LSL_EEG_matlab-master/** - MATLAB equivalent of Python EEG reader
- **OpenVEEG_Plotter/** - Real-time EEG visualization (wxPython GUI and simple matplotlib versions)

## Build Instructions

### C++ Video Module (Windows)

Requires Visual Studio 2015 (v140 toolset). Open `OpenVEEG_Video/OpenVEEG_video/VideosStream.vcxproj`.

**Dependencies (configure paths in Project Properties):**
- OpenCV 3.1 (`opencv_world.dll` must be in executable folder)
- Boost 1.64+
- LabStreamingLayer (`liblsl64.dll` must be in executable folder)
- BlackMagic DeckLink SDK

### Python EEG Module

```bash
pip install numpy pydaqmx nidaqmx
python OpenVEEG_Enceph/Read_LSL_EEG_python-master/Read_LSL_EEG.py
```

### MATLAB EEG Module

Requires Data Acquisition Toolbox and liblsl-Matlab library.

## Key Configuration Points

**Video format (DeckLinkCapture.cpp:162):** Must match camera's BMD input format (default: BMD YUV 8bit)

**Video codec (VideosStream.cpp:98):** Change AVI codec (default: DivX). Also update DeckLinkInputCallback.cpp:44 to match.

**Grayscale mode (VideosStream.cpp:115-117):** Uncomment lines 115-116 and change 'frame' to 'greymat' on line 117.

**Video defaults:** 1920x1080, 30fps, DivX codec, color

## Runtime

All modules are interactive CLI applications requiring user input during execution. Production deployment uses Windows Task Scheduler with 4-hour recurrence via batch files. Output naming: `{date}_{start_time}_{mouse_number}`.

## Notes

- Python modules use Python 2 syntax (`raw_input()`, print statements)
- No test framework configured
- Pre-compiled Windows executable available at `OpenVEEG_Video/OpenVEEG_Video.exe`
