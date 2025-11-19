# MCP Server Setup Guide - Global vs Project-Specific Configuration

Complete guide for installing and configuring MCP (Model Context Protocol) servers in Claude Code. Learn how to set up global and project-specific MCP servers for enhanced AI-powered development.

---

## What are MCP Servers?

MCP (Model Context Protocol) servers extend Claude Code's capabilities by connecting it to external services and tools. Instead of just being a chat interface, Claude can now directly interact with browsers, databases, APIs, and other development tools in real-time.

**Examples of MCP Servers:**
- **Chrome DevTools MCP** - Control and debug Chrome browser
- **Firebase MCP** - Manage Firebase projects
- **Database MCP** - Query databases directly
- **GitHub MCP** - Interact with GitHub repositories

---

## Why Project-Specific Configuration?

**Security Benefits:**
- Each project only accesses its designated tools/services
- Prevents accidental cross-project operations
- Isolates credentials per project
- Reduces attack surface

**Organization Benefits:**
- Clean global configuration
- No scrolling through irrelevant servers
- Context-aware development environment
- Easier team collaboration

---

## Prerequisites

### Required Software
- **Claude Code** installed and running
- **Node.js** and npm (for installing MCP packages)
- **Terminal/Command line** access

---

## Installation Method 1: Global Configuration

Global installation makes the MCP server available in all projects. Best for tools you use everywhere.

### Step 1: Install MCP Package

We'll use Chrome DevTools MCP as an example - a simple, powerful MCP server that lets Claude control Chrome browser:

```bash
npm install -g chrome-devtools-mcp
```

**What Chrome DevTools MCP does:**
- Opens and controls Chrome browser
- Takes screenshots
- Records performance traces
- Debugs web applications
- Analyzes network requests

### Step 2: Find Installation Path

```bash
npm list -g chrome-devtools-mcp
```

Example output:
```
/Users/username/.nvm/versions/node/v22.12.0/lib
└── chrome-devtools-mcp@0.8.1
```

The full path to the script will be:
```
/Users/username/.nvm/versions/node/v22.12.0/lib/node_modules/chrome-devtools-mcp/build/src/index.js
```

### Step 3: Configure Claude Code (Global)

Edit `~/.claude.json` and add to the global `mcpServers` section:

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "type": "stdio",
      "command": "node",
      "args": [
        "/Users/username/.nvm/versions/node/v22.12.0/lib/node_modules/chrome-devtools-mcp/build/src/index.js"
      ]
    }
  }
}
```

**Note:** Replace the path with your actual installation path from Step 2.

### Alternative: Using npx (Simpler)

Instead of installing globally, you can use `npx` which automatically downloads and runs the latest version:

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

**Benefits of npx:**
- ✅ No global installation needed
- ✅ Always uses latest version
- ✅ Simpler configuration
- ✅ Works across different machines

### Step 4: Verify Installation

```bash
claude mcp list
```

Expected output:
```
Checking MCP server health...

chrome-devtools: npx -y chrome-devtools-mcp@latest - ✓ Connected
```

### Step 5: Test It!

Ask Claude:
```
Check the performance of https://google.com
```

Claude should open Chrome browser, load the page, and provide a performance analysis!

---

## Installation Method 2: Project-Specific Configuration

Project-specific installation isolates MCP servers to individual project directories. Recommended for project-specific tools and production environments.

### When to Use Project-Specific?

**Use Global For:**
- Chrome DevTools (useful everywhere)
- General development tools
- Personal productivity tools

**Use Project-Specific For:**
- Database connections (different DB per project)
- Firebase projects (different Firebase per project)
- Sentry monitoring (different Sentry org per project)
- Project-specific APIs

### Step 1: Configure Project-Specific Server

Edit `~/.claude.json` and add to the `projects` section:

```json
{
  "projects": {
    "/Users/username/path/to/your-project": {
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

**Important:** The project path must be an absolute path and must exactly match your project directory.

**Configuration Options:**
- `--headless=true` - Run Chrome without GUI (project-specific setting)
- `--isolated=true` - Use temporary profile (cleans up after)
- `--channel=canary` - Use Chrome Canary instead of stable

### Step 2: Get Your Exact Project Path

```bash
cd /your/project/directory
pwd
```

Copy the output exactly and use it as the key in your `projects` configuration.

### Step 3: Configure Multiple Projects

Example with different settings per project:

```json
{
  "projects": {
    "/Users/username/work/ecommerce-app": {
      "mcpServers": {
        "chrome-devtools": {
          "type": "stdio",
          "command": "npx",
          "args": [
            "-y",
            "chrome-devtools-mcp@latest",
            "--headless=false"
          ]
        },
        "firebase": {
          "type": "stdio",
          "command": "npx",
          "args": [
            "-y",
            "firebase-tools@latest",
            "experimental:mcp"
          ]
        }
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
            "--headless=true",
            "--isolated=true"
          ]
        }
      }
    }
  }
}
```

### Step 4: Test Project Isolation

```bash
# Test outside any project
cd ~
claude mcp list
# Should show global servers only (if any)

# Test inside ecommerce project
cd /Users/username/work/ecommerce-app
claude mcp list
# Should show: chrome-devtools, firebase

# Test inside blog project
cd /Users/username/personal/blog
claude mcp list
# Should show: chrome-devtools (with different settings)
```

---

## Popular MCP Servers

### 1. Chrome DevTools MCP (Browser Control)

```json
{
  "chrome-devtools": {
    "type": "stdio",
    "command": "npx",
    "args": ["-y", "chrome-devtools-mcp@latest"]
  }
}
```

**Use Cases:**
- Performance testing
- Screenshot automation
- Web debugging
- Network analysis

### 2. Firebase MCP (Backend Services)

```json
{
  "firebase": {
    "type": "stdio",
    "command": "npx",
    "args": [
      "-y",
      "firebase-tools@latest",
      "experimental:mcp"
    ]
  }
}
```

**Use Cases:**
- Manage Firestore data
- Deploy functions
- Configure authentication
- Monitor analytics

### 3. Sentry MCP (Error Monitoring)

```json
{
  "sentry": {
    "type": "http",
    "url": "https://mcp.sentry.dev/mcp"
  }
}
```

**Use Cases:**
- Analyze production errors
- Review crash reports
- Monitor performance issues
- Track user feedback

### 4. Mobile MCP (iOS/Android Testing)

```json
{
  "mobile-mcp": {
    "type": "stdio",
    "command": "npx",
    "args": [
      "-y",
      "@mobilenext/mobile-mcp@latest"
    ]
  }
}
```

**Use Cases:**
- Test iOS apps
- Test Android apps
- Automate mobile UI
- Capture mobile screenshots

---

## Real-World Example: Multiple Projects Setup

**Scenario:** You're working on:
1. An e-commerce web app (needs Chrome DevTools + Firebase)
2. A mobile app (needs Mobile MCP + Sentry)
3. A blog (needs Chrome DevTools only)

**Configuration:**

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "chrome-devtools-mcp@latest"]
    }
  },
  "projects": {
    "/Users/username/work/ecommerce": {
      "mcpServers": {
        "firebase": {
          "type": "stdio",
          "command": "npx",
          "args": [
            "-y",
            "firebase-tools@latest",
            "experimental:mcp",
            "--dir",
            "."
          ]
        }
      }
    },
    "/Users/username/work/mobile-app": {
      "mcpServers": {
        "mobile-mcp": {
          "type": "stdio",
          "command": "npx",
          "args": ["-y", "@mobilenext/mobile-mcp@latest"]
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

**Result:**
- Chrome DevTools available everywhere (global)
- Firebase only in e-commerce project
- Mobile MCP + Sentry only in mobile-app project
- Blog project just has Chrome DevTools

---

## Troubleshooting

### MCP Server Not Found

**Symptoms:**
- Server doesn't appear in `claude mcp list`
- No error messages

**Solutions:**

1. **Check if Claude Code is running:**
   ```bash
   claude --version
   ```

2. **Verify JSON syntax:**
   - Use a JSON validator (jsonlint.com)
   - Check for missing commas, brackets, quotes

3. **Restart Claude Code:**
   - Close Claude Code completely
   - Reopen it
   - Wait 10 seconds for MCP servers to initialize

4. **Test with npx manually:**
   ```bash
   npx -y chrome-devtools-mcp@latest
   ```

### Connection Failed

**Symptoms:**
```
chrome-devtools: npx ... - ✗ Connection failed
```

**Solutions:**

1. **Update Node.js:**
   ```bash
   node --version  # Should be v20.19+
   ```

2. **Clear npm cache:**
   ```bash
   npm cache clean --force
   ```

3. **Reinstall package:**
   ```bash
   npm uninstall -g chrome-devtools-mcp
   npm install -g chrome-devtools-mcp@latest
   ```

### Project-Specific Server Not Loading

**Symptoms:**
- Server shows globally but not in project directory
- Wrong server appears in project directory

**Solutions:**

1. **Verify exact path match:**
   ```bash
   cd /your/project
   pwd
   # Copy this EXACT output to ~/.claude.json
   ```

2. **Check you're in the right directory:**
   ```bash
   pwd
   claude mcp list
   ```

3. **Look for typos in JSON structure:**
   ```json
   {
     "projects": {
       "/exact/path/here": {  // Must match pwd output
         "mcpServers": {      // Note: "mcpServers" not "mcpServer"
   ```

### Chrome Browser Issues

**Symptoms:**
- Chrome doesn't open
- "Chrome not found" error

**Solutions:**

1. **Install Chrome:**
   - Download from google.com/chrome
   - Make sure it's in Applications (Mac) or Program Files (Windows)

2. **Specify Chrome path:**
   ```json
   {
     "chrome-devtools": {
       "type": "stdio",
       "command": "npx",
       "args": [
         "-y",
         "chrome-devtools-mcp@latest",
         "--executablePath=/Applications/Google Chrome.app/Contents/MacOS/Google Chrome"
       ]
     }
   }
   ```

3. **Use headless mode (no GUI):**
   ```json
   {
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
   ```

---

## Configuration Reference

### Chrome DevTools MCP Options

| Option | Description | Example |
|--------|-------------|---------|
| `--headless` | Run without GUI | `--headless=true` |
| `--isolated` | Temporary profile | `--isolated=true` |
| `--channel` | Chrome version | `--channel=canary` |
| `--executablePath` | Custom Chrome path | `--executablePath=/path/to/chrome` |
| `--viewport` | Browser window size | `--viewport=1920x1080` |

### MCP Server Types

**stdio (Most Common):**
```json
{
  "type": "stdio",
  "command": "npx",
  "args": ["-y", "package@latest"]
}
```

**http (Web Services):**
```json
{
  "type": "http",
  "url": "https://api.example.com/mcp"
}
```

---

## Best Practices

### 1. Use npx for Simplicity

**Good:**
```json
{
  "chrome-devtools": {
    "command": "npx",
    "args": ["-y", "chrome-devtools-mcp@latest"]
  }
}
```

**Avoid:**
```json
{
  "chrome-devtools": {
    "command": "node",
    "args": ["/long/absolute/path/that/breaks/on/other/machines"]
  }
}
```

### 2. Use @latest for Auto-Updates

```json
"args": ["-y", "chrome-devtools-mcp@latest"]
```

This ensures you always get the latest features and fixes.

### 3. Keep Global Config Minimal

Only add globally what you use in every project:
- Chrome DevTools ✅
- General dev tools ✅
- Project-specific databases ❌
- Project-specific APIs ❌

### 4. Document Your Configuration

Add comments (if your editor supports JSON with comments):

```jsonc
{
  "projects": {
    "/Users/username/work/ecommerce": {
      // E-commerce needs Firebase for backend
      "mcpServers": {
        "firebase": { /* config */ }
      }
    }
  }
}
```

---

## Quick Start Checklist

- [ ] Install Node.js (v20.19+)
- [ ] Install Claude Code
- [ ] Open `~/.claude.json`
- [ ] Add chrome-devtools to global `mcpServers`
- [ ] Run `claude mcp list` to verify
- [ ] Test with: "Check the performance of https://google.com"
- [ ] (Optional) Add project-specific servers
- [ ] (Optional) Test project isolation

---

## Additional Resources

- [Chrome DevTools MCP Documentation](https://github.com/ChromeDevTools/chrome-devtools-mcp)
- [MCP Package Directory](https://www.npmjs.com/search?q=keywords:mcp-server)
- [Claude Code MCP Documentation](https://docs.anthropic.com/en/docs/claude-code/mcp)
- [Model Context Protocol Specification](https://modelcontextprotocol.io/)

---

## Support

For issues or questions:
- Chrome DevTools MCP: [GitHub Issues](https://github.com/ChromeDevTools/chrome-devtools-mcp/issues)
- Claude Code: [Documentation](https://docs.anthropic.com/en/docs/claude-code)
- MCP Protocol: [Specification](https://modelcontextprotocol.io/)
