# Change Log

## v2.25

- Bumped Explorer to v2.25.
- Reworked ROM audio selection around named, mutually exclusive audio profiles so new audio chips can be added without conflicting with SCC or system ROM options.
- Added a Dual PSG audio option for non-system, non-Konami SCC/Manbow2 ROMs, using the same secondary PSG port model as LoadROM.
- Updated the Explorer and PicoVerse 2350 documentation to describe Dual PSG support, audio profile exclusivity, and mapper restrictions.
- Added the shared PicoVerse 2350 Dual PSG implementation reference covering Explorer and LoadROM behavior.
- Updated the public README with user-facing Explorer instructions for selecting the Dual PSG audio profile on supported ROMs.
