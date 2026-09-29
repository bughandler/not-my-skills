---
name: potplayer-extension
description: Create, review, and debug PotPlayer AngelScript media extensions for URL browsing, playback URL resolution, playlists, metadata, and quality selection. Use for PotPlayer extension callbacks and Host APIs, not browser extensions or general player automation.
---

# PotPlayer extension development

Build AngelScript scripts hosted by PotPlayer. This skill covers PotPlayer's extension contracts and host APIs. For language syntax and semantics, use a standalone AngelScript skill when the user provides one, such as `angelscript`; no specific installation path is required. PotPlayer references are bundled here. Service-specific protocols not covered here require evidence supplied with the task rather than guessed endpoints.

## Workflow

1. Choose the extension role: `Media/UrlList` supplies browse/search results; `Media/PlayParse` resolves a page into playable media or expands a playlist. A service can need both. Preserve an existing extension's callback names, signatures, dictionary keys, and output types.
2. Read [callback contracts](references/contracts.md) for the chosen role. These contracts come from inspected YouTube and Twitch implementations; comment-only hooks and version-dependent behavior are explicitly marked.
3. Look up host declarations in the bundled [API document](references/api.txt). Use [host patterns and limitations](references/host-patterns.md) for HTTP, parsing, configuration, resource cleanup, and known documentation gaps. Prefer documented APIs over undocumented service-specific helpers.
4. Implement the smallest useful callback path. Start with cheap URL classification, then fetch/parse, then map validated results into the host's dictionary contract. Keep service parsing separate enough to exercise with response fixtures. For examples of output construction, use [implementation patterns](references/patterns.md).
5. Validate callback recognition, failure behavior, and output consumption in the target PotPlayer build when available. Record the build and distinguish reference review, script compilation/loading, and actual playback. Without a runnable host, report reference-checked code and the remaining runtime checks; a generic AngelScript engine cannot prove PotPlayer compatibility.

## Host invariants

- There is no automatic `main`. PotPlayer calls specifically named functions. `GetCategorys` and `PlayitemParse` must retain their exact spelling and case.
- The implemented playback signature returns a **string** and writes metadata and qualities through reference parameters. Do not copy the stale one-argument, array-returning declaration found in example comments.
- Treat `MetaData` and `QualityList` as potentially absent: the examples use `@MetaData !is null` and `@QualityList !is null`. Guard each output independently, including inside helpers.
- Return an empty string for unsuccessful playback resolution and an empty array for no list results. Do not return a fabricated stream URL or an error message as a playable URL.
- PotPlayer's embedded engine version and registered libraries are not fully established by these examples. A standalone language reference does not establish that a feature or API is available in the target PotPlayer build.
- Keep request authentication separate from media playback headers. A successful API request does not prove the player can fetch the returned stream.

## Packaging and verification

The observed layout uses `Media/PlayParse/MediaPlayParse - Service.as` and `Media/UrlList/MediaUrlList - Service.as`, under the extension root, with optional same-basename `.ico` files. The examples also use role-local `config.ini` files; those files configure those scripts and are not a demonstrated universal extension manifest. Do not assume both roles or an icon are mandatory.

Verify only the affected behaviors: discovery/loading, supported and unrelated URLs, empty/malformed responses, authentication failure, quality changes, playlist ordering, browsing pagination, Unicode metadata, and repeated invocation/resource cleanup. For a resolver, verify both the returned default media URL and any selectable alternatives. Installation/reload UI varies by build and is not specified by this reference.

The bundled API text preserves the supplied declarations and known typos; only its opening external documentation URL was removed to keep this package free of external links. The other references are synthesized development guidance, not a claim that the historical providers still operate.
