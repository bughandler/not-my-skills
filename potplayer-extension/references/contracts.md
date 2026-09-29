# Media callback and dictionary contracts

These observations are distilled from four real implementations: YouTube and Twitch URL-list scripts, and YouTube and Twitch playback parsers. Implemented functions are stronger evidence than their introductory comments. This is an observed interface, not an exhaustive versioned host specification.

## Shared callbacks

Both roles implement `string GetTitle()`, `string GetVersion()`, and `string GetDesc()`. Titles can be plain text. Examples also use localized tokens such as `{$CP0=English label$}` combined with other code-page variants; preserve existing tokens when editing them.

Optional account callbacks seen in implementations:

```angelscript
string GetLoginTitle()
string GetLoginDesc()
string ServerCheck(string User, string Pass)
string ServerLogin(string User, string Pass)
string GetWebAccountUrl()
string GetWebAccountDomain()
```

The Twitch URL-list example uses login callbacks to save a username and return display text. This does not prove a universal success/error return convention or mandate password authentication. The YouTube implementation uses the web-account callbacks and semicolon-separated cookie domains. Supply only account behavior the service actually requires.

`void OnInitialize()`, `void OnFinalize()`, `void ServerLogout()`, `string GetUserText()`, and `string GetPasswordText()` appear in callback comment lists. The API text additionally demonstrates `OnInitialize` for starting a thread. These observations do not establish ordering, concurrency, or universal availability of all hooks.

## URL browsing/search

```angelscript
array<dictionary> GetCategorys()
string GetSorts(string Category, string Extra, string PathToken, string Query)
array<dictionary> GetUrlList(string Category, string Extra,
                             string PathToken, string Query, string PageToken)
```

The Twitch script names the second `GetUrlList` parameter `Genre`; the YouTube script calls it `Extra` and parses comma-separated `key=value` options such as `genre` and `sort`. Parameter names do not change the signature; parameter order and type do. Avoid treating the second argument as a raw genre for every extension.

Category dictionaries:

| Key | Observed value/use |
|---|---|
| `title` | Display string, optionally localized |
| `Category` | Category identifier passed back to callbacks |
| `type` | String `search` in the YouTube category |
| `Genres` | Comma-separated `value=label` choices in the YouTube category |

`GetSorts` returns comma-separated `value=label` choices, or an empty string when none apply.

Result dictionaries:

| Key | Observed value/use |
|---|---|
| `url`, `title` | Item page URL and display title |
| `author`, `desc`, `date`, `thumbnail` | Optional strings |
| `folder` | String `1` for a folder, `parent` for an up-navigation row |
| `PathToken` | Opaque identifier for folder contents |
| `PageToken` | Continuation token placed on a returned row |

The YouTube example attaches `PageToken` to one video row. The callback receives tokens by value; assigning its local `PageToken` cannot deliver continuation state to PotPlayer. Attach the continuation token to an actual emitted result, after filtering invalid rows, and verify host pagination. Do not invent a separate return envelope. Returning an empty array means no rows; an empty page with only a continuation token has no demonstrated representation here.

Preserve opaque token case and contents; encode them as query components when sending them to a service. For folder navigation, emit an up row with `title = ".."` and `folder = "parent"` when appropriate. Do not conflate `PathToken` with `PageToken`.

## Playback resolution

Actual implementations use:

```angelscript
bool PlayitemCheck(const string &in path)
string PlayitemParse(const string &in path, dictionary &MetaData,
                     array<dictionary> &QualityList)
```

`PlayitemCheck` classifies supported input. Keep it cheap, and validate the hostname/path rather than matching a provider name anywhere in arbitrary text. `PlayitemParse` returns a playable media URL, with an empty string on failure. Its output containers can be absent; guard each independently. The stale example comment showing `array<dictionary> PlayitemParse(const string &in)` is not the implemented signature.

Observed metadata:

| Keys | Type/meaning |
|---|---|
| `title`, `author`, `content`, `date`, `thumbnail` | Strings; playback description uses `content`, whereas URL browsing uses `desc` |
| `webUrl`, `vid`, `fileExt`, `chatUrl` | Strings used by the YouTube resolver; service-specific applicability |
| `duration` | Numeric milliseconds in the YouTube playback metadata |
| `viewCount`, `likeCount`, `dislikeCount` | Strings in the YouTube example; the Twitch script also uses display text for view count |
| `errorMessage` | Failure explanation string; still return an empty URL when resolution fails |
| `type3D`, `is360` | Projection metadata; examples vary in integer/boolean representation, so verify before adding |
| `subtitle`, `chapter` | Arrays of dictionaries |

Subtitle rows use strings `name`, `url`, `langCode`, optionally `kind`, `langTranslated`, `langOriginal`. Chapter rows use `title` and `time`; the observed `time` is a string of milliseconds. These are metadata outputs, not evidence of a standalone subtitle-provider extension interface.

Quality-list rows:

| Keys | Observed type |
|---|---|
| `url`, `quality`, `qualityDetail`, `resolution`, `bitrate`, `format` | Strings |
| `itag` | Integer identifier; provider-specific, not a universal ranking |
| `fps` | Double |
| `type3D` | Integer |
| `is360`, `isHDR` | Boolean in quality rows |
| `audioName`, `audioCode` | Strings in the YouTube implementation |
| `audioIsDefault` | Boolean in the YouTube implementation |
| `bitrateVal` | Integer used by that script's quality deduplication; not established as required host metadata |

Emit the fields needed for the feature instead of filling every key with invented values. Keep audio-only and video variants distinct. Selecting the highest apparent resolution does not establish codec support or audio availability; preserve target-host behavior for adaptive streams and audio pairing. Do not invent URL-combining delimiters.

## Playlist expansion

```angelscript
bool PlaylistCheck(const string &in path)
array<dictionary> PlaylistParse(const string &in path)
```

These are implemented in the YouTube parser. Return ordered rows using `url`, `title`, optional `thumbnail`, and `current = "1"` for the selected entry. URLs can be page URLs subsequently resolved by `PlayitemParse`. Preserve order when deduplicating.

Playlist `duration` is inconsistent across example branches (including strings of seconds); do not apply playback metadata's milliseconds rule blindly to playlist rows. Confirm the target consumer's convention before changing it.

`void PlayitemCancel()`, `void PlaylistCancel()`, and `string GetStatus()` appear only in the inspected playback comment list, without implementations. Treat these as candidate hooks to verify, not required callbacks or a proven cancellation mechanism.

## Embedded broadcast browser

The YouTube playback script implements `string GetBroadcastListUrl()` returning semicolon-separated entry URLs and `string GetBroadcastListScript()` returning JavaScript. Its comment associates the latter with WebView2 document-created injection.

The observed JavaScript checks for `chrome.webview`, prevents duplicate installation, filters clicks so controls such as seeking remain usable, and sends an object with `type: 'potPlayer.video-click'` and a `url` field through `chrome.webview.postMessage`. Treat this as a build-specific observed bridge, not a general API guarantee. Preserve it when modifying that integration and verify the receiving host before introducing it elsewhere. The JavaScript runs in the browser; its DOM APIs are not AngelScript functions.
