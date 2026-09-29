# Host APIs and example-derived design

Use [api.txt](api.txt) for declarations. This guide adds the workflow context and limitations the declaration list does not provide.

## HTTP and parsing

- `HostUrlGetString` takes URL, user agent, header text, POST data, and `NoCookie`, in that order. Omitting POST data is the examples' GET pattern. Supply the correct content type when posting JSON; do not transpose user-agent and header arguments.
- Use `HostOpenHTTP`, `HostGetStatusHTTP`, `HostGetHeaderHTTP`, `HostGetContentHTTP`, and `HostCloseHTTP` when response status/header handling is needed. Balance each acquired handle with a close on every path. The document does not define failure-handle values; verify that detail before assuming zero or a negative sentinel.
- `HttpClient` exposes `Open`, `GetStatus`, `GetContent`, `GetHeader`, and `GetHeaderValue`. Its ownership behavior is not specified in the API text; do not invent a `Close` method for it.
- The `WithAPI` variants are described only as requests with an API key. The document does not specify key provisioning, supported providers, or how to configure it. Do not treat these functions as arbitrary OAuth support.
- Inspect empty/error content and parse success before accessing data. Check `JsonValue.isObject()`, `isArray()`, and array `size()` before indexing, then check field types before conversion. A valid empty JSON array is not a first record.
- XML parsing uses `XMLDocument.Parse`, followed by element traversal and `isValid()` checks. Do not import DOM APIs from JavaScript or generic XML add-ons.
- The string-returning regex overload expects one capture group. The array-output overload exposes match dictionaries with `first` and `second`; consult target examples before inferring field types or position semantics.

For playback endpoints that need headers, the supplied API includes `HostSetUrlUserAgentHTTP`, `HostSetUrlHeaderHTTP`, `HostSetUrlRefererHTTP`, and `HostSetUrlCookieHTTP`. Apply only the necessary values to the returned media URLs, including selectable variants when needed. API-request headers do not automatically establish playback-fetch headers. Do not log tokens, authorization headers, or complete signed media URLs.

## Resolution pipeline

The Twitch parser illustrates three paths: live streams, recorded videos, and clips. It obtains playback authorization, then a media playlist or clip quality list, builds quality dictionaries, and supplies title/author/content metadata. The YouTube parser illustrates URL normalization, IDs, multiple response formats, qualities, subtitles, chapters, and playlist continuation. These are architectural examples, not current service protocol documentation.

For new implementations:

1. Validate supported input and extract a stable identifier.
2. Request the service data required by that path; keep credentials configurable.
3. Validate the response, then build a playable default URL and optional quality choices.
4. Add metadata only when available, with the correct units/types.
5. Return an empty result with a useful diagnostic on unavailable media.

For HLS, parse URI lines without truncating query strings, resolve relative URIs against the playlist location, and keep each variant paired with its own attributes. Do not copy the historical Twitch regex that reconstructs a URL from a restricted character class. Use an explicit supported policy for default quality rather than assuming the first row is best.

Pagination/retry loops should have a bound and stop on empty or repeated continuation tokens. `HostIncTimeOut` extends the host timeout; it is not a substitute for a termination condition. Avoid replaying expired credentials or legacy proxy endpoints from examples.

## Files, configuration, and state

The examples store per-role options and credentials in `config.ini` and read relative extension paths. Those layouts belong to the examples. For new code, use host folder getters to locate the appropriate script/config directory rather than assuming the process working directory. Keep open/read/close separate so file handles are not leaked by nested `HostFileRead(HostFileOpen(...), ...)` expressions.

`HostFileCreate`, `HostFileDelete`, folder create/delete, and `IniFile.Save` are restricted to the config folder by the documentation; absolute paths and `..` are disallowed. Folder deletion requires an empty folder. Do not reuse an absolute script-folder read path as a config write target.

`IniFile` supports `Open`, `OpenString`, section/item enumeration, and typed profile access. The reference leaves several argument names unspecified; do not guess default/found-output ordering beyond the declared signature without checking the host.

`HostSaveString`/`HostLoadString` and integer equivalents are described as **temporary** storage. Do not promise persistence across application restarts. Prefix application-specific keys to avoid unintended collisions. Avoid network requests in global initializers when failure or initialization order can leave unusable state; initialize deliberately through a verified lifecycle hook or on demand.

## Concurrency and diagnostics

Use `HostOpenConsole` and `HostPrintUTF8` for focused diagnostics. Match encoding when using the UTF16 variants. Keep routine output minimal and redact secrets.

Only introduce background threads when needed. `HostCreateThread` expects a callback shaped as `void ThreadFunction(any@ param)`. The API calls for mutex protection of shared resources. `HostWaitThread` and `HostSleep` can hit script timeouts; use bounded short waits and extend timeouts only as needed. Thread-safe script globals do not prove all host APIs are safe on worker threads; that guarantee is absent here.

## Evidence gaps and known traps

| Observation | Development decision |
|---|---|
| API text says `string int HostLoadInteger`, `bool bool Open`, and has `_t` integer spelling inconsistencies | These are documentation defects, not reliable compilable declarations; verify the affected symbol in the target host |
| API text's custom string declarations and example mutation style differ | Confirm required mutation/return semantics; see the bundled AngelScript guide |
| URL-list example calls `HostUrlGetStringGoogle`, absent from the API text | Treat as an undocumented/version-specific helper; do not assume availability or invent its implementation |
| Playback example calls `HostDecodeSigUrl`, absent from the API text | Preserve only in a demonstrated compatible host; the bundled document cannot establish its registration or complete contract |
| Playback header comments disagree with actual `PlayitemParse` | Use the implemented three-parameter, string-returning signature |
| Some examples index an empty response, leak handles, contain fixed credentials, or log full responses | Learn the callback structure without copying those flaws |
| Lifecycle/cancellation hooks occur only in comments in parts of the examples | Do not claim tested behavior or mandatory implementation |

When a required detail is absent, state the exact gap and use target-host diagnostics or task-provided evidence. Do not fill it with a plausible-looking API.
