# Copilot instructions for SerialCommunication

Build, test, and lint commands
- Windows / Visual Studio (solution):
  - Build (Debug):
    msbuild SerialCommunication.slnx /t:Build /p:Configuration=Debug
  - Build (Release):
    msbuild SerialCommunication.slnx /t:Build /p:Configuration=Release
  - Clean:
    msbuild SerialCommunication.slnx /t:Clean
  - Open in Visual Studio: open SerialCommunication.slnx in VS
  - Run built app: SerialCommunication\bin\Debug\SerialCommunication.exe

- Arduino (sketch in repo root):
  - Compile with arduino-cli (example for Uno):
    arduino-cli compile --fqbn arduino:avr:uno "C:\Users\<you>\source\repos\SerialCommunication\SerialCommunication.ino"
  - Upload to device (replace COM3 and fqbn as needed):
    arduino-cli upload -p COM3 --fqbn arduino:avr:uno "C:\Users\<you>\source\repos\SerialCommunication\SerialCommunication.ino"

Notes:
- There are no automated tests or linting configs in the repository; use the above build commands to validate changes.

High-level architecture
- Two-part project:
  1) PC-side WinForms application (SerialCommunication/)
     - C# .NET Framework WinForms app that provides a UI and communicates with a microcontroller over a serial port.
     - Entry point: Program.cs. Main UI: Form1.cs (reads/writes via System.IO.Ports.SerialPort).
     - On connect the app sends a "ping" command and expects a "pong" response to verify the link.
     - UI exposes serial-port parameters (baudrate, parity, stopbits, handshake, RTS/DTR, datapins) and displays status.
     - Built output: SerialCommunication\bin\Debug\SerialCommunication.exe
  2) Device-side sketch and helper C/C++ (root):
     - SerialCommunication.ino implements a simple ASCII text command protocol using the bundled SerialCommand library (SerialCommand.h / SerialCommand.cpp).
     - Recognized commands: set, toggle, get, ping, help, debug. Responses are plain-text (println) and some use a prefixed format (e.g., "d<pin>: <value>", "a<pin>: <value>").
     - Pins: digital outputs 2..4, PWM 9..11, inputs 5..7, analog A0..A5.
     - SerialCommand tokenizes by space and uses a line terminator; handlers are registered via sCmd.addCommand(...).

Key conventions and repo-specific patterns
- Serial protocol conventions
  - Commands are ASCII tokens separated by spaces, terminated by newline(s). Use WriteLine/ReadLine from host to match the sketch's println/line-based parsing.
  - Pin identifiers use short prefixes: dN (digital), pwmN (PWM), aN (analog). Examples: "set d2 on", "get a3".
  - "ping" <-> "pong" handshake used by the WinForms app to confirm connectivity.
  - Error and status responses are sent as human-readable strings (often in Dutch); parsing the full protocol is done by string matching, not structured JSON.

- SerialCommand usage
  - The repo includes a copy of the SerialCommand Arduino helper (SerialCommand.h/.cpp). This library expects handlers of signature void f() and a default handler void f(char*).
  - To extend device commands, add sCmd.addCommand("cmd", handler) in setup() and implement the handler using sCmd.next() to retrieve arguments.

- UI expectations
  - The WinForms UI enumerates available COM ports and sets baudrate default to 115200 to match the sketch.
  - After opening the serial port the UI sends "ping" and reads a single line; it expects exactly "pong" (trimmed) before marking the device as connected.
  - The UI sets serial line settings based on control values (databits, parity, stopbits, handshake, RTS/DTR). These map directly to System.IO.Ports settings.

Files and places to look when changing behavior
- Host-side: SerialCommunication\Form1.cs (main logic), Program.cs (entry)
- Device-side: SerialCommunication.ino, SerialCommand.h/.cpp, analog.c
- Resources (UI images) in SerialCommunication\Resources\

AI assistant config files
- No existing Copilot/Claude/Cursor/other assistant config files found in repo root. If such files are added, include key rules here.

When editing the serial protocol
- Keep host messages line-oriented and send simple tokens separated by spaces.
- Update both device handlers and the WinForms connect/parse logic together to avoid breakage.

If you add tests or CI
- Document the test commands and add lint configs to the repository (this file should be updated to include them).

---
Generated: repository scan (Form1.cs, SerialCommand.*, SerialCommunication.ino)

If you'd like, configure MCP servers for this project (e.g., a Playwright server for web projects). Would you like me to configure any MCP servers now?  

Summary: created .github/copilot-instructions.md with build commands, architecture, and key conventions. Want any adjustments or additional coverage (e.g., deeper protocol examples or contributor workflow)?
