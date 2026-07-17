# RAVE – Real-time Analysis of Vocal tract Resonances (Windows fork)

**A Windows-compatible, patched fork of the RAVE GUI** from the Acoustics Lab at UNSW Sydney (Joe Wolfe's lab), originally developed by Noel Hanna and colleagues. RAVE estimates vocal tract resonances in real time by injecting a broadband acoustic signal at the lips and analysing the response while a subject speaks or sings.

Original repository: [noelhanna/real-time-resonance-estimation](https://github.com/noelhanna/real-time-resonance-estimation)
Background on the technique: [UNSW Music Acoustics – vocal tract resonance measurement](https://newt.phys.unsw.edu.au/jw/broadband.html) and this article [Using visual feedback to tune the second vocal tract resonance for singing in the high soprano range] (https://www.tandfonline.com/doi/abs/10.1080/14015439.2020.1834612) 

This fork exists because the original code was developed and tested on macOS, and did not run correctly on Windows. The bugs below were diagnosed and fixed on Windows 10/11, and several were confirmed by correspondence with Joe Wolfe.

---
## What was fixed ( by Fable 5 ) 
1. Mismatched .fig/.m file pair
The GitHub repository contains GUI_RAVE.m, but the downloadable figure file is GUI_RAVE_ICPHS.fig, whose callbacks are all hardcoded to call GUI_RAVE_ICPHS. Fix: copied the .fig to GUI_RAVE.fig and created a small wrapper file GUI_RAVE_ICPHS.m that forwards all calls to GUI_RAVE, so every button callback resolves correctly.
2. Startup crash in OpeningFcn
cla(handles.Phase) failed on startup with the mismatched .fig (stale handles). Fix: wrapped in try/catch.
3. Playback device bug in the broadband signal button (CalculationPlayStopBBSignal_Callback)
audioplayer was created with DevID_Mic (an input device) as the output device — this happened to work on the original Mac setup but fails on Windows with "Could not find the specified device". Fix: the output device ID is now resolved from the audio output menu selection. This was the main reason no sound was produced in Measure mode.
4. Recorder object only created when the BB signal button was off
In pushbutton_BeginRecord_Callback (RealTime branch), handles.recorder was created only inside if get(handles.CalculationPlayStopBBSignal,'Value')==0, so pressing the speaker button first and Record second crashed with "Unrecognized field name 'recorder'". Fix: recorder creation moved outside the condition (always executed).
5. Broadband buffer length
The speaker-button playback buffer was hardcoded to 50 s (DureeBruit = 50) and the Record path to 60 s, causing sound to stop mid-session. Fix: extended to 300 s.
6. Bugs in the >60 s re-record loop
The continuation loop referenced an undefined variable ([s, audio] → corrected to [s, egg]) and read a non-existent 3rd audio channel (newDATA(:,3) — disabled, as only 2 channels exist without the EGG hardware modification described in the README). These caused a system error sound and crash at the 60-second mark.
7. Savitzky-Golay filter crash during singing (real-time display freeze)
In the harmonic-deletion section of mafonctionaffichage_v3_NEW, the polynomial order degree = Nplage-indW-3 (and Nplage-2) can become negative for small Harmonic-width settings relative to df. On modern MATLAB, audiorecorder silently swallows TimerFcn errors, so every displayed frame died quietly after the H/R update but before the spectrum plot — the magnitude display appeared frozen whenever the user sang. Fix: clamped both order computations with max(1, …) and wrapped the harmonic-deletion loop in try/catch so a failed cosmetic step can never abort the frame. Same clamp applied to the copy in DisplayChosenCurvesNEW (which crashed on Stop).
8. Dead Java range slider on modern MATLAB
CalculationStop_Callback reads get(hjRangeSlider,'Maximum'/'Minimum'); the JIDE Java component is invalid in recent MATLAB releases, crashing the Stop button. Fix: try/catch with fallback to handles.MaxValSlider / handles.MinValSlider (computed just above from the recorded data).
9. Display performance improvements

Removed a per-frame assignin('base','samples',…) debug call (was generating ~100 workspace events per second).
EGG and Audio waveform axes are now redrawn only every 3rd frame (spectrum, phase, and all R/H analysis still update every frame). Profiling showed axes recreation and Java-component image streaming dominated frame time.

Known remaining limitations (unchanged from the original): real-time mode is not sample-synchronized (phase display is approximate, as documented in the README); the JIDE range-slider based selection UI depends on deprecated Java components; the ACUZ synchronous mode requires a pa_wavplay build that is not included.

Not addressed (deliberately): minor GUI layout differences caused by Mac/Windows font metric differences (If someone can solve this, hats off!). These are cosmetic and do not affect functionality. 

## Installation

**Requirements:** MATLAB on Windows 10 or 11 (developed and tested with a standard desktop MATLAB installation), plus the audio hardware described below.

1. **Get the code** — clone this repository (branch `windows-fixes`) or download it as a ZIP:
   ```
   git clone -b windows-fixes https://github.com/R2f0/real-time-resonance-estimation.git
   ```
2. **Get the GUI figure files** — download `GUI_RAVE.fig` and `GUI_RAVE_ICPHS.fig` from the [Releases page](../../releases) and place them **in the same folder** as the `.m` files.
   *(They are hosted as release assets because each is ~138 MB, above GitHub's 100 MB in-repo file limit.)*
3. **Configure your audio devices** in Windows and in MATLAB so that the excitation signal goes to the amplifier/compression driver and the measurement microphone is the input device.
4. Open MATLAB, `cd` into the folder, and run:
   ```matlab
   GUI_RAVE
   ```

## Hardware setup used for testing

This fork was developed and verified with the following measurement chain:

- **Measurement microphone:** miniDSP UMIK-1 (USB, calibrated)
- **Audio interface:** Behringer UMC22
- **Vocal microphone:** Shure SM58
- **Excitation source:** compression driver + cheap amplifier module (later tried with Fossi ZA3) 
- **Acoustic delivery:** silicone tubing from the compression driver to the lips (5mm inner diameter 1m lenght), with a custom 3D-printed adapter

Other hardware should work, but device routing is exactly where the original Windows bugs lived — if you use a different setup, double-check which device the excitation signal is being sent to.

## Status and caveats

- This is a **working research tool**, not a polished product. It is provided as-is, with no warranty (see GPL v3 terms).
- Fixes were tested on Windows 10/11 with the hardware above. macOS behaviour has not been re-verified after the patches.
- Issues and pull requests are welcome, though maintenance is on a best-effort basis.

## Credits and license

- **Original authors:** Noel Hanna and the Acoustics Lab, School of Physics, UNSW Sydney (Joe Wolfe's lab). All credit for the RAVE method and original implementation belongs to them.
- **Windows port and bug fixes:** this fork (2026). Thanks to Joe Wolfe for confirming several of the bugs by email.
- **License:** GNU General Public License v3.0, inherited from the original project. See [LICENSE](LICENSE). This is a modified version of the original software; modifications are marked in the commit history and in this README, as required by the GPL.


