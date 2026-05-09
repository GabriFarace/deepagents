# `libs/cli/deepagents_cli/unicode_security.py`

> Unicode security helpers for deceptive text and URL checks.

## Position in the system

This helper is imported by the CLI core rather than being an entry point itself. It keeps one peripheral concern isolated so `main.py`, `app.py`, and the server/client lifecycle files can stay focused on orchestration.

## Functions and classes

### `UnicodeIssue`

A dangerous Unicode character found in text.

Additional notes from the source docstring:

```text
Attributes:
    position: Zero-based index in the original string.
    character: The single raw character found in the input.
    codepoint: Uppercase code point string like ``U+202E``.
    name: Unicode character name.
```

Methods worth reading inside this class:

- `__post_init__(self)`: This helper performs one narrow step for the surrounding module while keeping normalization, error handling, or side-effect policy in one place.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `UrlSafetyResult`

Safety analysis output for a URL string.

Additional notes from the source docstring:

```text
A result may have `safe=True` with non-empty `warnings` when
informational warnings (e.g. punycode decoding) are present without
suspicious patterns.

Attributes:
    safe: `True` if no suspicious patterns were found.
    decoded_domain: Punycode-decoded hostname when it differs from the
        original hostname.

        `None` when unchanged or no hostname exists.
    warnings: Human-readable warning strings (immutable).
    issues: Dangerous Unicode issues found in the full URL (immutable).
```

The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `detect_dangerous_unicode(text: str)`

Detect deceptive or hidden Unicode code points in text.

Additional notes from the source docstring:

```text
Args:
    text: Input text to inspect.

Returns:
    A list of `UnicodeIssue` entries in source order.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `strip_dangerous_unicode(text: str)`

Remove known dangerous/invisible Unicode characters from text.

Additional notes from the source docstring:

```text
Args:
    text: Input text to sanitize.

Returns:
    Sanitized text with dangerous characters removed.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `render_with_unicode_markers(text: str)`

Render hidden Unicode characters as explicit markers.

Additional notes from the source docstring:

```text
Example output: `abc<U+202E RIGHT-TO-LEFT OVERRIDE>def`.

Args:
    text: Input text to render.

Returns:
    Text where dangerous characters are replaced with visible markers.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `summarize_issues(issues: list[UnicodeIssue], *, max_items: int=3)`

Summarize Unicode issues for warning messages.

Additional notes from the source docstring:

```text
Deduplicates by code point. When more than *max_items* unique entries exist,
the summary is truncated with a `+N more entries` suffix.

Args:
    issues: A list of detected issues.
    max_items: Max unique code points to include in output.

Returns:
    Comma-separated summary, e.g.
        `U+202E RIGHT-TO-LEFT OVERRIDE, U+200B ZERO WIDTH SPACE`.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `format_warning_detail(warnings: tuple[str, ...], *, max_shown: int=2)`

Join safety warnings into a display string with overflow indicator.

Additional notes from the source docstring:

```text
Args:
    warnings: Warning strings from a `UrlSafetyResult`.
    max_shown: Maximum warnings to include before truncating.

Returns:
    Semicolon-separated detail string, e.g. `'warn1; warn2; +1 more'`.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `check_url_safety(url: str)`

Check a URL for suspicious Unicode and domain spoofing patterns.

Additional notes from the source docstring:

```text
Args:
    url: URL string to inspect.

Returns:
    `UrlSafetyResult` including decoded domain and warning details.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_decode_hostname(hostname: str)`

Decode `xn--` punycode labels into Unicode labels when possible.

Additional notes from the source docstring:

```text
Returns:
    Tuple of (decoded hostname, list of labels that failed to decode).
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_split_hostname_labels(hostname: str)`

Split a hostname into non-empty labels.

Additional notes from the source docstring:

```text
Returns:
    Hostname labels without empty entries.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_is_local_or_ip_hostname(hostname: str)`

Return whether hostname is localhost or an IP address literal.

Additional notes from the source docstring:

```text
Returns:
    `True` when hostname is localhost or an IP literal, else `False`.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_scripts_in_label(label: str)`

Collect non-common scripts used by a domain label.

Additional notes from the source docstring:

```text
Returns:
    Set of script names used by the label, excluding common/inherited.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_label_has_suspicious_confusable_mix(label: str)`

Return whether a label has likely deceptive confusable characters.

Additional notes from the source docstring:

```text
Only flags labels that mix multiple scripts while containing confusable
characters. Single-script labels (even with confusables) are not flagged
because they represent legitimate use of that script.

Returns:
    `True` when the label mixes scripts and contains confusable characters.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_char_script(character: str)`

Classify a character into a coarse Unicode script bucket.

Additional notes from the source docstring:

```text
Returns:
    One of: `'Fullwidth'`, `'Latin'`, `'Cyrillic'`, `'Greek'`, `'Armenian'`,
        `'EastAsian'`, `'Inherited'`, `'Common'`, or `'Other'`.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_format_codepoint(character: str)`

Format character code point in `U+XXXX` uppercase form.

Additional notes from the source docstring:

```text
Returns:
    Uppercase `U+XXXX` codepoint string.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_unicode_name(character: str)`

Return a stable Unicode name with a fallback for unknown code points.

Additional notes from the source docstring:

```text
Returns:
    Unicode name string for the character.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `iter_string_values(data: dict[str, Any], *, prefix: str='')`

Flatten nested dict/list structures into key-path/string pairs.

Additional notes from the source docstring:

```text
Returns:
    List of ``(path, value)`` tuples for all string leaves.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_iter_string_values_from_list(values: list[Any], *, prefix: str)`

Flatten nested list values into key-path/string pairs.

Additional notes from the source docstring:

```text
Returns:
    List of `(path, value)` tuples for all string leaves.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `looks_like_url_key(arg_path: str)`

Return whether a key path suggests URL-like content.

Additional notes from the source docstring:

```text
Returns:
    `True` for URL-like key names, otherwise `False`.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

## Gotchas

Most helpers in this area are called from core CLI files that are documented separately. Check both sides of the call before changing return shapes or exception behavior.
