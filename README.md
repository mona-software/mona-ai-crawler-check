# mona-ai-crawler-check

Checks whether AI crawlers such as GPTBot, ClaudeBot and PerplexityBot are allowed to fetch a web page and whether the page gives them readable content.

It reports `PASS`, `WARN` or `FAIL` for each check, with a suggested fix. It is part of [MONA GEO OS](https://mona.media/mona-geo-os/).

## What it checks

- **robots.txt**, per AI user-agent token: `GPTBot`, `OAI-SearchBot`, `ChatGPT-User`, `ClaudeBot`, `Claude-Web`, `Google-Extended`, `PerplexityBot`, `CCBot`, `Bytespider`, `Amazonbot`, `meta-externalagent`. A token blocked for the requested path is `FAIL`. A missing `robots.txt` counts as allowed.
- **Server-rendered content**: the page must return 2xx with visible text in the initial HTML. A page with fewer than 80 visible characters plus scripts (a JavaScript-only shell) is `FAIL`.
- **llms.txt**: `/llms.txt` exists at the site root (HEAD, falling back to GET). Missing is `WARN`.
- **JSON-LD**: the page contains structured data. Missing is `WARN`.

The JSON report also records whether the page has a `<title>`, a meta description and an `h1`.

## Install

Requires Python 3.9+. Standard library only.

```bash
git clone https://github.com/mona-software/mona-ai-crawler-check
cd mona-ai-crawler-check
pip install -e .
```

## Usage

```bash
mona-ai-crawler-check https://your-site.com
mona-ai-crawler-check https://your-site.com/some/page --json
python -m mona_ai_crawler_check your-site.com     # https:// is assumed when omitted
```

Exit codes: `0` overall `PASS` or `WARN`, `1` at least one `FAIL`, `2` the URL is invalid or could not be fetched.

Offline demo with the bundled fixtures (run after `pip install -e .`):

```bash
python examples/demo.py
```

```
+----------------------------+--------+----------------------------------+
| CHECK                      | STATUS | DETAIL                           |
+----------------------------+--------+----------------------------------+
| Overall                    | WARN   | https://example.com              |
| robots: GPTBot             | PASS   | Allowed for requested path       |
| ...                        |        |                                  |
| robots: meta-externalagent | PASS   | Allowed for requested path       |
| Server content             | PASS   | 171 visible characters           |
| llms.txt                   | WARN   | /llms.txt was not found          |
| JSON-LD                    | WARN   | No JSON-LD structured data found |
+----------------------------+--------+----------------------------------+
Fixes:
- llms.txt: Generate and publish /llms.txt; you can use mona-llms-txt.
- JSON-LD: Add relevant schema.org JSON-LD to the server-rendered HTML.
```

`--json` prints `url`, `status`, `robots` (one entry per bot), `content`, `llms_txt`, `json_ld`, `metadata` (`title`, `meta_description`, `h1`) and `issues`. Each check has `name`, `status`, `detail` and `fix`.

## Library use

```python
from mona_ai_crawler_check import check_site

report = check_site("https://your-site.com")
print(report.render_text())   # table plus suggested fixes
print(report.to_json())
```

`check_site(url, fetcher=...)` accepts any object with `get` and `head` methods, for tests or caching. The building blocks `parse_robots`, `bot_allowed`, `analyze_content` and `check_llms_txt` can also be imported from the package.

When run against a live site, the tool requests only `/robots.txt`, `/llms.txt` and the given page.

## Development

```bash
pip install -e ".[test]"
pytest -q
```

Tests run offline against hand-written fixtures in `fixtures/`.

## License

MIT, see [LICENSE](LICENSE).

**`mona-ai-crawler-check` is a product of MONA Software, a member of The MONA Group.**
