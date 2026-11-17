# SDR# Clock — preview build

Compact dockable local/UTC clock for SDR# 1922 Beta and the Airspy ecosystem. Built from the Airspy VDK template; the included VDK license applies.

## Install

1. Close SDR#.
2. Extract the binary package. Copy its `Plugins/SDRSharp.Clock` folder into the `Plugins` folder of your SDR# installation. For this machine the result is `C:\SDRSharp New 64bit\Plugins\SDRSharp.Clock\SDRSharp.Clock.dll`.
3. Start SDR#. Open the main menu, Plugins, `Utilities : Clock`.
4. Drag the Clock window into your preferred SDR# dock. Use Automatic layout to stack the clocks in a narrow dock and put them side by side in a wide dock. Options can be collapsed for a compact display.

The package does not include or replace SDR# itself. Do not install a second copy of this plugin DLL elsewhere under Plugins; SDR# scans subfolders.

## Display choices

- Local, UTC, or both. Local time can use 12- or 24-hour notation; UTC uses 24-hour notation.
- Automatic, Side by Side, or Stacked layout.
- Optional date and weekday. Date formats: system short date, DD/MM/YYYY, MM/DD/YYYY, YYYY-MM-DD, and DD MMM YYYY.
- Vintage LCD (default), Seven-Segment LED, or System Font. System Font enables the installed-font selector; the segment shapes are drawn without a separate font installation.
- Clock size 12–72 points, Classic LCD / Amber / Red / Ice Blue / SDR# Theme / Custom colors, and digit/background color pickers.
- Settings persist when SDR# closes normally.

## Live and recording time

Live sources show the computer clock. The local display uses the PC's configured Windows time zone.

The built-in Baseband File Player switches the clock to recording time, including while paused. The calculation uses the recording start instant plus the file player's sample position divided by its sample rate. It accounts for the selected loop start and pending seek position; it does not advance recording time using a wall-clock stopwatch. The file player's Real Time checkbox does not control this plugin.

Recognized recording timestamps:

- SDR# folders and filenames such as `2025_04_14/05-40-00_162473418Hz.wav`. SDR#'s bundled recorder creates these using PC **local** time. Choose the recording PC's time zone under Recording Zone. The default is PC Time Zone, suitable when playback and recording PCs use the same zone. For the supplied Central-time example, the UTC start is 2025-04-14 10:40:00, not 05:40:00.
- Explicit UTC filenames containing `yyyyMMdd_HHmmssZ`, or `yyyy-MM-ddTHH:mm:ssZ` with colon, hyphen, or underscore time separators. These are UTC regardless of Recording Zone. ISO fractional seconds are recognized.

Local conversion uses the time zone's rule for the **recording date**, including historical daylight saving time, not the PC's current offset. Playback elapsed time is added in UTC before deriving the local display, so crossings of midnight and DST changes follow the recording timeline.

Recording folders/filenames contain no time-zone identifier. When exchanging recordings, select the zone used on the recording PC, or UTC if that PC was configured for UTC. Do not infer a transmitter's location from the recording zone.

The repeated local hour when DST ends has two possible UTC instants. The default Unspecified setting displays a request to choose the first or second occurrence under Repeated DST Hour. A nonexistent spring-forward local time is flagged as invalid. Files with no supported dated timestamp show an unavailable message and blank clock values instead of live time. Filesystem creation/modification dates are never substituted for recording dates.

## Validation

- Release build completed successfully: exit code 0, zero warnings, zero errors.
- Deployed and loaded in isolated SDR# 1922 Beta test installations. Live LCD faces and different local/UTC calendar dates were observed; the user confirmed appearance, file switching, play, pause, and seek alignment.
- The historical-DST update was loaded in SDR#. The supplied 2025-04-14 recording displayed Local 05:40:00 / UTC 10:40:00 while paused at its start, with both dates correct. The PC currently uses Central Standard Time; the recording date correctly resolves to the daylight offset.
- The user subsequently tested a recording outside DST and confirmed the standard-time conversion. The test window was also observed showing Local 21:40:09 on 2024-12-30 and UTC 03:40:09 on 2024-12-31: a six-hour offset and correct calendar-day rollover.
- Historical January/April offsets and repeated/nonexistent DST hours were checked against Windows TimeZoneInfo. The complete repeated-hour UI workflow has not yet been exercised with a recording during a DST transition.
- The user's main SDR# installation has not been modified by this package.

## Source and build

Extract the source package's `Clock` directory beside `lib`, `docs`, and `template` in the supplied VDK. Follow the VDK's AGENTS.md and CLAUDE.md. Create the project's separate `sdrsharp` test installation as described there, close that test application, and run `dotnet build -c Release` from Clock. .NET 10 SDK is required. The unchanged VDK deploy target copies the build into `Clock/sdrsharp/Plugins/SDRSharp.Clock`.

There are no DSP hooks: ClockProcessor holds display state and formats time, ClockTimeSource reads the host time source, and ClockFace renders the display. The clock runs while radio streaming is stopped. Standard configuration controls follow the SDR# theme; the requested clock-face palettes are rendered separately. A dedicated Utilities category avoids newer SharpConstants dependencies.

The kit documents SourceType but omits its public Source property; the reference assembly and both inspected 1922 host builds expose it. Playback support is isolated behind IClockTimeSource. The built-in file player's public FileName, Samplerate, CurrentPosition, SeekPosition, LoopStartPosition, and IsPlaying members were inspected before use. CurrentPosition is relative to LoopStartPosition, in complex samples.

The player exposes only the safe filename publicly. The version-specific adapter reads its existing `_fileName` string field, read-only and only for the exact built-in file-player type, to recover the directory date. It checks the field type and verifies that the safe filenames agree. This is a compatibility dependency on SDR# 1922's implementation; a future host API for recording path/start instant/position would allow replacing only this adapter. No SDR# files or recording contents are changed.
