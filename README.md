Ever crawl a site and get a bunch of garbage back? Content that relies on JavaScript, pages of navigational headers, sidebars and footers... and your content just... not... there...

----

## Crawl a site. Get the stuff you want. Every time.

**wgetjs**, part of the [DAY50](https://day50.dev) suite of open-source tools for AI workflows, is for javascript sites. it's backwards compatible with wget

Unlike traditional wget, wgetjs uses a headless Chrome browser to render JavaScript, ensuring you capture the fully-rendered HTML that SPAs and dynamic sites generate.

Does it work for everything? 

You can do all kinds of stupid things with JS. This only covers most of them.

## Usage

```bash
# Basic recursive download
wgetjs -rc -np http://somesite.com/

# With parallel workers
wgetjs -rc -np -P 4 http://somesite.com/

# Limit recursion depth
wgetjs -rc -l 3 http://somesite.com/

# Only download HTML and PDF files
wgetjs -rc -A html,pdf http://somesite.com/

# Exclude images and certain directories
wgetjs -rc -R jpg,png,gif -X /admin,/private http://somesite.com/

# Only follow links within specific domains
wgetjs -rc -D example.com,cdn.example.com http://example.com/

# Accept only URLs matching a pattern
wgetjs -rc --accept-regex='/docs/.*' http://somesite.com/

# Convert downloaded HTML to markdown
wgetjs -rc --post='text/html:markitdown {} > {}.md' http://example.com/

# Convert images to webp and HTML to markdown
wgetjs -rc --post='image/*:convert {} {}.webp\ntext/html:markitdown {} > {}.md' http://example.com/

# Read post-processing rules from a file
wgetjs -rc --post=@post-rules.txt http://example.com/
```

## Options

### Download Control

| Option | Description |
|--------|-------------|
| `-rc` | Recursive download (follow links) |
| `-np` | No parent — don't ascend to parent directories |
| `-P N` | Number of parallel workers (default: 1) |
| `-l N`, `--level=N` | Max recursion depth (default: 5, use `0` or `inf` for infinite) |
| `-w SECONDS` | Wait SECONDS between retrievals |
| `--waitretry=SECONDS` | Wait 1..SECONDS between retries (backoff) |
| `--random-wait` | Wait 0.5×WAIT to 1.5×WAIT secs (randomized) |

### URL Filtering

| Option | Description |
|--------|-------------|
| `-A LIST`, `--accept=LIST` | Accept extensions/patterns (e.g., `html,pdf`, `*.mp3`) |
| `-R LIST`, `--reject=LIST` | Reject extensions/patterns (e.g., `jpg,png`, `*.[zb]p2`) |
| `--accept-regex=REGEX` | Accept URLs matching regex |
| `--reject-regex=REGEX` | Reject URLs matching regex |
| `--regex-type=TYPE` | Regex type: `posix` (default) or `pcre` |
| `-D LIST`, `--domains=LIST` | Accepted domains (requires `-H`) |
| `--exclude-domains=LIST` | Rejected domains |
| `-H`, `--span-hosts` | Follow links to other hosts |
| `-L`, `--relative` | Follow relative links only |
| `-I LIST`, `--include-dirs=LIST` | Include directories (wildcards supported) |
| `-X LIST`, `--exclude-dirs=LIST` | Exclude directories (wildcards supported) |
| `--ignore-case` | Ignore case when matching patterns |
| `--post=RULES` | Run post-processing commands scoped by MIME type (see below) |

### Other

| Option | Description |
|--------|-------------|
| `-E`, `--adjust-extension` | Append `.html` to filenames that don't already have an HTML extension |
| `--method=CMD` | Custom command to fetch HTML (alternative to Chrome) |
| `-h, --help` | Show help |

## Post-Processing (`--post`)

Run commands on downloaded files, scoped by MIME type. Format is `mimetype:command` where `{}` is replaced with the file path.

```bash
# Single rule
--post='text/html:markitdown {} > {}.md'

# Multiple rules (separated by \n)
--post='image/*:convert {} {}.webp\ntext/html:markitdown {} > {}.md'

# Rules from a file (one per line, # comments supported)
--post=@rules.txt
```

MIME patterns support `*` globbing — `image/*` matches `image/png`, `image/jpeg`, etc. Use `*` or `*/*` to match everything.

Example rules file:
```
# Convert HTML to markdown
text/html: markitdown {} > {}.md

# Compress all images to webp
image/*: convert {} {}.webp
```

## Custom Fetch Method

By default, wgetjs uses headless Chrome via the Chrome DevTools Protocol. If you prefer a different renderer, use `--method` with a command that outputs HTML to stdout:

```bash
# URL is appended by default
wgetjs --method="lightpanda --dump markdown" http://somesite.com/

# Or use {} for explicit URL placement (like --post)
wgetjs --method="curl -s {} | tidy -q -" http://somesite.com/
wgetjs --method="playwright fetch {} --html" http://somesite.com/
```
