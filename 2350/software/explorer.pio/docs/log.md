# Change Log

## v2.28

- Bumped Explorer to v2.28.
- Re-enabled microSD MP3 discovery in Explorer, added MP3 list/detail UI with Play, Stop, Pause, Resume, and play counter controls, and wired the menu to the Pico MP3 decoder service.
- Prevented idle MP3 Stop/Pause/Resume commands from lazy-starting Core 1 during Explorer startup or normal flash browsing.
- Moved Explorer I2S audio to PIO1 SM2 and removed MP3 teardown of PIO0 so MP3 selection cannot disrupt the live MSX bus PIO programs.
- Removed the unused MP3 now-playing memory window that overlapped the Explorer ROM/query exchange area and could corrupt the MP3 detail screen.
- Rendered MP3 action rows from fixed literals instead of a RAM pointer table to avoid corrupted action labels on the MSX detail screen.
- Refreshed the MP3 play counter from the MSX JIFFY timer, cleared the menu shortcut line on the MP3 detail page, and stopped playback when leaving MP3 details with ESC.
- Removed the unavailable MP3 total-duration field from the detail screen and expanded the MP3 name display to use the full 80-column line.
- Restored the MP3 elapsed play time and playback status to the detail page content area while keeping the normal menu shortcut hints hidden.
- Blocked File Hunter downloads above the 4 MB microSD/PSRAM launch limit used for SD-loaded ROMs.
- Kept the I2S DAC mute line asserted while Explorer is idle, only unmuting it after MP3 playback or a ROM audio profile starts.
- Simplified MP3 detail actions into Play/Stop and Pause/Resume toggle rows that follow the current playback state.
- Added a ROM detail PSG option that mirrors primary PSG writes to the Pico DAC and mixes with existing ROM audio profiles.
- Updated Explorer, public, feature, Dual PSG, and PIO documentation for the MP3 player and primary PSG DAC mirroring behavior.

## v2.27

- Bumped Explorer to v2.27.
- Raised the MSX-MUSIC/FM-PAC audio output gain to match the existing SCC, SCC+, and Dual PSG I2S volume boost while preserving sample clipping protection.

## v2.26

- Bumped Explorer to v2.26.
- Added an MSX-MUSIC audio profile to the MSX Explorer ROM detail screen for non-system, non-SCC-class ROMs.
- Ported the LoadROM YM2413/emu2413 audio engine into Explorer and added runtime FM-PAC expanded-slot handling that reads the bundled FM-PAC BIOS from the Explorer UF2 flash payload.
- Moved the Explorer ROM cache from RP2350 SRAM to PSRAM so MSX-MUSIC can coexist with the existing Explorer features without overflowing firmware RAM.
- Updated the Explorer user documentation for MSX-MUSIC profile selection, mapper restrictions, and flash-payload FM-PAC BIOS storage.
- Allowed SPACE as well as ENTER to execute a ROM from the ROM detail screen when `Action: Run` is selected.
- Changed the File Hunter ROM download flow to show `Downloading...` instead of the generic network status check message.
- Removed the WiFi setup hint and F4 special handling from the help page return prompt.
- Increased the Explorer microSD ROM size limit and PSRAM SD ROM buffer from 2 MB to 4 MB.

## v2.25

- Bumped Explorer to v2.25.
- Reworked ROM audio selection around named, mutually exclusive audio profiles so new audio chips can be added without conflicting with SCC or system ROM options.
- Added a Dual PSG audio option for non-system, non-Konami SCC/Manbow2 ROMs, using the same secondary PSG port model as LoadROM.
- Updated the Explorer and PicoVerse 2350 documentation to describe Dual PSG support, audio profile exclusivity, and mapper restrictions.
- Added the shared PicoVerse 2350 Dual PSG implementation reference covering Explorer and LoadROM behavior.
- Updated the public README with user-facing Explorer instructions for selecting the Dual PSG audio profile on supported ROMs.
