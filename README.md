# oh-my-playwright

An [opencode](https://github.com/anomalyco/opencode) skill for browser automation and website exploration with **automatic login detection, authentication handling, and zero-config-first-time setup**.

## What it does

- **Explores websites** and identifies key functionalities
- **Auto-detects login walls** — recognizes 7+ auth types (password form, OAuth, SSO, MFA, QR code, CAPTCHA, SMS)
- **Handles authentication** — guides users through manual login in a visible browser, or connects to their already-logged-in browser
- **One-time setup** — asks 3 questions, generates final config, restart once and done
- **Generates test cases** from exploration results

## How it works

### First-time setup (Phase 0)

On first use, the skill detects your current Playwright MCP config and walks you through setup with 3 questions:

1. **Connection mode** — Extension (recommended), Persistent Profile, or CDP
2. **Browser** — Edge (recommended), Chrome, or built-in Chromium
3. **Headless** — visible (recommended) or invisible

It then generates the correct config for your combination, checks prerequisites, and applies it. You restart opencode once and never touch the config again.

### Login detection (Phase 1)

When navigating to a website, the skill automatically detects if the page is a login wall by checking:
- URL patterns (`login`, `signin`, `auth`, `sso`, `oauth`, etc.)
- Password fields, OAuth buttons, CAPTCHA elements
- QR code images, SMS verification prompts
- Page title and content analysis

### Authentication handling (Phase 2)

If a login wall is detected:
1. Informs the user what auth type was found
2. User manually logs in (supports ALL methods: password, QR code, MFA, SSO, CAPTCHA)
3. Skill verifies login success and continues
4. Offers to save session state for future use

If login fails (e.g., site blocks automated browsers), the skill can switch to connecting to the user's real browser.

### Exploration & test generation (Phase 3-4)

- Identifies 3-5 core features
- Interacts with each feature and documents UI elements
- Generates structured test cases

## Requirements

- [opencode](https://github.com/anomalyco/opencode) CLI
- Node.js 18+
- Playwright MCP (`npx @playwright/mcp@latest`)
- One of:
  - **Extension mode**: Playwright MCP Bridge browser extension
  - **Persistent Profile mode**: nothing extra
  - **CDP mode**: remote debugging enabled in your browser

## Installation

```bash
# Clone into your opencode skills directory
git clone https://github.com/<your-username>/oh-my-playwright ~/.opencode/skills/oh-my-playwright
```

Or manually:
1. Create directory `~/.opencode/skills/oh-my-playwright/`
2. Copy `SKILL.md` into it

## Configuration

The skill will guide you through configuration on first use. To manually configure Playwright MCP in `~/.config/opencode/opencode.json`:

**Extension mode (recommended):**
```json
"playwright": {
    "type": "local",
    "command": ["npx", "@playwright/mcp@latest", "--extension", "--browser", "msedge"],
    "enabled": true
}
```

**Persistent Profile mode:**
```json
"playwright": {
    "type": "local",
    "command": ["npx", "@playwright/mcp@latest", "--channel", "msedge", "--user-data-dir", "C:\\Users\\<you>\\.playwright-profile"],
    "enabled": true
}
```

## Attribution

This skill was originally based on the `playwright-explore-website` template skill bundled with opencode. It has been substantially rewritten with:

- Complete login detection and authentication handling (Phase 1-2)
- One-time setup wizard with config generation (Phase 0)
- Multi-browser support (Edge, Chrome, Chromium)
- Anti-idle detection for automated video playback scenarios
- Chinese language support in detection patterns

The original template contained only a 17-line instruction prompt. This skill is a ground-up rewrite at 300+ lines.

## License

MIT
