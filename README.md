# Claude Code Hooks

This directory contains custom hooks for Claude Code that enhance the development workflow.

## Available Hooks

### 📝 Log User Prompts
**Files:** `log-user-prompts.sh`, `log-user-prompts.ps1`

**Trigger:** `UserPromptSubmit` - Runs when you submit a prompt (before Claude processes it)

**Purpose:** Automatically logs all user prompts with timestamps to `PROMPTLOG.md` in the project root.

### 🤖 Log Assistant Work
**Files:** `log-assistant-work.sh`, `log-assistant-work.ps1`

**Trigger:** `Stop` - Runs when Claude Code completes a response (after processing)

**Purpose:** Automatically logs summaries of work completed by Claude to `WORKLOG.md` in the project root.

**Features:**
- ✅ Saves summary of work completed by Claude
- ✅ Includes timestamp (Vietnamese timezone GMT+7)
- ✅ Formats as markdown with emoji (🤖)
- ✅ Auto-creates log file if doesn't exist
- ✅ Auto-archives to `docs/claude_code/` when exceeds 1000 lines
- ✅ Appends new work summaries to existing log
- ✅ WORKLOG.md is gitignored (won't be committed)
- ✅ Extracts assistant messages from transcript file

**Log Format:**
```markdown
## 🤖 Work Completed - 2025-11-18 12:45:30

Summary of work done by Claude (extracted from last assistant message in transcript)

---
```

**How it works:**
1. Hook receives JSON with `transcript_path` field
2. Reads JSONL transcript file (`.claude/projects/{project}/{session_id}.jsonl`)
3. Extracts all assistant messages with `role: "assistant"`
4. Uses the **last assistant message** as work summary
5. Logs clean work summary to WORKLOG.md
6. Returns original JSON to Claude Code

**Implementation Details:**
- **Windows (log-assistant-work.ps1)**: Uses PowerShell with .NET `ReadAllLines()` for UTF-8 handling
- **Linux/macOS (log-assistant-work.sh)**: Uses `jq` for JSON parsing (requires installation)

**Usage:**
1. Hook runs automatically when Claude completes any response
2. Check `WORKLOG.md` in root directory to see work history
3. Log file is excluded from git (in .gitignore)

**Platform Support:**
- **Linux/Mac:** Uses `log-assistant-work.sh` (bash)
- **Windows:** Uses `log-assistant-work.ps1` (PowerShell)

---

**Features:**
- ✅ Saves every prompt you send to Claude
- ✅ Includes timestamp (Vietnamese timezone GMT+7)
- ✅ UTF-8 encoding with proper Vietnamese character support
- ✅ Clean formatting for easy reading and copy-pasting
- ✅ Markdown structure validation (balanced code fences)
- ✅ Auto-creates log file if doesn't exist
- ✅ Auto-archives to `docs/claude_code/` when exceeds 1000 lines
- ✅ Appends new prompts to existing log
- ✅ PROMPTLOG.md is gitignored (won't be committed)

**Log Format:**
```markdown
## 📝 Prompt - 2025-11-14 10:30:45

Your prompt text here (extracted from JSON)
Multi-line prompts preserve original formatting
Easy to read and copy without blockquote markers

---
```

**How it works:**
1. Hook receives JSON input: `{"prompt": "your text", "session_id": "...", ...}`
2. **Windows (log-user-prompts.ps1)**:
   - Reads stdin using StreamReader with UTF-8 encoding
   - Parses JSON to extract `"prompt"` field
   - Sanitizes markdown: escapes backticks only (no blockquote markers for readability)
   - Validates code fence balance (wraps in code block if unbalanced)
   - Writes to file using UTF-8 without BOM
   - Returns JSON using StreamWriter with UTF-8
3. **Linux/macOS (log-user-prompts.sh)**:
   - Uses `jq` for JSON parsing (UTF-8 handled natively by bash)
   - Sanitizes markdown: escapes backticks only (no blockquote markers for readability)
   - Validates code fence balance
   - Returns original JSON

**Usage:**
1. Hook runs automatically when you send any prompt to Claude
2. Check `PROMPTLOG.md` in root directory to see all your prompts
3. Log file is excluded from git (in .gitignore)

**Platform Support:**
- **Linux/Mac:** Uses `log-user-prompts.sh` (bash)
- **Windows:** Uses `log-user-prompts.ps1` (PowerShell)

Claude Code automatically selects the appropriate version based on your platform.

## Hook Configuration

### Enable/Disable Hooks

Both hooks are **already configured** in `.claude/settings.local.json`:

**Windows Configuration:**
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

**Linux/macOS Configuration:**
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

**To disable hooks:**
1. Edit `.claude/settings.local.json`
2. Remove or comment out specific hook sections

**Notes:**
- Windows: Uses `-ExecutionPolicy Bypass` to avoid PowerShell execution policy issues
- Linux/macOS: Ensure scripts are executable (`chmod +x .claude/hooks/*.sh`)
- Linux/macOS: Install `jq` for stop.sh to work (`sudo apt install jq` or `brew install jq`)

### Hook Execution Order

```
User sends prompt
    ↓
UserPromptSubmit hook runs → Logs to PROMPTLOG.md
    ↓
Claude processes prompt
    ↓
Claude generates response
    ↓
Stop hook runs → Logs work summary to WORKLOG.md
    ↓
Response shown to user
```

## Benefits

### Prompt Logging (PROMPTLOG.md)
✅ **Complete History** - Never lose track of what you asked
✅ **Debug Aid** - Review past prompts when debugging
✅ **Learning** - Analyze which prompts work best
✅ **Pattern Analysis** - Identify successful prompt patterns

### Work Logging (WORKLOG.md)
✅ **Work Summary** - Automatic record of Claude's work
✅ **Development History** - Track all changes and implementations
✅ **Collaboration** - Share work summaries with team
✅ **Documentation** - Auto-generated development logs

### General Benefits
✅ **Privacy** - Both logs stored locally, not committed to git
✅ **Timestamped** - Vietnamese timezone (GMT+7) for accurate tracking
✅ **Markdown Format** - Easy to read and search

## File Locations

```
.claude/
├── hooks/
│   ├── README.md                    # This file
│   ├── log-user-prompts.sh          # Prompt log hook (Linux/Mac)
│   ├── log-user-prompts.ps1         # Prompt log hook (Windows)
│   ├── log-assistant-work.sh        # Work log hook (Linux/Mac)
│   └── log-assistant-work.ps1       # Work log hook (Windows)
│
PROMPTLOG.md                          # User prompts log (gitignored)
WORKLOG.md                            # Work summary log (gitignored)
```

## Troubleshooting

### Hooks Not Running

1. **Check hooks are executable (Linux/Mac):**
   ```bash
   chmod +x .claude/hooks/log-user-prompts.sh
   chmod +x .claude/hooks/log-assistant-work.sh
   ```

2. **Install dependencies (Linux/macOS only):**
   ```bash
   # For log-assistant-work.sh - requires jq for JSON parsing
   sudo apt install jq           # Ubuntu/Debian
   brew install jq               # macOS
   ```

3. **Check Claude Code settings:**
   - Verify hooks are configured in `.claude/settings.local.json`
   - Check that hook file paths are correct
   - Ensure `stop_hook_active` is not `false` (indicates disabled hook)

4. **Test manually:**
   ```bash
   # Linux/macOS - Test with mock transcript
   echo '{"session_id":"test","transcript_path":"path/to/transcript.jsonl","hook_event_name":"Stop"}' | .claude/hooks/log-assistant-work.sh

   # Windows PowerShell
   '{"session_id":"test","transcript_path":"C:\\path\\to\\transcript.jsonl","hook_event_name":"Stop"}' | .\.claude\hooks\log-assistant-work.ps1
   ```

### Log File Issues

1. **PROMPTLOG.md not created:**
   - Check write permissions in project root
   - Ensure hook script is executable

2. **PROMPTLOG.md contains JSON instead of prompt text:**
   - This was fixed in latest version (parses JSON to extract `"prompt"` field)
   - Update hook scripts to latest version from this README

3. **WORKLOG.md shows "[No assistant messages found in transcript]":**
   - This means transcript file exists but contains no assistant messages yet
   - Usually happens on first response or when hook runs before transcript is written
   - Subsequent responses should work correctly

4. **WORKLOG.md shows "[jq not installed]" (Linux/macOS only):**
   - Install jq: `sudo apt install jq` (Ubuntu/Debian) or `brew install jq` (macOS)
   - Windows PowerShell works automatically (uses built-in JSON parser)

5. **Hook was working but stopped:**
   - Check if Claude Code disabled the hook (`stop_hook_active: false`)
   - Common causes: hook error, timeout, or infinite loop detection
   - Solution: Modify hook file (touch timestamp) or restart Claude Code session
   - Verify in `.claude/settings.local.json` that hook path is correct

6. **Vietnamese text or emoji corrupted in PROMPTLOG.md (Windows only):**
   - **Fixed in latest version** using StreamReader/StreamWriter with UTF-8
   - **Solution**: Update `log-user-prompts.ps1` to use .NET StreamReader/StreamWriter
   - PowerShell's `$input` variable and `Write-Output` don't handle UTF-8 properly
   - Use `[System.Console]::OpenStandardInput()` with UTF-8 encoding instead
   - Example fix:
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

7. **Encoding issues (general):**
   - Scripts use UTF-8 encoding
   - Check your terminal encoding settings
   - Linux/macOS: Bash handles UTF-8 natively (no special setup needed)
   - Windows: Must use StreamReader/StreamWriter for proper UTF-8

8. **File too large:**
   - **Both files auto-archive when exceeding 1000 lines**
   - **WORKLOG.md**: Moved to `docs/claude_code/WORKLOG_YYYYMMDD_HHMMSS.md`
   - **PROMPTLOG.md**: Moved to `docs/claude_code/PROMPTLOG_YYYYMMDD_HHMMSS.md`
   - Manual archiving (if needed):
   ```bash
   # Archive with date
   mv PROMPTLOG.md "PROMPTLOG_$(date +%Y%m%d).md"

   # Compress archived logs
   gzip PROMPTLOG_*.md
   ```

## Implementation Details

### UTF-8 Encoding Handling

**Windows PowerShell (user-prompt-submit.ps1):**

PowerShell's built-in `$input` variable and `Write-Output` cmdlet don't handle UTF-8 properly for Vietnamese text and emojis. The fix uses .NET StreamReader/StreamWriter:

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

**Linux/macOS Bash (user-prompt-submit.sh):**

Bash handles UTF-8 natively on modern systems. No special encoding setup needed:

```bash
# Read stdin (UTF-8 handled automatically)
INPUT=$(cat)

# Write to file (UTF-8 preserved)
cat >> "$LOG_FILE" << EOF
Content here with Vietnamese: tiếng Việt
EOF

# Echo back (UTF-8 preserved)
echo "$INPUT"
```

### Markdown Sanitization

Both scripts sanitize prompts for proper markdown rendering while keeping text easy to read and copy:

1. **Escape backticks**: Prevents code block conflicts (`\`` → `\\``)
2. **Clean formatting**: No blockquote markers for better readability and copy-ability
3. **Code fence validation**: Counts `` ``` `` occurrences
4. **Unbalanced fix**: Wraps entire prompt in code block if count is odd

**PowerShell Example:**
```powershell
# Keep original formatting, only escape backticks
$sanitizedPrompt = $prompt -replace '`', '``'

# Validate code fence balance
$backtickCount = ([regex]::Matches($prompt, '```')).Count
if ($backtickCount % 2 -ne 0) {
    # Wrap in code block if unbalanced
    $sanitizedPrompt = "``````text`n$prompt`n``````"
}
```

**Bash Example:**
```bash
# Keep original formatting, only escape backticks
SANITIZED_PROMPT=$(echo "$PROMPT" | sed 's/`/\\`/g')

# Validate code fence balance
BACKTICK_COUNT=$(echo "$PROMPT" | grep -o '```' | wc -l)
if [ $((BACKTICK_COUNT % 2)) -ne 0 ]; then
    # Wrap in code block if unbalanced
    SANITIZED_PROMPT="\`\`\`text\n$PROMPT\n\`\`\`"
fi
```

## Customization

### Change Log Format

Edit the hook scripts to customize the log format:

```bash
# In log-user-prompts.sh or log-user-prompts.ps1
# Modify this section:
cat >> "$LOG_FILE" << EOF
## 📝 Prompt - $TIMESTAMP

Your custom format here

EOF
```

### Change Timezone

**Linux/Mac (log-user-prompts.sh):**
```bash
# Change from GMT+7 to your timezone
TIMESTAMP=$(TZ='America/New_York' date '+%Y-%m-%d %H:%M:%S')
```

**Windows (log-user-prompts.ps1):**
```powershell
# Change from +7 hours to your timezone offset
$timestamp = (Get-Date).ToUniversalTime().AddHours(-5).ToString("yyyy-MM-dd HH:mm:ss")
```

### Add More Metadata

You can extend the hook to log additional information:
- Git branch name
- Current working directory
- Environment variables
- Project context

Example:
```bash
BRANCH=$(git branch --show-current 2>/dev/null || echo "N/A")
CWD=$(pwd)

cat >> "$LOG_FILE" << EOF
## 📝 Prompt - $TIMESTAMP
**Branch:** $BRANCH
**Directory:** $CWD

\`\`\`
$PROMPT
\`\`\`
EOF
```

## Best Practices

1. ✅ **Review Logs Periodically** - Archive old logs to keep file size manageable
2. ✅ **Don't Commit PROMPTLOG.md** - Already in .gitignore
3. ✅ **Backup Important Prompts** - Copy useful prompts to documentation
4. ✅ **Clean Up Regularly** - Delete logs after archiving/reviewing
5. ✅ **Use for Learning** - Analyze which prompt patterns work best

## Debugging

### Check JSON Structure (For Developers)

If WORKLOG.md shows error messages, check the debug file:

```bash
# View debug file (created when JSON parsing fails)
cat WORKLOG_DEBUG.txt

# Pretty-print JSON structure (requires jq)
cat WORKLOG_DEBUG.txt | jq '.'

# Check specific fields
cat WORKLOG_DEBUG.txt | jq '.messages[]? | select(.role == "assistant")'
cat WORKLOG_DEBUG.txt | jq '.content'
```

### Manual Testing

Test hooks manually to debug issues:

```bash
# Test user-prompt-submit hook (bash)
echo '{"prompt":"Test prompt","session_id":"test123"}' | .claude/hooks/user-prompt-submit.sh

# Test stop hook (bash)
echo '{"messages":[{"role":"assistant","content":[{"type":"text","text":"Test work done"}]}]}' | .claude/hooks/stop.sh

# Check logs
cat PROMPTLOG.md
cat WORKLOG.md
```

```powershell
# Test user-prompt-submit hook (PowerShell)
'{"prompt":"Test prompt","session_id":"test123"}' | .\.claude\hooks\user-prompt-submit.ps1

# Test stop hook (PowerShell)
'{"messages":[{"role":"assistant","content":[{"type":"text","text":"Test work done"}]}]}' | .\.claude\hooks\stop.ps1

# Check logs
cat PROMPTLOG.md
cat WORKLOG.md
```

## Advanced Usage

### Filter Logs by Date

```bash
# Show today's prompts only
grep -A 5 "$(date '+%Y-%m-%d')" PROMPTLOG.md

# Show prompts from specific date
grep -A 5 "2025-11-14" PROMPTLOG.md
```

### Count Total Prompts

```bash
# Count number of prompts
grep -c "^## 📝 Prompt" PROMPTLOG.md
```

### Extract Only Prompt Text

```bash
# Extract all prompts without timestamps
awk '/^```$/,/^```$/ {if (!/^```$/) print}' PROMPTLOG.md
```

## Security & Privacy

⚠️ **Important:**
- PROMPTLOG.md is **gitignored** by default
- Logs contain ALL your prompts - may include sensitive information
- Keep PROMPTLOG.md local, don't share publicly
- Review before sharing project with others

## Support

For issues or questions about Claude Code hooks:
- Check Claude Code documentation
- Review hook script comments
- Test hooks manually to debug

---

**Created:** 2025-11-14
**Status:** Active
**Maintained by:** Project Team
