# Video Script: MCP Servers in Claude Code - Complete Setup Guide

## Video Title
"MCP Servers in Claude Code: Global vs Project-Specific Setup (Complete Tutorial)"

## Video Description
Learn how to install and configure MCP (Model Context Protocol) servers in Claude Code. This tutorial shows you how to extend Claude's capabilities with browser control, Firebase integration, and more using both global and project-specific configurations.

**What You'll Learn:**
- What MCP servers are and why they're powerful
- Installing Chrome DevTools MCP (browser control)
- Global vs project-specific configuration
- Testing and troubleshooting
- Real-world multi-project setup

**Timestamps:**
0:00 - Introduction
1:15 - What are MCP Servers?
3:00 - Why Project-Specific Configuration?
4:30 - Prerequisites Check
5:45 - Method 1: Global Setup (Chrome DevTools)
11:00 - Testing Your First MCP Server
13:15 - Method 2: Project-Specific Setup
19:00 - Real-World Multi-Project Example
22:30 - Troubleshooting Common Issues
26:00 - Conclusion & Next Steps

**Resources:**
- GitHub Repository: [Link in description]
- Complete README Guide: [Link]
- Chrome DevTools MCP: https://github.com/ChromeDevTools/chrome-devtools-mcp

---

## INTRO (0:00 - 1:15)

**[Screen: Channel intro animation]**

Hey everyone, Dimitar here! Today I'm going to show you how to supercharge Claude Code with MCP servers.

**[Screen: Title card - "MCP Servers: Complete Setup Guide"]**

By the end of this video, you'll know how to:
- Install MCP servers in minutes
- Control Chrome browser directly from Claude
- Set up project-specific configurations
- Avoid common setup mistakes

**[Screen: Demo - Claude controlling Chrome browser]**

Imagine asking Claude "check the performance of google.com" and watching it open Chrome, run tests, and give you a full analysis. That's what we're building today!

Let's get started!

---

## WHAT ARE MCP SERVERS? (1:15 - 3:00)

**[Screen: Simple MCP architecture diagram]**

MCP stands for Model Context Protocol. It's a way to connect Claude Code to external tools and services.

**[Screen: Animation showing Claude ↔ MCP Server ↔ Browser/Database/API]**

Without MCP, Claude is just a chat interface. WITH MCP servers, Claude can:
- Control Chrome browser
- Query databases
- Manage Firebase projects
- Monitor production errors with Sentry
- Test mobile apps

**[Screen: Live example]**

Watch this - I ask Claude: "Take a screenshot of https://github.com"

**[Screen: Claude opening Chrome, taking screenshot]**

Claude just opened Chrome, navigated to GitHub, and took a screenshot. All automatically!

**[Screen: Available MCP servers showcase]**

There are MCP servers for:
- Chrome DevTools (what we'll install today)
- Firebase
- Sentry
- Mobile testing
- And many more!

---

## WHY PROJECT-SPECIFIC? (3:00 - 4:30)

**[Screen: Diagram showing multiple projects]**

Here's the problem with putting everything global: imagine you're working on three projects:
- Your e-commerce site (production database)
- Your personal blog (test database)
- A client's app (client's Firebase)

**[Screen: Disaster scenario]**

With everything global, Claude has access to ALL of them at once. You're in your blog project and ask Claude to "delete test data" - oops! It might delete from your e-commerce database!

**[Screen: Project isolation diagram]**

Project-specific configuration fixes this. Each project only sees its own tools:
- E-commerce → Production database + Sentry
- Blog → Test database only
- Client app → Client's Firebase only

**[Screen: Benefits list]**

Benefits:
✓ No accidental cross-project operations
✓ Clean, organized environment
✓ Better security
✓ Perfect for teams

---

## PREREQUISITES CHECK (4:30 - 5:45)

**[Screen: Checklist]**

Quick check before we start:

**[Screen: ✓ Node.js check]**

1. Node.js installed (v20.19 or newer)

```bash
node --version
```

If you see v20 or higher, you're good!

**[Screen: ✓ Claude Code installed]**

2. Claude Code installed

```bash
claude --version
```

**[Screen: ✓ Chrome browser]**

3. Chrome browser (for our example)

**[Screen: ✓ Terminal open]**

4. Terminal/command line ready

**[Screen: Time estimate]**

Total setup time: About 15 minutes. Let's go!

---

## METHOD 1: GLOBAL SETUP (5:45 - 11:00)

**[Screen: Terminal window]**

We'll start with global setup - this makes the MCP server available in all your projects.

### Step 1: Choose Installation Method (5:45 - 7:00)

**[Screen: Two options displayed]**

You have two ways to install:

**Option A: Global Installation**
```bash
npm install -g chrome-devtools-mcp
```

**Option B: Using npx (Recommended)**

No installation needed! Just configure and go.

**[Screen: Comparison table]**

| Method | Pros | Cons |
|--------|------|------|
| Global Install | Slightly faster startup | Must update manually |
| npx | Auto-updates, no install | ~1 second slower first run |

**[Highlight npx]**

I recommend npx - it's simpler and always up-to-date.

### Step 2: Configure Claude Code (7:00 - 9:30)

**[Screen: Opening ~/.claude.json]**

Let's configure Claude Code. Open the config file:

```bash
code ~/.claude.json
```

**[Screen: JSON editor]**

Look for the `mcpServers` section at the root level. If it doesn't exist, add it:

**[Screen: Typing slowly with highlights]**

```json
{
  "mcpServers": {
```

**[Pause]**

This is where global MCP servers live.

```json
    "chrome-devtools": {
```

**[Pause]**

This is the name - you'll see it in `claude mcp list`.

```json
      "type": "stdio",
```

**[Pause]**

"stdio" means Claude talks to the server via standard input/output.

```json
      "command": "npx",
      "args": [
        "-y",
        "chrome-devtools-mcp@latest"
      ]
```

**[Pause - highlight @latest]**

`@latest` ensures you always get the newest version!

```json
    }
  }
}
```

**[Screen: Complete config highlighted]**

Here's the complete configuration:

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "type": "stdio",
      "command": "npx",
      "args": [
        "-y",
        "chrome-devtools-mcp@latest"
      ]
    }
  }
}
```

**[Screen: Save file animation]**

Save the file (Cmd+S or Ctrl+S).

### Step 3: Verify Installation (9:30 - 11:00)

**[Screen: Terminal]**

Let's check if it works:

```bash
claude mcp list
```

**[Screen: Loading animation]**

Wait a few seconds...

**[Screen: Success output]**

```
Checking MCP server health...

chrome-devtools: npx -y chrome-devtools-mcp@latest - ✓ Connected
```

**[Screen: Green checkmark zoom]**

Perfect! The green checkmark means it's working!

**[Screen: Troubleshooting note]**

If you see an error, don't worry - we'll cover troubleshooting at the end.

---

## TESTING YOUR FIRST MCP SERVER (11:00 - 13:15)

**[Screen: Opening Claude Code interface]**

Now let's actually use it! Open Claude Code.

### Test 1: Performance Check (11:00 - 11:45)

**[Screen: Typing in Claude]**

Type this prompt:

```
Check the performance of https://google.com
```

**[Screen: Claude opening Chrome browser]**

Watch! Claude is:
1. Opening Chrome browser
2. Navigating to Google
3. Recording performance metrics
4. Analyzing the results

**[Screen: Performance results]**

And here are the results! Claude analyzed:
- Page load time
- Largest Contentful Paint (LCP)
- First Input Delay (FID)
- Cumulative Layout Shift (CLS)

This is insane! Claude can now test website performance automatically!

### Test 2: Screenshot (11:45 - 12:30)

**[Screen: New prompt]**

Let's try another one:

```
Take a screenshot of https://github.com
```

**[Screen: Chrome opening]**

Claude opens Chrome, navigates to GitHub...

**[Screen: Screenshot displayed]**

And there's our screenshot! Embedded right in the chat!

### Test 3: Debug Console Errors (12:30 - 13:15)

**[Screen: Another prompt]**

One more:

```
Check https://example.com for any console errors or warnings
```

**[Screen: Claude checking console]**

Claude opens the page, checks the browser console, and reports back:

"No errors found! The page loaded cleanly."

**[Screen: Celebration]**

This is the power of MCP servers! Claude can now be your browser automation assistant!

---

## METHOD 2: PROJECT-SPECIFIC SETUP (13:15 - 19:00)

**[Screen: Multiple project folders]**

Now let's level up with project-specific configuration.

### When to Use Project-Specific (13:15 - 14:30)

**[Screen: Decision tree]**

**Use GLOBAL for:**
- Chrome DevTools (useful everywhere)
- General dev tools
- Personal productivity

**Use PROJECT-SPECIFIC for:**
- Database connections (different per project)
- Firebase projects (different per project)
- Sentry (different organizations)
- Client-specific APIs

**[Screen: Example scenario]**

Imagine you have:
- Project A: E-commerce (needs Firebase + Sentry)
- Project B: Blog (needs just Chrome DevTools)
- Project C: Mobile app (needs Mobile MCP + Sentry)

Project-specific config keeps them isolated!

### Step 1: Get Project Path (14:30 - 15:15)

**[Screen: Terminal]**

First, get your exact project path:

```bash
cd /path/to/your/project
pwd
```

**[Screen: Output showing path]**

```
/Users/username/work/ecommerce
```

**[Highlight and copy]**

Copy this EXACT path - we'll need it!

### Step 2: Configure Project (15:15 - 17:30)

**[Screen: Back to ~/.claude.json]**

Open `~/.claude.json` again. This time, find or create the `projects` section:

**[Screen: JSON structure]**

```json
{
  "mcpServers": {
    "chrome-devtools": { /* global config */ }
  },
  "projects": {
```

**[Pause]**

This is NEW - the `projects` section.

```json
    "/Users/username/work/ecommerce": {
```

**[Pause - highlight]**

**CRITICAL:** This path must EXACTLY match your `pwd` output!

```json
      "mcpServers": {
        "firebase": {
          "type": "stdio",
          "command": "npx",
          "args": [
            "-y",
            "firebase-tools@latest",
            "experimental:mcp"
          ]
        },
        "sentry": {
          "type": "http",
          "url": "https://mcp.sentry.dev/mcp"
        }
      }
    }
  }
}
```

**[Screen: Highlight firebase and sentry]**

Notice:
- Firebase: Only for this e-commerce project
- Sentry: Also project-specific
- Chrome DevTools: Still global (available everywhere)

### Step 3: Add Multiple Projects (17:30 - 19:00)

**[Screen: Adding blog project]**

Let's add your blog project:

```json
{
  "projects": {
    "/Users/username/work/ecommerce": {
      "mcpServers": {
        "firebase": { /* ... */ },
        "sentry": { /* ... */ }
      }
    },
    "/Users/username/personal/blog": {
      "mcpServers": {
        "chrome-devtools": {
          "type": "stdio",
          "command": "npx",
          "args": [
            "-y",
            "chrome-devtools-mcp@latest",
            "--headless=true"
          ]
        }
      }
    }
  }
}
```

**[Screen: Highlight --headless=true]**

Notice the blog project has Chrome DevTools with `--headless=true` - it runs without showing the browser window!

Each project can have different settings!

---

## REAL-WORLD EXAMPLE (19:00 - 22:30)

**[Screen: Three terminal windows side by side]**

Let's prove this works! I'll test all three locations.

### Test 1: Outside Any Project (19:00 - 19:45)

**[Left screen: Terminal]**

```bash
cd ~
claude mcp list
```

**[Output]**

```
chrome-devtools: ✓ Connected
```

Only global servers show!

### Test 2: In E-commerce Project (19:45 - 20:30)

**[Middle screen: Terminal]**

```bash
cd /Users/username/work/ecommerce
claude mcp list
```

**[Output]**

```
chrome-devtools: ✓ Connected (global)
firebase: ✓ Connected
sentry: ✓ Connected
```

E-commerce project sees all three!

### Test 3: In Blog Project (20:30 - 21:15)

**[Right screen: Terminal]**

```bash
cd /Users/username/personal/blog
claude mcp list
```

**[Output]**

```
chrome-devtools: ✓ Connected (headless mode)
```

Blog only sees Chrome DevTools with headless mode!

### Live Demo (21:15 - 22:30)

**[Screen: Claude Code in e-commerce project]**

I'm in the e-commerce project now. Let me ask Claude:

"Show me all Firebase projects"

**[Screen: Claude listing Firebase projects]**

It works! Claude can see my Firebase!

**[Screen: Switching to blog project]**

Now I switch to my blog project:

"Show me all Firebase projects"

**[Screen: Claude's response]**

"I don't have access to Firebase tools in this project."

Perfect isolation! No accidents possible!

---

## TROUBLESHOOTING (22:30 - 26:00)

**[Screen: Common errors list]**

Let's quickly cover the most common issues.

### Issue 1: MCP Server Not Found (22:30 - 23:15)

**[Screen: Error message]**

If `claude mcp list` doesn't show your server:

**[Screen: Solutions checklist]**

1. **Check JSON syntax**
   - Use jsonlint.com
   - Look for missing commas/brackets

2. **Restart Claude Code**
   - Close completely
   - Reopen
   - Wait 10 seconds

3. **Verify Node.js version**
   ```bash
   node --version  # Must be v20.19+
   ```

### Issue 2: Connection Failed (23:15 - 24:00)

**[Screen: Red X icon]**

If you see `✗ Connection failed`:

**[Screen: Solutions]**

1. **Clear npm cache**
   ```bash
   npm cache clean --force
   ```

2. **Test manually**
   ```bash
   npx -y chrome-devtools-mcp@latest
   ```

3. **Check Chrome is installed**
   - Visit google.com/chrome
   - Install if missing

### Issue 3: Wrong Project Path (24:00 - 25:00)

**[Screen: Path mismatch diagram]**

Most common mistake! Your path doesn't match:

**[Screen: Side-by-side comparison]**

**❌ Wrong:**
```json
{
  "projects": {
    "/Users/username/work/project": {  // Missing /ecommerce!
```

**✓ Correct:**
```bash
pwd
# /Users/username/work/ecommerce/project

# Use this EXACT path in JSON:
```
```json
{
  "projects": {
    "/Users/username/work/ecommerce/project": {
```

### Issue 4: Chrome Not Opening (25:00 - 26:00)

**[Screen: Headless mode option]**

If Chrome won't open, use headless mode:

```json
{
  "args": [
    "-y",
    "chrome-devtools-mcp@latest",
    "--headless=true"
  ]
}
```

Chrome runs in background - no window!

---

## CONCLUSION (26:00 - 28:00)

**[Screen: Summary slide]**

Amazing! Let's recap what we covered:

**[Screen: Checklist appearing]**

✓ What MCP servers are
✓ Installing Chrome DevTools MCP
✓ Global configuration
✓ Project-specific setup
✓ Testing and isolation
✓ Troubleshooting

**[Screen: Benefits recap]**

The power you now have:
- Claude controls Chrome browser
- Automated performance testing
- Screenshot capture
- Console debugging
- Perfect project isolation

**[Screen: Next steps card]**

**Your Action Plan:**

1. Add Chrome DevTools globally (takes 2 minutes)
2. Test with "Check performance of https://google.com"
3. Add project-specific servers as needed
4. Share your results in the comments!

**[Screen: More MCP servers showcase]**

**Explore More MCP Servers:**
- Firebase MCP (backend services)
- Sentry MCP (error monitoring)
- Mobile MCP (iOS/Android testing)
- Database MCP servers

Links in the description!

**[Screen: Subscribe animation]**

If this helped you:
- 👍 Hit that like button
- 🔔 Subscribe for more Claude Code tutorials
- 💬 Drop questions in comments

**[Screen: GitHub repo card]**

All code and documentation on GitHub - link below!

**[Screen: Personal message]**

I read every comment and answer questions. If you get stuck, describe your issue and I'll help you debug!

**[Screen: End screen with video suggestions]**

Thanks for watching! Check out these related videos on Claude Code tips and automation workflows.

See you in the next one!

---

## VIDEO METADATA

### Title Options
1. "MCP Servers in Claude Code: Complete Setup Guide (Global vs Project-Specific)"
2. "Install MCP Servers in Claude Code - Chrome DevTools Tutorial"
3. "Claude Code MCP: Browser Control & Project Isolation Tutorial"

### Description
```
Learn how to install and configure MCP (Model Context Protocol) servers in Claude Code! This complete tutorial shows you how to extend Claude's capabilities with Chrome DevTools MCP using both global and project-specific configurations.

⏰ TIMESTAMPS:
0:00 - Introduction
1:15 - What are MCP Servers?
3:00 - Why Project-Specific Configuration?
4:30 - Prerequisites
5:45 - Global Setup with Chrome DevTools
11:00 - Testing Your First MCP Server
13:15 - Project-Specific Configuration
19:00 - Real-World Multi-Project Example
22:30 - Troubleshooting
26:00 - Conclusion

🔗 RESOURCES:
- GitHub Repository: [Your Link]
- Complete README Guide: [Your Link]
- Chrome DevTools MCP: https://github.com/ChromeDevTools/chrome-devtools-mcp
- MCP Package Directory: https://www.npmjs.com/search?q=keywords:mcp-server

📋 QUICK START:
1. Install Node.js v20.19+
2. Add to ~/.claude.json:
{
  "mcpServers": {
    "chrome-devtools": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "chrome-devtools-mcp@latest"]
    }
  }
}
3. Run: claude mcp list
4. Test: "Check performance of https://google.com"

🏷️ TAGS:
#ClaudeCode #MCP #ChromeDevTools #AI #Programming #DevTools #Automation #WebDev #Tutorial #Anthropic

💬 Questions? Drop them in the comments!
👍 LIKE if this helped you!
🔔 SUBSCRIBE for more developer tutorials!
```

### Tags
#ClaudeCode #MCP #ChromeDevTools #Programming #AI #Development #Automation #WebDevelopment #Tutorial #CodingTutorial #DeveloperTools #Anthropic #BrowserAutomation #PerformanceTesting #ProjectManagement

### Thumbnail Design
**Text on Thumbnail:**
- "MCP SERVERS"
- "SETUP GUIDE"
- "15 MIN"

**Visual Elements:**
- Claude Code logo
- Chrome logo
- Checkmarks for "Global" and "Project-Specific"
- Terminal window mockup
- Your face/branding in corner

### Thumbnail Colors
- Background: Dark blue/purple gradient
- Text: White/Yellow (high contrast)
- Accent: Green (for checkmarks)

---

## FILMING NOTES

### Screen Recording
- Use 1920x1080 resolution minimum
- High contrast terminal theme (Dracula recommended)
- Font size 16pt minimum for code
- Zoom to 150% for JSON editing
- Slow down typing for complex commands

### B-Roll Ideas
1. Terminal commands being typed
2. JSON configuration with syntax highlighting
3. Chrome browser opening automatically
4. Performance metrics displaying
5. Multiple terminal windows for project comparison
6. Before/after comparisons
7. Success checkmarks appearing

### Audio
- Clear microphone (lapel or shotgun)
- Minimize background noise
- Speak clearly and slowly
- Pause between major sections
- Emphasize important warnings

### Editing
- Add zoom effects for important config lines
- Use arrows/highlights for JSON sections
- Add transition animations between methods
- Include progress bars for wait times
- Picture-in-picture for side-by-side comparisons
- Add sound effects for success/error moments

### Key Moments to Emphasize
1. The "@latest" benefit (auto-updates)
2. Exact path matching for project-specific
3. Testing isolation (show 3 terminals)
4. Troubleshooting section (slow down)
5. Quick start checklist (make it downloadable)
