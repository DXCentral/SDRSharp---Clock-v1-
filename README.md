# SDR# Clock — preview build

Compact, customizable local and UTC clock for SDR# 1922 Beta. Display live time while receiving, or the recording's date and time when reviewing IQ files with the built-in Baseband File Player.

## Install

1. Close SDR#.
2. Download and extract the **preview package**. Copy its `Plugins/SDRSharp.Clock` folder into your SDR# installation's `Plugins` folder. The resulting layout should be:

   ```text
   <SDR# installation folder>/
     Plugins/
       SDRSharp.Clock/
         SDRSharp.Clock.dll
   ```

3. Start SDR#. Open the main menu, Plugins, `Utilities : Clock`.
4. Drag the Clock window into your preferred SDR# dock. Use Automatic layout to stack the clocks in a narrow dock and put them side by side in a wide dock. Options can be collapsed for a compact display.

When updating, replace the existing `SDRSharp.Clock.dll` while SDR# is closed. Keep only one copy of the plugin DLL under `Plugins`, including its subfolders.

The preview package contains the compiled plugin and instructions. The source package is for developers who want to inspect, modify, or build the plugin. SDR# must be installed separately.

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

- SDR#'s dated-folder format: `YYYY_MM_DD/HH-mm-ss_<frequency>Hz.wav`. SDR#'s bundled recorder creates these using the recording PC's **local** time. Choose that PC's time zone under Recording Zone. The default, PC Time Zone, is suitable when the recording and playback PCs use the same time zone.
- Explicit UTC filenames containing `yyyyMMdd_HHmmssZ`, or `yyyy-MM-ddTHH:mm:ssZ` with colon, hyphen, or underscore time separators. These are UTC regardless of Recording Zone. ISO fractional seconds are recognized.

Local conversion uses the time zone's rule for the **recording date**, including historical daylight saving time, not the PC's current offset. Playback elapsed time is added in UTC before deriving the local display, so crossings of midnight and DST changes follow the recording timeline.

SDR#'s dated-folder format contains no time-zone identifier. When exchanging recordings, select the time zone used on the recording PC, or UTC if that PC was configured for UTC. The local clock continues to use the playback PC's time zone. A recording's time zone does not identify the transmitter's location.

The repeated local hour when DST ends has two possible UTC instants. The default Unspecified setting displays a request to choose the first or second occurrence under Repeated DST Hour. A nonexistent spring-forward local time is flagged as invalid. Files with no supported dated timestamp show an unavailable message and blank clock values instead of live time. Filesystem creation/modification dates are never substituted for recording dates.

## Preview status and compatibility

- Tested in SDR# 1922 Beta. Other SDR# versions have not been verified.
- Verified live display, recording changes, play/pause and seek alignment, daylight/standard-time conversion, and local/UTC calendar-day rollover.
- Release build completed with zero warnings and zero errors.
- Repeated and nonexistent DST hours were checked against Windows time-zone rules. The complete repeated-hour selection workflow has not yet been verified with a recording during a DST transition.

Playback integration targets SDR# 1922's built-in Baseband File Player. Recovering the recording's dated folder currently depends on a version-specific, read-only adapter to the player's full file path. Changes to SDR# internals may require an adapter update. Recording files are not modified.

## Source and build

Requirements: the **.NET 10 SDK** and the **Airspy SDR# VDK**.

1. Extract the source package's `Clock` directory into the VDK folder, beside `lib`, `docs`, and `template`.
2. Follow the VDK's `AGENTS.md` and `CLAUDE.md`, including setup of the project's separate `sdrsharp` test installation.
3. Close that test installation and run the following from the `Clock` directory:

   ```text
   dotnet build -c Release
   ```

The VDK deploy target copies the build into `Clock/sdrsharp/Plugins/SDRSharp.Clock`.

The plugin runs independently of DSP streaming. `ClockProcessor` holds display settings and formats time, `ClockTimeSource` supplies live or recording time, and `ClockFace` renders the display. Standard configuration controls follow the SDR# theme; clock-face palettes are rendered separately.

Playback support is isolated behind `IClockTimeSource`. The adapter reads the built-in player's file name, sample rate, position, loop start, seek position, and playback state. The full-path adapter reads the existing `_fileName` string field only for the expected player type, checks its type, and verifies that the filenames agree. This dependency is limited to the adapter so it can be updated if SDR# provides a public recording-time API.

## License

Built from the Airspy VDK template. See `LICENSE.txt` for the applicable license and Airspy ecosystem usage terms.
