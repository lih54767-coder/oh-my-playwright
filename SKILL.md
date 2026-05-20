---
name: oh-my-playwright
description: 'Website exploration for testing using Playwright MCP. Auto-detects login walls, handles authentication, manages browser sessions. Handles first-time setup of dependencies, extensions, and config. Use when asked to explore, test, or audit a website. Triggers on: explore website, test website, browser automation, 网站探索, 网站测试.'
---

# Website Exploration for Testing

Explore a website, identify key functionalities, and generate test cases.
Includes **login detection**, **authentication state management**, and **first-time setup**.

---

## Phase 0: First-Time Setup

**Goal: Ask everything in ONE question, generate the FINAL config, restart ONCE.**

### 0.1 Detect current state

Read `~/.config/opencode/opencode.json`, find `mcp.playwright` section.
Also try calling a Playwright MCP tool to check if it's alive.

Based on what you find, determine:
- Is Playwright MCP configured? What mode? Is it working?
- Is there `--headless`? (MUST be flagged as a problem)
- Is there `--extension`? `--cdp-endpoint`? `--user-data-dir`?

If MCP is already configured AND working AND not headless → skip to Phase 1.

Otherwise, proceed to 0.2.

### 0.2 The ONE big question

Use `question` tool with THREE questions in a single call.
This is the ONLY interaction before config is written.

**IMPORTANT**: Every question MUST have a clear "(Recommended)" label on the recommended option.
Users often don't know what to choose — the recommendation is critical.

**Question 1: Connection mode**

```
header: "连接方式"
question: "Playwright MCP 如何连接你的浏览器？这决定了 AI 能否复用你的登录状态。"
options:
  - label: "Extension 模式 (Recommended)"
    description: "AI 通过浏览器扩展控制你已打开的标签页。你已登录的网站直接可用，无需重新登录。需要安装 Playwright MCP Bridge 扩展。"
  - label: "Persistent Profile 模式"
    description: "AI 启动一个独立的浏览器，登录一次后自动保存。不想让 AI 碰你真实浏览器时选这个。"
  - label: "CDP 模式"
    description: "AI 通过 Chrome DevTools Protocol 直连你的浏览器。需要开启远程调试。功能最强但设置最复杂。"
```

**Question 2: Browser**

```
header: "浏览器"
question: "你日常主要用哪个浏览器？"
options:
  - label: "Microsoft Edge (Recommended)"
    description: "Windows 默认浏览器，大多数 Windows 用户的首选"
  - label: "Google Chrome"
    description: "如果 Chrome 是你的主力浏览器，选这个"
  - label: "其他 / 都没有"
    description: "AI 将使用 Playwright 自带的 Chromium"
```

**Question 3: Headless**

```
header: "无头模式"
question: "是否以无头（不可见）模式运行浏览器？⚠️ 无头模式下你看不到浏览器窗口，遇到需要登录（MFA、二维码、验证码）的网站将无法操作。"
options:
  - label: "No - 可见模式 (Recommended)"
    description: "浏览器窗口可见，你可以手动登录、扫码、输入验证码。适合需要登录网站的场景。"
  - label: "Yes - 无头模式"
    description: "浏览器在后台运行，不显示窗口。仅适合测试公开网站（不需要登录的场景）。"
```

### 0.3 Generate the final config

Based on the answers, generate the EXACT config line:

| Mode | Browser | Config command array |
|------|---------|---------------------|
| Extension | Edge | `["npx", "@playwright/mcp@latest", "--extension", "--browser", "msedge"]` |
| Extension | Chrome | `["npx", "@playwright/mcp@latest", "--extension", "--browser", "chrome"]` |
| Extension | Other | `["npx", "@playwright/mcp@latest", "--extension"]` |
| Persistent | Edge | `["npx", "@playwright/mcp@latest", "--channel", "msedge", "--user-data-dir", "<platform-path>"]` |
| Persistent | Chrome | `["npx", "@playwright/mcp@latest", "--channel", "chrome", "--user-data-dir", "<platform-path>"]` |
| Persistent | Other | `["npx", "@playwright/mcp@latest", "--user-data-dir", "<platform-path>"]` |
| CDP | Edge | `["npx", "@playwright/mcp@latest", "--cdp-endpoint=msedge"]` |
| CDP | Chrome | `["npx", "@playwright/mcp@latest", "--cdp-endpoint=chrome"]` |
| CDP | Other | `["npx", "@playwright/mcp@latest", "--cdp-endpoint=chrome"]` |

If headless = Yes, append `"--headless"` to the array.

Platform paths for `--user-data-dir`:
- Windows: `C:\\Users\\<username>\\.playwright-profile`
- macOS: `/Users/<username>/.playwright-profile`
- Linux: `/home/<username>/.playwright-profile`

### 0.4 Pre-restart checklist

Before modifying config, check prerequisites based on the chosen mode:

**If Extension mode:**
1. Ask user: "Have you installed the Playwright MCP Bridge extension?"
   - Link: https://chromewebstore.google.com/detail/playwright-mcp-bridge/mmlmfjhmonkocbjadbfplnigmagldckm
   - If not installed: instruct user to install NOW, before restarting
   - "Open this link in your Edge/Chrome, click 'Add to Chrome' or 'Get', confirm installation"

**If CDP mode:**
1. Ask user: "Have you enabled remote debugging in your browser?"
   - Chrome: navigate to `chrome://inspect/#remote-debugging`, enable "Allow remote debugging"
   - Edge: navigate to `edge://inspect/#remote-debugging`, enable "Allow remote debugging"
   - If not done: instruct user to do it NOW

**If Persistent Profile mode:**
1. Create the profile directory: `mkdir -p <path>` (or equivalent on the platform)

**All modes:**
1. Run `npx playwright install chromium` to ensure fallback browser is available

### 0.5 Apply config and restart ONCE

1. Edit `~/.config/opencode/opencode.json`, update `mcp.playwright.command`
2. Tell the user exactly what was changed and why
3. Say: "All done. Please restart opencode now. After restart, everything should work — no more config changes needed."

**The config is now FINAL. The user should NOT need to restart again for Playwright reasons.**

### 0.6 Quick reference for the user

After restart, inform the user:

> "Playwright MCP is configured in **[mode name]** mode with **[browser name]**.
>
> What this means:
> - **Extension mode**: Keep your browser open with the extension active. AI can see and control your tabs. Your existing logins work.
> - **Persistent Profile mode**: AI launches its own browser window. Log in once, sessions persist. If login expires, you'll see the browser window and can re-login manually.
> - **CDP mode**: AI connects to your running browser directly. Keep remote debugging enabled.
>
> To switch modes later, just tell me and I'll update the config."

---

## Phase 1: Navigate & Login Detection

### 1.1 Navigate

Navigate to the target URL using `browser_navigate`. If no URL was provided, ask the user.

### 1.2 Snapshot

After navigation, run `browser_snapshot` to inspect the page.

### 1.3 Login Detection

Check if the current page is a login/authentication wall. Indicators:

- **URL patterns**: `login`, `signin`, `auth`, `sso`, `oauth`, `account/login`, `idp-`, `authenticate`, `cas/`, `shibboleth`
- **Password field**: form with `type="password"` and minimal other content
- **Page title**: "Sign in", "Log in", "Login", "Authentication", "SSO", "验证", "登录"
- **OAuth buttons**: "Sign in with Google", "Continue with GitHub", etc.
- **CAPTCHA elements**: `g-recaptcha`, `cf-turnstile`, `h-captcha`
- **QR code login**: QR code image on page, instructions like "Scan with app"
- **SMS/Email verification**: fields asking for verification codes
- **Page content**: mostly a login form with no app functionality visible

**If login wall detected** → go to Phase 2.
**If no login wall** → skip to Phase 3.

---

## Phase 2: Authentication Handling

### 2.1 Analyze auth type

Classify what kind of authentication is required:

| Type | Indicators | Difficulty |
|------|-----------|------------|
| Simple form | username + password fields only | Easy |
| OAuth/SSO | "Sign in with X" buttons | Medium |
| MFA/2FA | Verification code field after password | Medium |
| QR code | QR code image displayed | Easy (scan) |
| CAPTCHA | reCAPTCHA/Turnstile/hCaptcha widget | Hard |
| SMS/Email code | "Enter the code sent to..." | Medium |
| Combination | Multiple of the above | Hard |

### 2.2 Inform the user

Tell the user:
- What site requires login
- What type of auth was detected
- The browser window is visible — they can interact with it directly

### 2.3 Wait for user to complete login

Use `question` tool:

> "The browser window should be visible on your screen. Please log in manually.
> You can use any method (password, QR code, SMS, OAuth, etc.).
> Let me know when you're done."

**Important**: Do NOT attempt to:
- Fill in passwords automatically
- Click CAPTCHA elements
- Solve verification challenges
- Click OAuth buttons without user consent

### 2.4 Verify login

After user confirms:
1. Run `browser_snapshot` to check the current page
2. If the page has moved past the login wall (no more login form visible) → proceed to Phase 3
3. If still on login page → ask the user to try again

### 2.5 Fallback: connect to user's real browser

If manual login in Playwright's browser doesn't work (e.g., site blocks automated browsers, or user prefers their own browser):

1. Ask the user to open the target site in their own browser and log in there
2. Offer to switch to one of these modes:

Use `question` tool:

> "Would you like to switch to connecting to your real browser?
> This will let AI use your already-logged-in session.
>
> Options:
> - Extension mode: AI controls tabs via the Playwright MCP Bridge extension
> - CDP mode: AI connects directly to your running browser"

Based on their choice, update the opencode config:
- Extension: `["npx", "@playwright/mcp@latest", "--extension"]`
- CDP Edge: `["npx", "@playwright/mcp@latest", "--cdp-endpoint=msedge"]`
- CDP Chrome: `["npx", "@playwright/mcp@latest", "--cdp-endpoint=chrome"]`

Inform user: "Config updated. Please restart opencode."

### 2.6 Save auth state (optional)

After successful login, offer to save the session for future use:

Use `browser_run_code`:
```javascript
async (page) => { return await page.context().storageState(); }
```

Save to a project-local file like `.playwright/auth-state.json`.
Add `.playwright/` to `.gitignore` if not already there.

---

## Phase 3: Explore

### 3.1 Identify features

Identify 3-5 core features or user flows from the current page.

### 3.2 Interact with each feature

For each feature:
1. `browser_snapshot` → understand page structure
2. Interact (click, type, navigate)
3. `browser_snapshot` → observe result
4. Document: interaction steps, UI elements (with accessibility refs), expected outcome

### 3.3 Visual documentation

Use `browser_take_screenshot` at key points for visual records.

---

## Phase 4: Document & Generate Tests

### 4.1 Close browser

Close the browser context using `browser_close`.

### 4.2 Summary

Provide a concise summary:
- Site overview and authentication requirements
- Features explored with UI element references
- Issues or unexpected behaviors found

### 4.3 Test cases

Generate structured test cases:
- Happy path for each explored feature
- Edge cases based on observations
- Auth-related tests (login/logout/session expiry) if applicable

Format:
```
### TC-XX: <Test name>
- Preconditions: <including auth state>
- Steps:
  1. ...
- Expected: <result>
```

---

## Rules

- **Never** attempt to fill passwords or submit login forms without user consent
- **Never** click CAPTCHA elements or try to solve them
- **Never** store or log credentials
- **Never** click "Sign out" / "Log out" links during exploration
- **Never** assume which browser the user prefers — always ask
- **Never** skip the headless check — invisible browser = user can't log in
- **Always** use `browser_snapshot` for element discovery, not screenshots
- **Always** use `question` tool when user action or decision is needed
- **Always** support both Chrome and Edge — ask which one the user uses

---

## Core Discipline: Observe → Act → Verify (OAV)

**This is the single most important operational discipline. Every web interaction MUST follow this loop.**

Browser automation is inherently unreliable — elements may not exist, may not be visible, may not respond to clicks the way you expect, or may use frameworks (Vue, React, Angular) that manage state internally and won't react to DOM manipulation. **You cannot assume your action succeeded. You must verify.**

### The OAV Loop

```
┌──────────────────────────────────────────────┐
│  1. OBSERVE: Read the page BEFORE acting     │
│     - What elements exist? What's visible?    │
│     - What framework is in use?               │
│     - What's the current state?               │
│                                               │
│  2. ACT: Perform ONE atomic action            │
│     - Prefer Playwright native click/type     │
│     - Avoid JS evaluate for state changes     │
│                                               │
│  3. VERIFY: Re-read the page AFTER acting     │
│     - Did the state change as expected?       │
│     - Check framework-specific indicators     │
│     - If not expected → diagnose before retry │
└──────────────────────────────────────────────┘
```

### Rule 1: Observe Before You Act

**Never** execute an action without first understanding the current page state. Blind clicks are the #1 source of wasted iterations.

Before clicking/typing anything:
1. Run `browser_snapshot` or `browser_evaluate` to inspect the target element
2. Confirm the element **exists**, is **visible** (`offsetWidth > 0`), and is **interactable** (not disabled, not covered by a dialog)
3. For form elements: understand the framework's state management. Does it use native DOM state (`input.checked`, `selected`) or internal state (Vue `data()`, React `state`)? **If the latter, JS DOM manipulation will NOT work** — you must use Playwright's native click to trigger the framework's event handlers.

**Common pitfall**: `document.querySelector('label').click()` via `evaluate` may set `input.checked = true` but Vue/React won't update their internal state. The UI looks selected but the framework doesn't know about it. **Always use Playwright's native `browser_click` for state-changing interactions.**

### Rule 2: Verify After Every Action

**Every** click, type, navigate, or evaluate that is supposed to change page state **MUST** be followed by a verification step. No exceptions.

After acting:
1. Re-read the page (snapshot or evaluate) to confirm the expected change occurred
2. Check **framework-specific** indicators, not just DOM state:
   - Vue: look for CSS classes like `is-checked`, `is-checked-alone`, `active`, `is-focus`
   - React: look for state-indicating attributes or classes
   - Custom components: look for class changes, attribute changes, `aria-*` attributes
3. If the action didn't produce the expected result → **STOP and diagnose**. Do not retry the same action. Understand WHY it failed before trying again.

**Verification checklist** (internalize this):

```
- Did the element's CSS classes change? (e.g., appeared `is-checked`)
- Did the element's attributes change? (e.g., `aria-checked="true"`)
- Did surrounding content change? (e.g., progress counter updated)
- Did a new element appear or disappear? (e.g., dialog opened/closed)
- Did the URL change? (e.g., navigation occurred)
```

### Rule 3: Use Playwright Native Clicks for State Changes

**Critical distinction**: `browser_evaluate(() => el.click())` is NOT the same as `browser_click`.

| Action | Playwright native (`browser_click`) | JS evaluate (`el.click()`) |
|--------|-------------------------------------|---------------------------|
| Dispatches browser events | ✅ Full event chain (mousedown→mouseup→click) | ❌ Only `click` event |
| Triggers framework listeners | ✅ Yes | ❌ May not |
| Works with shadow DOM | ✅ Yes | ❌ Usually not |
| Updates framework state | ✅ Yes (Vue, React, etc.) | ❌ Only if framework listens to native events |
| Speed | Slower (one at a time) | Faster (batch possible) |

**When to use what**:
- **State changes** (clicking buttons, checking checkboxes, selecting radio buttons, toggling dropdowns): **ALWAYS use Playwright native `browser_click`**. Never use JS evaluate for these.
- **Reading data** (extracting text, checking state, counting elements): **Use `browser_evaluate`**. This is safe because you're not changing anything.
- **Batch operations on many elements**: Use `browser_run_code_unsafe` with Playwright's `page.locator().click()` — this is Playwright native click, not JS evaluate click.

### Rule 4: Batch Operations Must Still Verify

When clicking many elements in a batch (e.g., 40+ checkboxes):
1. Use `browser_run_code_unsafe` with `page.locator(selector).nth(i).click()` — this is native Playwright click
2. After the batch, run a verification pass:
   - Count how many elements are in the expected state (framework classes, not just DOM attributes)
   - Compare against expected count
   - Report any mismatches
3. **Never** assume "I clicked 115 elements so they must all be selected." Verify.

### Rule 5: When Things Go Wrong

If verification shows the action didn't work:
1. **Don't retry the same way**. That's the definition of insanity.
2. **Diagnose first**: Read the element's full class list, parent structure, and framework indicators
3. **Try a different approach**:
   - JS click failed → try Playwright native click
   - `browser_click` on the label failed → try clicking the inner `input` element
   - Selector too generic → use a more specific selector
   - Element not visible → scroll to it first, or dismiss blocking dialogs
4. **Verify again** after each new approach

### Rule 6: Never Assume Page Content Is Stable (THE #1 LESSON)

**This rule exists because of a real incident that cost a 95.6→9 score collapse.**

Web pages are **dynamic**. Every page load, every navigation, every session may present:
- **Different ordering** of items (questions, options, list items, table rows)
- **Different content** (randomized questions, A/B variants, time-based content)
- **Different structure** (responsive layouts, conditional sections, lazy-loaded elements)

**Hard rules — zero tolerance**:

1. **Never cache page content across navigations or sessions.** What you saw on the previous page load is INVALID after any navigation, refresh, or "new attempt" action. Re-read everything.

2. **Never map by position ("the 3rd item") when content matters.** If you need to select "option B" because the text says "follow minimum privilege principle", you must READ the text of each option on the CURRENT page and match by CONTENT, not by index.

3. **Always re-extract before acting.** Before selecting/typing/clicking based on content you "remember" from a previous interaction:
   ```
   # WRONG: "I remember question 1's answer is B"
   click(option_B_on_question_1)
   
   # RIGHT: "Let me read what question 1 actually says NOW"
   evaluate(() => getQuestionText(1))
   # → "关于MCP架构..." — wait, this is a different question!
   # → Re-analyze, THEN click the correct option
   ```

4. **When doing repeated operations (e.g., retaking an exam, re-submitting a form)**, treat EVERY attempt as if it's your FIRST time seeing the page. Previous attempt's data is **historical reference only**, not a copy-paste template.

5. **If the operation involves understanding content (answering questions, filling forms with domain-specific data)**, ALWAYS use a sub-agent to analyze the content fresh. The sub-agent has no memory bias — it will read what's actually there.

**The pattern for content-dependent operations**:
```
Step 1: EXTRACT — Read all relevant content from the current page
Step 2: ANALYZE — Use sub-agent or reasoning to determine correct actions based on CONTENT
Step 3: EXECUTE — Use Playwright native clicks to perform actions
Step 4: VERIFY — Re-read the page to confirm actions took effect
```

**Why this matters beyond exams**: Imagine you're automating a deployment dashboard. You "remember" the staging server is the 3rd option. But someone reordered the list. You just deployed to production. **Same root cause.** Content instability is not an edge case — it's the default assumption.

### Rule 7: Content-First Selection Strategy

When you need to select options based on their semantic meaning (not just their position):

1. **Extract all option texts** using `browser_evaluate`
2. **Match by text content**, not by index or position
3. **Tag matching elements** with `data-` attributes based on the analysis result
4. **Click using Playwright native** on the tagged elements
5. **Verify** that the selected options' text matches what you intended

Example for a form with dynamic option order:
```javascript
// Step 1: Extract
const options = document.querySelectorAll('.option');
const optionTexts = Array.from(options).map(o => o.textContent);

// Step 2: Match by content (NOT by index)
const targetText = "minimum privilege principle";
const targetOption = Array.from(options).find(o => o.textContent.includes(targetText));

// Step 3: Tag for Playwright click
if (targetOption) targetOption.setAttribute('data-target', 'true');

// Step 4: Click via Playwright native (browser_click or run_code_unsafe)
// Step 5: Verify the correct option was selected
```

---

## Anti-Patterns (Lessons from Real Failures)

These are documented mistakes that wasted real time. Don't repeat them.

### ❌ Anti-Pattern 1: "Fire and Forget" Clicking

```
# BAD: Click 115 checkboxes via JS evaluate, then submit without verifying
browser_evaluate(() => { labels.forEach(l => l.click()); return 'done'; })
# → 0/40 correct because Vue didn't register the clicks
```

```
# GOOD: Click via Playwright native, then verify each one
browser_run_code_unsafe(async (page) => {
  for (const sel of selectors) { await page.locator(sel).click(); }
});
browser_evaluate(() => checkVueState());
```

### ❌ Anti-Pattern 2: Assuming DOM State = Framework State

```
# BAD: Check input.checked and assume the framework knows
input.checked === true  // ← DOM says yes, but Vue data model says no
```

```
# GOOD: Check framework-specific indicators
label.classList.contains('is-checked')  // ← Vue actually knows about this one
```

### ❌ Anti-Pattern 3: Blind Selector Guessing

```
# BAD: Try 5 different selectors hoping one works
browser_click('.checkbox-group label:nth-child(2)')   # fail
browser_click('[role="checkbox"]:nth-child(2)')       # fail
browser_click('input[type="checkbox"]')                # wrong element
# ... 3 more attempts
```

```
# GOOD: Inspect first, then click with confidence
browser_evaluate(() => {
  // Find the actual element, its classes, its structure
  return { selector: '...', classes: [...], parent: '...' };
});
browser_click(actualSelector);
browser_evaluate(() => verify());
```

### ❌ Anti-Pattern 4: Skipping Verification Before Submit

```
# BAD: Fill form, immediately submit
browser_click('#option-b');
browser_click('#submit');
# → Wrong answer because option wasn't actually selected
```

```
# GOOD: Verify selection state before submitting
browser_click('#option-b');
browser_evaluate(() => {
  return document.querySelector('#option-b').classList.contains('selected');
});
# Confirmed? Now submit.
browser_click('#submit');
```

### ❌ Anti-Pattern 5: Assuming Content Order Is Stable (COST: 95.6→9)

```
# BAD: Memorize "question 1 = option B" from a previous attempt
# and blindly re-apply the same answer map
const answers = { 1: 'B', 2: 'A', 3: 'B', ... };
clickAnswer(1, answers[1]);
# → Question 1 changed to a DIFFERENT question
# → You just selected the wrong answer for a question you would have known
# → Score: 9/100. Worse than random guessing.
```

```
# GOOD: Re-read every question, match answers to CONTENT
# Extract all questions
const questions = extractAllQuestions();
# Send to sub-agent for analysis (fresh, no cached assumptions)
const answers = analyzeQuestions(questions);
# Match answers to actual option TEXT on the current page
for (const [qIdx, answer] of Object.entries(answers)) {
  const option = findOptionByText(qIdx, answer);
  option.setAttribute('data-target', 'answer');
}
# Click via Playwright native
# Verify each selection
```

**This anti-pattern applies everywhere, not just exams:**
- Re-running a deployment script? The server list order may have changed.
- Re-submitting a form? Dropdown options may have been updated.
- Re-running a test? Test data may have been refreshed.
- **Always re-read before re-act.**
