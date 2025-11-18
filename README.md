# Claude Code Hooks

Professional hooks for [Claude Code](https://claude.ai/code) that automatically log your development workflow with timestamps, UTF-8 support, and markdown formatting.

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Quick Start](#quick-start)
- [Available Hooks](#available-hooks)
  - [Log User Prompts](#log-user-prompts)
  - [Log Assistant Work](#log-assistant-work)
- [Installation](#installation)
- [Configuration](#configuration)
- [Log File Structure](#log-file-structure)
- [Implementation Details](#implementation-details)
- [Troubleshooting](#troubleshooting)
- [Customization](#customization)
- [FAQ](#faq)
- [Contributing](#contributing)
- [License](#license)

---

## Overview

This hook collection provides automatic logging for Claude Code sessions:

- **User Prompts** → `PROMPTLOG.md` - Every question/request you send to Claude
- **Assistant Responses** → `WORKLOG.md` - Summary of work completed by Claude

Both logs use markdown format with timestamps, support UTF-8 (Vietnamese characters, emojis), auto-archive when exceeding 1000 lines, and are automatically gitignored.

## Features

✅ **Automatic Logging** - No manual intervention required
✅ **UTF-8 Support** - Perfect handling of Vietnamese text, emojis, and special characters
✅ **Markdown Format** - Easy to read, search, and version control
✅ **Cross-Platform** - Works on Windows (PowerShell) and Linux/macOS (Bash)
✅ **Auto-Archiving** - Files exceeding 1000 lines automatically move to `docs/claude_code/`
✅ **IDE Context** - Captures file selections and opened files with clean formatting
✅ **Gitignored** - Log files excluded from version control by default
✅ **Timezone Support** - Configurable timezone (default: GMT+7 Vietnam)

---

## Quick Start

### 1. Clone/Copy Hook Files

```bash
# Copy hook files to your project
.claude/
├── hooks/
│   ├── log-user-prompts.ps1       # Windows - User prompts
│   ├── log-user-prompts.sh        # Linux/macOS - User prompts
│   ├── log-assistant-work.ps1     # Windows - Assistant responses
│   └── log-assistant-work.sh      # Linux/macOS - Assistant responses
```

### 2. Configure Hooks

Add to `.claude/settings.local.json`:

**Windows:**
```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "powershell.exe -ExecutionPolicy Bypass -File .claude/hooks/log-user-prompts.ps1"
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "powershell.exe -ExecutionPolicy Bypass -File .claude/hooks/log-assistant-work.ps1"
          }
        ]
      }
    ]
  }
}
```

**Linux/macOS:**
```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "bash .claude/hooks/log-user-prompts.sh"
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "bash .claude/hooks/log-assistant-work.sh"
          }
        ]
      }
    ]
  }
}
```

### 3. Set Permissions (Linux/macOS only)

```bash
chmod +x .claude/hooks/log-user-prompts.sh
chmod +x .claude/hooks/log-assistant-work.sh
```

### 4. Install Dependencies (Linux/macOS only)

```bash
# For log-assistant-work.sh - requires jq for JSON parsing
sudo apt install jq           # Ubuntu/Debian
brew install jq               # macOS
```

### 5. Start Logging

That's it! Your prompts and Claude's responses will now be automatically logged to `PROMPTLOG.md` and `WORKLOG.md`.

---

## Available Hooks

### 📝 Log User Prompts

**Files:** `log-user-prompts.ps1`, `log-user-prompts.sh`
**Trigger:** `UserPromptSubmit` - Runs when you submit a prompt (before Claude processes it)
**Output:** `PROMPTLOG.md` in project root

#### Features

- ✅ Captures every prompt sent to Claude
- ✅ Includes timestamp (Vietnamese timezone GMT+7)
- ✅ UTF-8 encoding with proper Vietnamese character support
- ✅ IDE context formatting (file selections, opened files) with blank lines
- ✅ Clean text formatting (no blockquote markers for easy copy-paste)
- ✅ Markdown structure validation (balanced code fences)
- ✅ Auto-creates log file if doesn't exist
- ✅ Auto-archives to `docs/claude_code/` when exceeds 1000 lines
- ✅ Gitignored by default

#### Log Format

```markdown
## 📝 Prompt - 2025-01-18 14:30:45

<ide_opened_file>The user opened the file src/index.ts in the IDE.</ide_opened_file>

<ide_selection>Selected code from lines 10-25</ide_selection>

How do I optimize this function for better performance?

---
```

#### How It Works

1. Hook receives JSON input: `{"prompt": "your text", "session_id": "...", ...}`
2. **Windows (log-user-prompts.ps1)**:
   - Reads stdin using `StreamReader` with UTF-8 encoding
   - Parses JSON to extract `"prompt"` field
   - Formats IDE tags (`<ide_opened_file>`, `<ide_selection>`) with blank lines after closing tags
   - Sanitizes markdown: escapes backticks (no blockquote markers for readability)
   - Validates code fence balance (wraps in code block if unbalanced)
   - Writes to file using UTF-8 without BOM
   - Returns JSON using `StreamWriter` with UTF-8
3. **Linux/macOS (log-user-prompts.sh)**:
   - Uses `jq` for JSON parsing (UTF-8 handled natively by bash)
   - Formats IDE tags with blank lines after closing tags
   - Sanitizes markdown: escapes backticks (no blockquote markers)
   - Validates code fence balance
   - Returns original JSON

---

### 🤖 Log Assistant Work

**Files:** `log-assistant-work.ps1`, `log-assistant-work.sh`
**Trigger:** `Stop` - Runs when Claude Code completes a response (after processing)
**Output:** `WORKLOG.md` in project root

#### Features

- ✅ Saves summary of work completed by Claude
- ✅ Includes timestamp (Vietnamese timezone GMT+7)
- ✅ UTF-8 encoding with emoji support (🤖)
- ✅ Extracts assistant messages from transcript file
- ✅ Auto-creates log file if doesn't exist
- ✅ Auto-archives to `docs/claude_code/` when exceeds 1000 lines
- ✅ Gitignored by default

#### Log Format

```markdown
## 🤖 Work Completed - 2025-01-18 14:32:10

I've updated the authentication module with the following changes:
- Added JWT token validation
- Implemented refresh token logic
- Updated unit tests to cover new scenarios

---
```

#### How It Works

1. Hook receives JSON with `transcript_path` field
2. Reads JSONL transcript file (`.claude/projects/{project}/{session_id}.jsonl`)
3. Extracts all assistant messages with `role: "assistant"`
4. Uses the **last assistant message** as work summary
5. Logs clean work summary to WORKLOG.md
6. Returns original JSON to Claude Code

**Implementation:**
- **Windows (log-assistant-work.ps1)**: Uses PowerShell with .NET `ReadAllLines()` for UTF-8 handling
- **Linux/macOS (log-assistant-work.sh)**: Uses `jq` for JSON parsing (requires installation)

---

## Installation

### Method 1: Direct Copy

1. Copy hook files to `.claude/hooks/` directory in your project
2. Update `.claude/settings.local.json` with configuration (see [Quick Start](#quick-start))
3. Set executable permissions (Linux/macOS): `chmod +x .claude/hooks/*.sh`
4. Install `jq` (Linux/macOS): `sudo apt install jq` or `brew install jq`

### Method 2: Git Submodule (Recommended for teams)

```bash
# Add hooks as a submodule
git submodule add https://github.com/your-username/claude-code-hooks .claude/hooks

# Initialize submodule
git submodule update --init --recursive

# Configure hooks (add to .claude/settings.local.json)
# See Quick Start section
```

### Method 3: NPM Package (Future)

```bash
npm install --save-dev claude-code-hooks
```

---

## Configuration

### Hook Settings Location

All hook configurations are stored in `.claude/settings.local.json`.

### Enable/Disable Hooks

**To disable a hook:**
```json
{
  "hooks": {
    "UserPromptSubmit": [],  // Empty array disables the hook
    "Stop": []
  }
}
```

**To re-enable:**
- Add the hook configuration back (see [Quick Start](#quick-start))
- Modify the hook file timestamp to trigger re-activation

### Common Issues

#### Hook Disabled by Claude Code

If you see `stop_hook_active: false` in logs:

**Cause:** Claude Code automatically disables hooks that error, timeout, or cause infinite loops.

**Solution:**
1. Fix the error in your hook script
2. Modify the hook file (touch timestamp):
   ```bash
   touch .claude/hooks/log-assistant-work.ps1
   ```
3. Or restart Claude Code session

---

## Log File Structure

```
project-root/
├── .claude/
│   ├── hooks/
│   │   ├── README.md                    # This file
│   │   ├── log-user-prompts.sh          # Prompt hook (Linux/Mac)
│   │   ├── log-user-prompts.ps1         # Prompt hook (Windows)
│   │   ├── log-assistant-work.sh        # Work hook (Linux/Mac)
│   │   └── log-assistant-work.ps1       # Work hook (Windows)
│   └── settings.local.json              # Hook configuration
│
├── PROMPTLOG.md                          # User prompts (gitignored)
├── WORKLOG.md                            # Work summaries (gitignored)
│
└── docs/
    └── claude_code/                      # Archived logs
        ├── PROMPTLOG_20250118_143045.md
        └── WORKLOG_20250118_143045.md
```

### Auto-Archiving

When a log file exceeds 1000 lines:
1. Current file is moved to `docs/claude_code/LOGNAME_YYYYMMDD_HHMMSS.md`
2. New empty log file is created
3. Message is printed to stderr: `📦 Archived PROMPTLOG.md -> docs/claude_code/... (1234 lines)`

---

## Implementation Details

### UTF-8 Encoding Handling

#### Windows PowerShell

PowerShell's built-in `$input` variable and `Write-Output` cmdlet don't handle UTF-8 properly for Vietnamese text and emojis. The solution uses .NET `StreamReader`/`StreamWriter`:

```powershell
# UTF-8 encoder without BOM
$utf8NoBom = New-Object System.Text.UTF8Encoding $false

# Force UTF-8 for console I/O
[Console]::InputEncoding = [System.Text.Encoding]::UTF8
[Console]::OutputEncoding = [System.Text.Encoding]::UTF8

# Read stdin with UTF-8 encoding
$stdin = [System.Console]::OpenStandardInput()
$reader = New-Object System.IO.StreamReader($stdin, [System.Text.Encoding]::UTF8)
$jsonInput = $reader.ReadToEnd()
$reader.Close()

# Write to file with UTF-8 no BOM
[System.IO.File]::AppendAllText($filePath, $content, $utf8NoBom)

# Write stdout with UTF-8 encoding
$stdout = [System.Console]::OpenStandardOutput()
$writer = New-Object System.IO.StreamWriter($stdout, [System.Text.Encoding]::UTF8)
$writer.WriteLine($jsonInput)
$writer.Flush()
$writer.Close()
```

#### Linux/macOS Bash

Bash handles UTF-8 natively on modern systems. No special encoding setup needed:

```bash
# Read stdin (UTF-8 handled automatically)
INPUT=$(cat)

# Write to file (UTF-8 preserved)
cat >> "$LOG_FILE" << EOF
Content here with Vietnamese: tiếng Việt, emoji: 📝
EOF

# Echo back (UTF-8 preserved)
echo "$INPUT"
```

### Markdown Sanitization

Both scripts sanitize prompts for proper markdown rendering:

1. **IDE context formatting**: Adds blank lines after `<ide_opened_file>` and `<ide_selection>` closing tags
2. **Escape backticks**: Prevents code block conflicts (`` ` `` → `` \` ``)
3. **Clean formatting**: No blockquote markers for easy copy-paste
4. **Code fence validation**: Counts `` ``` `` occurrences
5. **Unbalanced fix**: Wraps entire prompt in code block if count is odd

#### PowerShell Example

```powershell
# Keep original formatting, only escape backticks
$sanitizedPrompt = $prompt -replace '`', '``'

# Format IDE tags with blank lines for better readability
if ($sanitizedPrompt -match '<ide_opened_file>|<ide_selection>') {
    # Add blank line after closing tags
    $sanitizedPrompt = $sanitizedPrompt -replace '(<\/ide_opened_file>)', "`$1`n"
    $sanitizedPrompt = $sanitizedPrompt -replace '(<\/ide_selection>)', "`$1`n"
}

# Validate code fence balance
$backtickCount = ([regex]::Matches($prompt, '```')).Count
if ($backtickCount % 2 -ne 0) {
    # Wrap in code block if unbalanced
    $sanitizedPrompt = "``````text`n$prompt`n``````"
}
```

#### Bash Example

```bash
# Keep original formatting, only escape backticks
SANITIZED_PROMPT=$(echo "$PROMPT" | sed 's/`/\\`/g')

# Format IDE tags with blank lines for better readability
if echo "$SANITIZED_PROMPT" | grep -q '<ide_opened_file>\|<ide_selection>'; then
    SANITIZED_PROMPT=$(echo "$SANITIZED_PROMPT" | sed 's#</ide_opened_file>#&\n#g')
    SANITIZED_PROMPT=$(echo "$SANITIZED_PROMPT" | sed 's#</ide_selection>#&\n#g')
fi

# Validate code fence balance
BACKTICK_COUNT=$(echo "$PROMPT" | grep -o '```' | wc -l)
if [ $((BACKTICK_COUNT % 2)) -ne 0 ]; then
    # Wrap in code block if unbalanced
    SANITIZED_PROMPT="\`\`\`text\n$PROMPT\n\`\`\`"
fi
```

### Transcript File Format

Claude Code stores conversation transcripts in JSONL format (one JSON object per line):

**Location:** `.claude/projects/{project_id}/{session_id}.jsonl`

**Format:**
```json
{"timestamp":"2025-01-18T07:30:45.123Z","message":{"role":"user","content":[{"type":"text","text":"Hello"}]}}
{"timestamp":"2025-01-18T07:30:47.456Z","message":{"role":"assistant","content":[{"type":"text","text":"Hi! How can I help?"}]}}
```

The hooks parse this file to extract assistant messages for work logging.

---

## Troubleshooting

### Hooks Not Running

#### 1. Check Hooks Are Executable (Linux/Mac)

```bash
chmod +x .claude/hooks/log-user-prompts.sh
chmod +x .claude/hooks/log-assistant-work.sh
```

#### 2. Install Dependencies (Linux/macOS)

```bash
# For log-assistant-work.sh - requires jq for JSON parsing
sudo apt install jq           # Ubuntu/Debian
brew install jq               # macOS
```

#### 3. Check Claude Code Settings

- Verify hooks are configured in `.claude/settings.local.json`
- Check that hook file paths are correct
- Ensure `stop_hook_active` is not `false` (indicates disabled hook)

#### 4. Test Manually

**Linux/macOS:**
```bash
# Test work log hook with mock transcript
echo '{"session_id":"test","transcript_path":"path/to/transcript.jsonl","hook_event_name":"Stop"}' | .claude/hooks/log-assistant-work.sh

# Test prompt log hook
echo '{"prompt":"Test prompt","session_id":"test"}' | .claude/hooks/log-user-prompts.sh
```

**Windows PowerShell:**
```powershell
# Test work log hook
'{"session_id":"test","transcript_path":"C:\\path\\to\\transcript.jsonl","hook_event_name":"Stop"}' | .\.claude\hooks\log-assistant-work.ps1

# Test prompt log hook
'{"prompt":"Test prompt","session_id":"test"}' | .\.claude\hooks\log-user-prompts.ps1
```

### Log File Issues

#### PROMPTLOG.md Not Created

- Check write permissions in project root
- Ensure hook script is executable (Linux/macOS)
- Verify hook is running: check for error messages in Claude Code console

#### PROMPTLOG.md Contains JSON Instead of Text

- Update to latest hook version (older versions didn't parse JSON)
- The current version properly extracts the `"prompt"` field

#### WORKLOG.md Shows "[No assistant messages found]"

- This means transcript file exists but contains no assistant messages yet
- Usually happens on first response or when hook runs before transcript is written
- Subsequent responses should work correctly

#### WORKLOG.md Shows "[jq not installed]" (Linux/macOS)

- Install jq: `sudo apt install jq` (Ubuntu/Debian) or `brew install jq` (macOS)
- Windows PowerShell works automatically (uses built-in JSON parser)

#### Hook Was Working But Stopped

**Check if Claude Code disabled the hook:**
- Look for `stop_hook_active: false` in settings
- Common causes: hook error, timeout, or infinite loop detection

**Solution:**
1. Fix any errors in hook script
2. Modify hook file timestamp:
   ```bash
   touch .claude/hooks/log-assistant-work.ps1
   ```
3. Or restart Claude Code session
4. Verify in `.claude/settings.local.json` that hook path is correct

### Encoding Issues

#### Vietnamese Text or Emoji Corrupted (Windows Only)

**Fixed in latest version** using StreamReader/StreamWriter with UTF-8.

**If you still see issues:**
1. Update `log-user-prompts.ps1` to use .NET StreamReader/StreamWriter
2. PowerShell's `$input` variable and `Write-Output` don't handle UTF-8 properly
3. Use `[System.Console]::OpenStandardInput()` with UTF-8 encoding instead

**Example fix:**
```powershell
# Read stdin with UTF-8
$stdin = [System.Console]::OpenStandardInput()
$reader = New-Object System.IO.StreamReader($stdin, [System.Text.Encoding]::UTF8)
$jsonInput = $reader.ReadToEnd()

# Write stdout with UTF-8
$stdout = [System.Console]::OpenStandardOutput()
$writer = New-Object System.IO.StreamWriter($stdout, [System.Text.Encoding]::UTF8)
$writer.WriteLine($jsonInput)
$writer.Flush()
```

#### General Encoding Issues

- Scripts use UTF-8 encoding
- Check your terminal encoding settings
- **Linux/macOS:** Bash handles UTF-8 natively (no special setup needed)
- **Windows:** Must use StreamReader/StreamWriter for proper UTF-8

### File Too Large

**Both files auto-archive when exceeding 1000 lines:**
- **WORKLOG.md:** Moved to `docs/claude_code/WORKLOG_YYYYMMDD_HHMMSS.md`
- **PROMPTLOG.md:** Moved to `docs/claude_code/PROMPTLOG_YYYYMMDD_HHMMSS.md`

**Manual archiving (if needed):**
```bash
# Archive with date
mv PROMPTLOG.md "PROMPTLOG_$(date +%Y%m%d).md"

# Compress archived logs
gzip PROMPTLOG_*.md
```

---

## Customization

### Change Log Format

Edit the hook scripts to customize output:

```bash
# In log-user-prompts.sh or log-user-prompts.ps1
# Modify this section:
cat >> "$LOG_FILE" << EOF
## 📝 Prompt - $TIMESTAMP

Your custom format here

EOF
```

### Change Timezone

#### Linux/Mac (log-user-prompts.sh)

```bash
# Change from GMT+7 to your timezone
TIMESTAMP=$(TZ='America/New_York' date '+%Y-%m-%d %H:%M:%S')
```

#### Windows (log-user-prompts.ps1)

```powershell
# Change from +7 hours to your timezone offset
$timestamp = (Get-Date).ToUniversalTime().AddHours(-5).ToString("yyyy-MM-dd HH:mm:ss")
```

### Add More Metadata

Extend hooks to log additional information:

```bash
BRANCH=$(git branch --show-current 2>/dev/null || echo "N/A")
CWD=$(pwd)

cat >> "$LOG_FILE" << EOF
## 📝 Prompt - $TIMESTAMP
**Branch:** $BRANCH
**Directory:** $CWD

$PROMPT

---
EOF
```

### Change Archive Threshold

Default: 1000 lines. To change:

```bash
# In the hook script, find:
if [ "$LINE_COUNT" -gt 1000 ]; then

# Change to your desired threshold:
if [ "$LINE_COUNT" -gt 5000 ]; then
```

### Custom Emoji

```bash
# In the hook script, change:
## 📝 Prompt - $TIMESTAMP

# To your preferred emoji:
## 💬 Prompt - $TIMESTAMP
## ❓ Question - $TIMESTAMP
## 🚀 Request - $TIMESTAMP
```

---

## FAQ

### Q: Do hooks work offline?

**A:** Yes! Hooks run locally and don't require internet connection.

### Q: Are logs visible to Claude/Anthropic?

**A:** No. Logs are stored locally on your machine and never sent to Anthropic servers.

### Q: Can I use hooks in CI/CD?

**A:** Yes, but you'll need to configure the CI environment to:
1. Install dependencies (`jq` for Linux)
2. Set up `.claude/settings.local.json`
3. Ensure proper permissions for hook scripts

### Q: How do I share hooks with my team?

**A:** Two options:
1. **Git submodule** (recommended): Add hooks as a submodule
2. **Direct commit**: Add hooks to your repository (ensure `.gitignore` excludes log files)

### Q: Can I use custom log file names?

**A:** Yes! Edit the `LOG_FILE` variable in hook scripts:

```bash
# Change from PROMPTLOG.md to custom name
LOG_FILE="my-custom-log.md"
```

### Q: Do hooks slow down Claude Code?

**A:** No. Hooks run asynchronously and have minimal performance impact (typically < 100ms).

### Q: Can I use hooks with multiple projects?

**A:** Yes! Each project has its own `.claude/hooks/` directory and log files.

### Q: How do I migrate old logs?

**A:** Simply copy old `PROMPTLOG.md` and `WORKLOG.md` files to the new project directory.

---

## Contributing

Contributions are welcome! Please follow these guidelines:

### Development Setup

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/my-feature`
3. Make your changes
4. Test on both Windows and Linux/macOS
5. Update documentation if needed
6. Submit a pull request

### Testing

**Test PowerShell hooks (Windows):**
```powershell
# Test prompt hook
'{"prompt":"Test prompt","session_id":"test"}' | .\.claude\hooks\log-user-prompts.ps1
cat PROMPTLOG.md

# Test work hook (requires valid transcript path)
'{"transcript_path":"C:\\path\\to\\transcript.jsonl"}' | .\.claude\hooks\log-assistant-work.ps1
cat WORKLOG.md
```

**Test Bash hooks (Linux/macOS):**
```bash
# Test prompt hook
echo '{"prompt":"Test prompt","session_id":"test"}' | .claude/hooks/log-user-prompts.sh
cat PROMPTLOG.md

# Test work hook (requires valid transcript path)
echo '{"transcript_path":"/path/to/transcript.jsonl"}' | .claude/hooks/log-assistant-work.sh
cat WORKLOG.md
```

### Code Style

- **PowerShell**: Follow [PowerShell Best Practices](https://docs.microsoft.com/en-us/powershell/scripting/developer/cmdlet/strongly-encouraged-development-guidelines)
- **Bash**: Follow [Google Shell Style Guide](https://google.github.io/styleguide/shellguide.html)
- **Documentation**: Use clear, concise English with code examples

### Reporting Issues

Please include:
- Operating system and version
- Claude Code version
- Hook script version
- Error messages (if any)
- Steps to reproduce

---

## License

MIT License

Copyright (c) 2025 [Your Name]

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

---

## Changelog

### v2.0.0 (2025-01-18)
- ✨ **New:** IDE context formatting with blank lines after tags
- ✨ **New:** Renamed hooks for clarity (`log-user-prompts`, `log-assistant-work`)
- 🐛 **Fixed:** UTF-8 encoding issues on Windows using StreamReader/StreamWriter
- 🐛 **Fixed:** Markdown structure validation for unbalanced code fences
- 📝 **Updated:** Comprehensive README with troubleshooting and examples
- 🔧 **Changed:** Removed blockquote markers for better copy-paste experience

### v1.0.0 (2025-01-14)
- 🎉 Initial release
- ✨ User prompt logging
- ✨ Assistant work logging
- ✨ Auto-archiving at 1000 lines
- ✨ Cross-platform support (Windows, Linux, macOS)

---

## Acknowledgments

- Built for [Claude Code](https://claude.ai/code) by Anthropic
- Inspired by the need for better development workflow tracking
- Thanks to the Claude Code community for feedback and testing

---

## Support

- 📖 **Documentation:** [https://github.com/your-username/claude-code-hooks](https://github.com/your-username/claude-code-hooks)
- 🐛 **Issues:** [https://github.com/your-username/claude-code-hooks/issues](https://github.com/your-username/claude-code-hooks/issues)
- 💬 **Discussions:** [https://github.com/your-username/claude-code-hooks/discussions](https://github.com/your-username/claude-code-hooks/discussions)

---

**Made with ❤️ for the Claude Code community**
