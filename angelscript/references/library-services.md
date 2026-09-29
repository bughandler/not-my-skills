# Optional SDK services

Adapted from the official 2.39.0 topics: Exception handling; Co-routines; System functions; datetime; file; filesystem; socket; Math functions. All APIs below depend on host registration; they are not guaranteed language built-ins.

## Exceptions and coroutines

`void throw(const string &in exception)` raises an exception explicitly. `string getExceptionInfo()` retrieves the last exception string. The language catch syntax itself is:

```angelscript
// Requires registered string, throw, and getExceptionInfo functions.
void Example()
{
    try { throw("example failure"); }
    catch { string reason = getExceptionInfo(); }
}
```

The coroutine facility declares `funcdef void coroutine(dictionary)` and `void createCoRoutine(coroutine@, dictionary)`. New coroutines start yielded; `void yield()` hands control to the next queued coroutine. Scheduling is round-robin, and execution resumes after the previous yield. Do not replace this with unsupported `async`/`await` syntax or assume concurrent threads.

## System functions

The documented interface includes `void print(const string &in line)` (no automatic newline), `string getInput()`, and `array<string>@ getCommandLineArgs()`. `int exec(const string &in cmd)` and its overload with `string &out output` run a system command, optionally capturing stdout; success returns the process exit code, while error returns -1 or raises an exception. Their presence depends on the application's exposed interface.

## Date and time

`datetime()` starts at the current UTC time, with seconds precision. `datetime(y, m, d, h = 0, mi = 0, s = 0)` builds a date/time; copy construction is supported. Read-only properties include `year`, `month` (1–12), `day`, `hour` (0–23), `minute`, `second`, and `weekDay` (0 is Sunday).

`bool setDate(uint year, uint month, uint day)` and `bool setTime(uint hour, uint minute, uint second)` leave the object unchanged on invalid input. Subtracting two datetimes gives a difference in seconds. Adding/subtracting seconds produces a new datetime. The type supports comparisons and assignment; timezone adjustments are the caller's responsibility.

## File and filesystem

For `file`, check `open(filename, mode) >= 0`, with mode `"r"`, `"w"`, or `"a"`. `close()` is explicit. `getSize()` returns a negative value if no file is open. `readString(uint length)` reads bytes; `readLine()` retains the newline. `isEndOfFile()` tests the current position.

```angelscript
// Requires registered file and string types.
string ReadText(const string &in path)
{
    file input;
    if (input.open(path, "r") < 0)
        return "";
    int size = input.getSize();
    string result;
    if (size >= 0)
        result = input.readString(uint(size));
    input.close();
    return result;
}
```

Binary methods include `readInt(bytes)`, `readUInt(bytes)`, `readFloat()`, `readDouble()` and corresponding write methods. `writeString`, integer writes, and floating-point writes return bytes written or a negative error. Numeric byte order is controlled by `mostSignificantByteFirst`, false by default. Position methods are `getPos()`, `setPos(int)`, and `movePos(int)`; setters return the previous position or a negative error.

The `filesystem` object maintains its own current path: `changeCurrentPath(path)` does not change the application's working directory. `getCurrentPath()`, `getDirs()`, `getFiles()`, `isDir(path)`, `isLink(path)`, and `getSize(path)` support inspection. Size returns -1 on failure.

`makeDir`, `removeDir`, `deleteFile`, `copyFile`, and `move` return zero on success. `removeDir` only removes an empty directory. `getCreateDateTime` and `getModifyDateTime` return UTC values and raise an exception when the file is missing or inaccessible.

## TCP sockets

The SDK `socket` provides TCP connections using queues/buffers. It is not a documented UDP API.

| Method | Contract |
|---|---|
| `int listen(uint16 port)` | Start listening; negative on failure |
| `socket@ accept(int64 timeout = 0)` | New connection or null; timeout in microseconds |
| `int connect(uint ipv4address, uint16 port)` | Connect; negative on failure |
| `int send(const string &in data)` | Bytes sent or negative error |
| `string receive(int64 timeout = 0)` | Received bytes; timeout in microseconds |
| `bool isActive() const` | Listening or connected |
| `int close()` | Close; negative on failure |

Zero timeout returns immediately when nothing is available. The address is a packed IPv4 integer; loopback is `0x7F000001`. Check return values and null handles. The reference does not specify an application-message framing protocol; do not invent one as socket-library behavior.

## Math

The math add-on registers trigonometric functions (`cos`, `sin`, `tan`, inverse and hyperbolic variants), `atan2(y, x)`, natural `log`, `log10`, `pow`, `sqrt`, `abs`, `ceil`, `floor`, and `fraction`. Angles are radians. Functions normally use `float`; the add-on can be built to register `double` instead via `AS_USE_FLOAT=0`.

`closeTo` provides approximate comparison: the documented default epsilon is `0.00001f` for float and `0.0000000001` for double. `fpFromIEEE` / `fpToIEEE` convert between floating-point values and their integer bit representations.

The separately registered `complex` type has float `r` and `i` components, default/copy/one-float/two-float constructors, arithmetic and compound assignment, equality, `abs()` for magnitude, and `ri`/`ir` swizzles. Do not assume it is registered just because basic math functions exist.
