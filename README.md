# TCI Virtual Audio Cable

<img src="icon.png" width="96" align="right" alt="">

Connects the audio of a **TCI** host (the TCI server of your SDR software) to normal Windows audio
devices, so that digital-mode programs can use it without a third-party virtual audio cable:

| Windows device | Kind | What it carries |
|---|---|---|
| **TCI RX 1**, **TCI RX 2**, ... | Recording | Receive audio of receiver 1, 2, ... |
| **TCI TX** | Playback | Audio to transmit |

Transmit can be keyed by **VOX**, which is **off by default**: when you switch it on, the bridge turns the
transmitter on as soon as audio is played to "TCI TX", and off again shortly after the audio ends.

> **This is a test version.** Read [What "test version" means](#what-test-version-means) before you
> install it. Use it on a **test computer or in a virtual machine**, not on the PC you depend on.

## The TCI host and this tool can run on different computers

The TCI host (your SDR software) and the TCI Virtual Audio Cable do **not** have to run on the same
PC: the bridge connects to the host by its address. That is why a **virtual machine is the ideal way to
try it**: the SDR software keeps running on your normal PC, and the driver, the test mode and the
digital-mode program live in the VM. If something goes wrong, you throw the VM away.

> **Only tested on a local network.** So far the tool was tested with the TCI host on the same PC or in
> the same local network (a virtual machine on the host PC). **Do not use it over the internet.**
> The TCI connection is unencrypted and has no login, so never open the TCI port of your SDR software
> to the internet or to a network you do not trust. Delay and dropouts on a long-distance link are
> untested and likely to break digital modes, whose timing is strict.

## Requirements

- Windows 11, 64 bit
- Administrator rights
- A TCI host that is reachable from the PC, on the same PC or in the same local network (TCI server
  enabled in the SDR software, default port 50001). Not for use over the internet.
- A valid amateur radio licence: see the [licence](#licence)

## What "test version" means

Windows only loads kernel drivers that are signed by Microsoft. This version is **not**. It is signed
with a self-made test certificate, and this has consequences:

- Windows must run in **test signing mode**. The installer offers to turn it on (`bcdedit`). It needs a
  restart, and **Secure Boot must be off** in the firmware. A "Test Mode" watermark appears on the desktop.
- Test signing lowers the security of the computer. Some programs (some anti-cheat software, company
  management software, BitLocker setups) do not like it or refuse to run.
- The installer adds the test certificate to the trusted certificates of the computer. The uninstaller
  removes it again.
- A driver bug can crash Windows (blue screen). Back up first, and best try it in a virtual machine.

If enough people are interested, a Microsoft-signed version is planned. That is not free for the author,
so it depends on the interest in this tool.

## Install

1. Download `TciVirtualAudioCable-Setup-<version>-testsigned.exe` from the
   [Releases](../../releases) page.
2. Run it (it asks for administrator rights) and accept the licence agreement.
3. Enter the **TCI host** (name or IP address), the **port** and how many receivers should get a
   recording device (1 to 8). Choose whether **VOX** should key the transmitter.
4. If test signing is off, the installer offers to turn it on. Restart Windows when the installer asks
   for it.

After the restart the devices **TCI RX n** and **TCI TX** are in *Settings > System > Sound* (or
`mmsys.cpl`), and the service **TciBridge** connects to the TCI host by itself. It starts with Windows and
reconnects when the host goes away.

## Settings

Start menu > **TCI configuration**:

- TCI host and port
- Receivers (0 = as many as the driver has) and the number of **RX endpoints**
- **Sample rate** of the recording devices ("RX sample rate") and of the playback device ("TX sample rate")
- VOX: threshold (dB below full scale), hang time (ms) and the transmitter number

*Apply* saves the settings and restarts what needs it. A change of the RX endpoints or of a sample rate
re-creates the driver device, which takes a few seconds. The dialog also shows the state of the service
and can start, stop, install and remove it.

For automatic installations, the installer accepts
`/VERYSILENT /HOST=<ip> /PORT=50001 /ENDPOINTS=2 /VOX=1`, and optionally `/RXRATE=48000 /TXRATE=48000`.

## Sample rate

Each device works at one fixed sample rate (stereo, 16 bit). Choose it in the configuration:

| Rate | Note |
|---|---|
| 8000, 12000, 24000, 48000 Hz | The rates a TCI host streams. The audio passes through unchanged. |
| 44100 Hz | For programs that want it. The audio is converted from and to 48 kHz. |

The default is **48000 Hz**. All "TCI RX n" devices share one rate, "TCI TX" has its own. The TCI host does
not deliver audio above 48 kHz, so a higher rate would add nothing. A low rate limits the audio bandwidth
to half the rate (8000 Hz: 4 kHz). Windows converts for programs that want another rate.

Select "TCI RX n" as the input and "TCI TX" as the output in your program. Play a test tone into a dummy
load before you transmit, and keep the output level so that the transmit power stays within your limits.

## If something does not work

- **No devices after the installation:** test signing on? (Command prompt as administrator:
  `bcdedit /enum {current}`, line *testsigning* must say *Yes*), Secure Boot off, Windows restarted.
- **Devices there, but no audio:** in *TCI configuration* check host and port, and that the TCI server is
  enabled in your SDR software. The service state must say *running*.
- **Log files:** `%ProgramData%\TciVirtualAudioCable\bridge.log` (bridge) and `install.log` (installer).
- Report problems in the [Issues](../../issues) with the log and your Windows version.

## Uninstall

*Settings > Apps > Installed apps > TCI Virtual Audio Cable > Uninstall.* This removes the service, the
device, the driver, the test certificate and the settings. Test signing stays on: to turn it off, run
`bcdedit /set testsigning off` as administrator and restart.

## Licence

Copyright (C) 2026 HB9DUT. Free of charge for **licensed radio amateurs**, for **non-commercial amateur
radio use only**. No sale, no redistribution and no inclusion in software collections without the
explicit written consent of the author. Provided **"as is", without any liability**. The full agreement
is in [EULA.txt](EULA.txt), the installer shows it too.

The driver contains portions of the Microsoft ACX audio codec sample (Copyright (c) Microsoft
Corporation, Microsoft Public License).
