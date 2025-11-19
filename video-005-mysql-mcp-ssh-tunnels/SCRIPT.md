# MySQL MCP Server with SSH Tunnels - YouTube Video Script

**Video Length:** ~35 minutes
**Difficulty:** Intermediate
**Prerequisites:** Basic terminal knowledge, SSH access to a remote server

---

## Video Metadata

**Title:** How to Connect Claude Code to MySQL Databases with SSH Tunnels - Complete MCP Server Setup

**Description:**
Learn how to set up MySQL MCP servers in Claude Code with secure SSH tunnel connections. This tutorial shows you how to enable Claude to directly query, update, and manage your MySQL databases through encrypted connections. Perfect for developers working with remote databases who want AI-powered database operations.

**Topics Covered:**
- What is MySQL MCP and why use it
- SSH tunnels for secure database connections
- Centralized tunnel management system
- Global vs project-specific MCP configuration
- Real-world multi-project setup
- Testing and troubleshooting

**Tags:**
#ClaudeCode #MySQL #SSH #Database #AI #MCP #Development #Tutorial #DatabaseManagement #Tunneling

---

## Equipment & Setup Notes

- **Screen Recording:** Full screen with terminal prominently visible
- **Resolution:** 1920x1080 or higher
- **Font Size:** Terminal at least 16pt for readability
- **Color Scheme:** Dark theme recommended for better contrast
- **Cursor:** Make sure cursor is visible and highlighted
- **Pace:** Slow and deliberate, especially during configuration editing

---

## Script Structure

### [0:00 - 1:30] Introduction

**[SCREEN: Title slide or editor]**

"Hey everyone! In this tutorial, I'm going to show you how to connect Claude Code to your MySQL databases using MCP servers and SSH tunnels.

By the end of this video, you'll be able to ask Claude things like:
- 'Show me all users who registered in the last 30 days'
- 'Update the price of this product'
- 'Create a new table for storing reviews'

And Claude will directly execute these queries on your actual database - securely and through an encrypted connection.

This is incredibly powerful for database administration, data analysis, and rapid development. Let's dive in!"

---

### [1:30 - 4:00] What is MySQL MCP?

**[SCREEN: Show Claude Code interface]**

"First, let me explain what MySQL MCP is and why you'd want to use it.

MCP stands for Model Context Protocol - it's a way to extend Claude Code's capabilities by connecting it to external services. In this case, we're connecting it to MySQL databases.

**[SCREEN: Diagram or simple animation showing the flow]**

Here's what happens:
1. You ask Claude a question about your database in natural language
2. Claude uses the MySQL MCP server to translate that into SQL
3. The query is sent through an SSH tunnel to your remote database
4. Results come back and Claude presents them to you

**[SCREEN: Back to terminal]**

The SSH tunnel part is crucial because it means:
- All your database traffic is encrypted
- You don't need to expose MySQL to the internet
- You use your existing SSH authentication
- It's much more secure than opening MySQL ports publicly

Think of it as giving Claude secure, controlled access to your databases."

---

### [4:00 - 6:30] Prerequisites and Overview

**[SCREEN: Show checklist or text editor with requirements]**

"Before we start, here's what you need:

**Software:**
- Claude Code installed and running
- Node.js version 20.19 or higher
- SSH access to your remote server where MySQL is hosted
- MySQL client is optional but helpful for testing

**Access Requirements:**
- SSH credentials for your remote server
- MySQL database credentials: username, password, and database name
- Permission to establish SSH connections

**[SCREEN: Show architecture diagram if available]**

Here's our strategy: We're going to create a centralized tunnel management system. This means:
- One script manages SSH tunnels for all your projects
- Tunnels start automatically when Claude launches
- Each project gets its own port to avoid conflicts
- Easy to add new projects later

We'll set this up in two parts:
1. First, the SSH tunnel management
2. Second, the MySQL MCP server configuration

Let's get started!"

---

### [6:30 - 11:00] Part 1: SSH Tunnel Setup

**[SCREEN: Terminal - show current directory]**

"Alright, let's create our SSH tunnel management system.

First, create the directory for our tunnel scripts:"

```bash
mkdir -p ~/Dropbox/others/mcp-tunnels
cd ~/Dropbox/others/mcp-tunnels
```

**[TYPE IT OUT ON SCREEN]**

"Now let's create the tunnel management script. I'll create a file called manage_ssh_tunnels.sh:"

```bash
nano manage_ssh_tunnels.sh
```

**[SCREEN: Show nano editor or your preferred editor]**

"Here's the script we'll use. I'll walk you through the key sections:"

**[PASTE and EXPLAIN each section:]**

```bash
#!/bin/bash

# SSH Tunnel Manager for MCP MySQL Servers
```

"This is a bash script that will manage all our SSH tunnels.

First, we define our SSH server configuration:"

```bash
SSH_USER="your_ssh_username"
SSH_HOST="your.server.com"
SSH_PORT="22"
REMOTE_MYSQL_PORT="3306"
```

"You'll need to replace these with your actual values:
- SSH_USER: Your SSH username
- SSH_HOST: Your server hostname or IP address
- SSH_PORT: Usually 22, but could be 1022 or 2222 if your host uses custom ports
- REMOTE_MYSQL_PORT: Usually 3306, which is MySQL's default port

Next, we define our projects:"

```bash
PROJECT1_CONFIG="3307:myapp_production"
PROJECT2_CONFIG="3308:analytics_db"
```

"This is the format: local_port:database_name

Each project needs a unique local port. I'm starting at 3307 because 3306 is usually taken by local MySQL. You can add as many projects as you need - just increment the port number.

**[CONTINUE showing the function sections]**

The script has several functions:
- check_tunnel: Verifies if a tunnel is running
- start_tunnel: Creates a new SSH tunnel
- stop_tunnel: Closes an existing tunnel
- status: Shows status of all tunnels
- start_all and stop_all: Batch operations

**[SAVE THE FILE]**

Let me save this file and make it executable:"

```bash
chmod +x manage_ssh_tunnels.sh
```

"Now let's test it:"

```bash
./manage_ssh_tunnels.sh start
```

**[SHOW OUTPUT]**

"You should see output like:
```
Starting all SSH tunnels...
Starting SSH tunnel for project1 on port 3307...
✓ Successfully started project1 tunnel
```

Let's check the status:"

```bash
./manage_ssh_tunnels.sh status
```

**[SHOW STATUS OUTPUT]**

"Great! Our tunnels are running. Now let's move on to configuring the MySQL MCP server."

---

### [11:00 - 15:00] Part 2: MySQL MCP Server Installation

**[SCREEN: Terminal]**

"Now we need to install the MySQL MCP package. This is the actual server that Claude will use to talk to MySQL.

Let's install it globally with npm:"

```bash
npm install -g @bulgariamitko/mcp-server-mysql-write
```

**[WAIT for installation to complete]**

"Once that's installed, we need to find where npm put it:"

```bash
npm list -g @bulgariamitko/mcp-server-mysql-write
```

**[SHOW OUTPUT]**

"You'll see something like:
```
/Users/username/.nvm/versions/node/v22.11.0/lib
└── @bulgariamitko/mcp-server-mysql-write@1.0.0
```

The full path to the script we need is this lib directory plus:
`/node_modules/@bulgariamitko/mcp-server-mysql-write/dist/index.js`

So in my case, the full path is:
```
/Users/username/.nvm/versions/node/v22.11.0/lib/node_modules/@bulgariamitko/mcp-server-mysql-write/dist/index.js
```

Copy your path - we'll need it in the next step."

---

### [15:00 - 21:00] Part 3: Configure Claude Code (Global)

**[SCREEN: Open ~/.claude.json in editor]**

"Now let's configure Claude Code to use our MySQL MCP server.

I'm going to open the Claude configuration file:"

```bash
nano ~/.claude.json
```

**[SHOW THE FILE]**

"If this is your first time setting up MCP servers, your file might be pretty minimal. That's okay.

We're going to add a new section called 'mcpServers'. This is for global configuration - meaning this database will be available in all your projects.

Here's what to add:"

```json
{
  "mcpServers": {
    "myapp-mysql": {
      "type": "stdio",
      "command": "node",
      "args": [
        "/Users/username/.nvm/versions/node/v22.11.0/lib/node_modules/@bulgariamitko/mcp-server-mysql-write/dist/index.js"
      ],
      "env": {
        "MYSQL_HOST": "127.0.0.1",
        "MYSQL_PORT": "3307",
        "MYSQL_USER": "your_db_username",
        "MYSQL_PASS": "your_db_password",
        "MYSQL_DB": "your_database_name",
        "ALLOW_INSERT_OPERATION": "true",
        "ALLOW_UPDATE_OPERATION": "true",
        "ALLOW_DELETE_OPERATION": "true",
        "ALLOW_DDL_OPERATION": "true",
        "MYSQL_DISABLE_READ_ONLY_TRANSACTIONS": "true"
      }
    }
  }
}
```

**[EXPLAIN each field as you type/paste]**

"Let me explain each part:

**myapp-mysql** - This is the name of your MCP server. You'll see this in Claude. Make it descriptive.

**type: stdio** - This tells Claude to communicate with the server via standard input/output.

**command: node** - We're running the server with Node.js.

**args** - This array contains the path to the MCP server script. Use the path you copied earlier.

Now for the environment variables:

**MYSQL_HOST: 127.0.0.1** - Always use 127.0.0.1 when using SSH tunnels. This is localhost.

**MYSQL_PORT: 3307** - This matches the local port we set up in our tunnel script.

**MYSQL_USER, MYSQL_PASS, MYSQL_DB** - Your actual database credentials. Replace these with your real values.

**The ALLOW_* flags** - These control what operations Claude can perform:
- ALLOW_INSERT_OPERATION: Can Claude add new records?
- ALLOW_UPDATE_OPERATION: Can Claude modify existing records?
- ALLOW_DELETE_OPERATION: Can Claude delete records?
- ALLOW_DDL_OPERATION: Can Claude create/alter/drop tables?

For development, I'm setting these all to true. For production databases, you might want to set DELETE and DDL to false for safety.

**MYSQL_DISABLE_READ_ONLY_TRANSACTIONS** - Set this to true to allow write operations.

**[SAVE THE FILE]**

Now save the file and let's verify the connection:"

```bash
claude mcp list
```

**[SHOW OUTPUT]**

"You should see your MySQL server listed with a green checkmark:
```
myapp-mysql: node /path/to/index.js - ✓ Connected
```

If you see that, congratulations! Claude can now access your database."

---

### [21:00 - 26:00] Part 4: Project-Specific Configuration

**[SCREEN: Back to ~/.claude.json]**

"Now, let me show you something really powerful - project-specific configuration.

With global configuration, the MySQL server is available everywhere. But what if you have multiple projects with different databases? You don't want to accidentally query the wrong database!

That's where project-specific configuration comes in.

Let me show you how to set this up:"

```json
{
  "projects": {
    "/Users/username/path/to/your-project": {
      "mcpServers": {
        "project-mysql": {
          "type": "stdio",
          "command": "node",
          "args": [
            "/Users/username/.nvm/versions/node/v22.11.0/lib/node_modules/@bulgariamitko/mcp-server-mysql-write/dist/index.js"
          ],
          "env": {
            "MYSQL_HOST": "127.0.0.1",
            "MYSQL_PORT": "3308",
            "MYSQL_USER": "project_user",
            "MYSQL_PASS": "project_password",
            "MYSQL_DB": "project_database",
            "ALLOW_INSERT_OPERATION": "true",
            "ALLOW_UPDATE_OPERATION": "true",
            "ALLOW_DELETE_OPERATION": "true",
            "ALLOW_DDL_OPERATION": "true",
            "MYSQL_DISABLE_READ_ONLY_TRANSACTIONS": "true"
          }
        }
      }
    }
  }
}
```

"The key difference here is the path. You need to use the EXACT path to your project directory.

Let me show you how to get it:"

```bash
cd /path/to/your/project
pwd
```

**[SHOW PWD OUTPUT]**

"Copy that exact path - don't retype it, copy it. Then paste it as the key in your projects section.

Also notice I'm using port 3308 here, which matches PROJECT2_CONFIG in our tunnel script.

**[SAVE THE FILE]**

Now let's test the isolation. First, outside the project directory:"

```bash
cd ~
claude mcp list
```

**[SHOW OUTPUT - should show global servers only]**

"See? project-mysql doesn't appear here.

Now let's go into the project directory:"

```bash
cd /path/to/your/project
claude mcp list
```

**[SHOW OUTPUT - should show project-specific server]**

"Now project-mysql appears! This isolation is really powerful for:
- Production databases - keeps them separate
- Client projects - prevents cross-contamination
- Multiple environments - dev, staging, production

Each project only sees its own databases."

---

### [26:00 - 29:00] Part 5: Real-World Usage Examples

**[SCREEN: Claude Code interface]**

"Alright, let's see this in action! I'm going to show you some real queries.

**[TYPE in Claude Code]**

'Show me all tables in this database'

**[WAIT for Claude response]**

**[SHOW Claude executing SQL and returning results]**

'Great! Claude executed SHOW TABLES and listed all our tables.

Let me try something more complex:

'Show me the top 10 users by number of orders'

**[WAIT for response]**

**[SHOW Query and results]**

'Claude wrote a JOIN query, executed it, and gave us the results. That's pretty amazing!

Now let's try an update operation:

'Update user ID 123 to mark their email as verified'

**[WAIT for response]**

**[SHOW update confirmation]**

'Claude executed the UPDATE statement and confirmed the change.

One more - let's try creating a table:

'Create a new table called product_reviews with columns for review_id, product_id, user_id, rating (1-5), review_text, and created_at timestamp'

**[WAIT for response]**

**[SHOW CREATE TABLE statement and confirmation]**

'Excellent! Claude created the table with proper constraints, foreign keys, and even added a CHECK constraint for the rating field.

This is the power of MySQL MCP - you can interact with your database in natural language, and Claude handles all the SQL details."

---

### [29:00 - 32:00] Part 6: Automatic Tunnel Management

**[SCREEN: Back to ~/.claude.json]**

"One more thing before we wrap up - let's make our tunnels start automatically when Claude launches.

We'll use Claude Code hooks for this. Open ~/.claude.json again:"

```bash
nano ~/.claude.json
```

"Add a hooks section at the top level:"

```json
{
  "hooks": {
    "onStartup": "~/Dropbox/others/mcp-tunnels/manage_ssh_tunnels.sh start",
    "onShutdown": "~/Dropbox/others/mcp-tunnels/manage_ssh_tunnels.sh stop"
  },
  "mcpServers": {
    ...
  }
}
```

"This tells Claude to:
- Run our start script when it launches
- Run our stop script when it exits

**[SAVE FILE]**

Now let's test it. I'll quit Claude Code and restart it:"

```bash
# Close Claude Code
# Reopen Claude Code
```

**[SHOW terminal with tunnel status]**

```bash
~/Dropbox/others/mcp-tunnels/manage_ssh_tunnels.sh status
```

**[SHOW OUTPUT showing tunnels running]**

"Perfect! The tunnels started automatically.

And when you close Claude Code:"

```bash
# Close Claude Code again
~/Dropbox/others/mcp-tunnels/manage_ssh_tunnels.sh status
```

**[SHOW OUTPUT showing tunnels stopped]**

"The tunnels automatically stopped. No manual management needed!"

---

### [32:00 - 34:30] Troubleshooting Common Issues

**[SCREEN: Split screen or document with common errors]**

"Before we finish, let me quickly cover some common issues you might run into:

**Issue 1: 'Connection refused' when starting tunnel**

This usually means:
- SSH credentials are wrong
- Server is not reachable
- Port is already in use

Solution: Test your SSH connection first:
```bash
ssh -p 22 your_user@your.server.com
```

If that works, check if the port is in use:
```bash
lsof -i :3307
```

**Issue 2: MCP server shows as 'Disconnected'**

Check these things:
- Is the tunnel running? Run status command
- Are credentials correct in ~/.claude.json?
- Can you connect with mysql client?

Test direct connection:
```bash
mysql -h 127.0.0.1 -P 3307 -u your_user -p
```

**Issue 3: 'Permission denied' when Claude tries to update**

Check your ALLOW_* flags in the configuration. Make sure they're set to 'true' and not 'false'.

**Issue 4: Project-specific server appears everywhere**

The path in your projects section must match EXACTLY. Use pwd to get the exact path, don't type it manually.

For more troubleshooting, check the README file I've created - it has a comprehensive troubleshooting section."

---

### [34:30 - 35:00] Conclusion

**[SCREEN: Back to Claude Code or title screen]**

"And that's it! You now have:
- Secure SSH tunnels to your MySQL databases
- MySQL MCP servers configured in Claude Code
- Global and project-specific database access
- Automatic tunnel management

You can now use Claude to:
- Query your databases in natural language
- Update records safely
- Analyze data and schemas
- Create and modify tables
- All through encrypted SSH connections

This is incredibly powerful for database administration, data analysis, and rapid development.

If you found this helpful, please like and subscribe! Drop any questions in the comments below.

The complete configuration files and detailed documentation are linked in the description.

Thanks for watching, and happy coding!"

---

## Filming Notes

### Terminal Commands Summary
Create a cheat sheet with all commands ready to copy:

```bash
# Setup
mkdir -p ~/Dropbox/others/mcp-tunnels
cd ~/Dropbox/others/mcp-tunnels
nano manage_ssh_tunnels.sh
chmod +x manage_ssh_tunnels.sh

# Test tunnels
./manage_ssh_tunnels.sh start
./manage_ssh_tunnels.sh status
./manage_ssh_tunnels.sh stop

# Install MCP
npm install -g @bulgariamitko/mcp-server-mysql-write
npm list -g @bulgariamitko/mcp-server-mysql-write

# Configure Claude
nano ~/.claude.json
claude mcp list

# Test database
mysql -h 127.0.0.1 -P 3307 -u user -p database

# Get project path
cd /path/to/project
pwd
```

### B-Roll Ideas
- Diagram animations showing SSH tunnel flow
- Close-ups of configuration file sections
- Screen recordings of Claude executing queries
- Terminal output with successful connections
- MySQL Workbench or similar showing database structure

### On-Screen Text Overlays
- "SSH Tunnel: Encrypted Connection"
- "Port 3307 → Remote MySQL"
- "MCP: Model Context Protocol"
- "Global vs Project-Specific"
- Key configuration values as they're typed

### Common Mistakes to Avoid During Recording
1. Don't show real credentials - use "your_username", "your_password"
2. Don't show real server hostnames - use "your.server.com"
3. Don't show real database names - use generic names like "myapp_production"
4. Make sure terminal font is large enough
5. Slow down when typing configuration - viewers need to follow along
6. Pause after commands to show output clearly
7. Test everything before recording - no failed commands in final video

### Post-Production Checklist
- [ ] Add chapter markers at major section breaks
- [ ] Add closed captions for accessibility
- [ ] Speed up long installation/download sequences
- [ ] Add background music at low volume
- [ ] Highlight important terminal output
- [ ] Add links to GitHub repo in description
- [ ] Add links to npm package
- [ ] Add timestamps in description

### Video Description Template

```
Learn how to set up MySQL MCP servers in Claude Code with secure SSH tunnels!

⏱️ TIMESTAMPS:
0:00 - Introduction
1:30 - What is MySQL MCP?
4:00 - Prerequisites
6:30 - SSH Tunnel Setup
11:00 - Install MySQL MCP
15:00 - Configure Claude Code (Global)
21:00 - Project-Specific Configuration
26:00 - Real-World Usage Examples
29:00 - Automatic Tunnel Management
32:00 - Troubleshooting
34:30 - Conclusion

📚 RESOURCES:
- Full documentation: [GitHub link]
- MySQL MCP Package: https://www.npmjs.com/package/@bulgariamitko/mcp-server-mysql-write
- Claude Code: https://claude.com/code
- MCP Protocol: https://modelcontextprotocol.io/

💻 COMMANDS USED:
All commands are in the GitHub repository linked above.

🔒 SECURITY NOTES:
- Never commit .claude.json to version control
- Use strong passwords for database access
- Limit database user permissions appropriately
- Regularly rotate credentials
- Monitor database logs for suspicious activity

❓ QUESTIONS?
Drop them in the comments! I respond to everyone.

🔔 Don't forget to LIKE and SUBSCRIBE for more Claude Code tutorials!

#ClaudeCode #MySQL #SSH #Database #Tutorial
```

---

## Alternative Takes / Extended Content

If you want to make this video longer or split into parts:

### Part A: SSH Tunnels Only (15 min)
- Focus entirely on SSH tunnel setup
- More depth on SSH configuration
- Multiple server examples
- Advanced SSH options

### Part B: MySQL MCP Configuration (20 min)
- Assumes tunnels are already working
- Focus on MCP configuration
- More usage examples
- Advanced queries and operations

### Extended Version: Complete Series (3 videos)
1. **Video 1:** SSH Tunnels for Database Access (15 min)
2. **Video 2:** MySQL MCP Server Setup (20 min)
3. **Video 3:** Advanced Usage & Production Tips (25 min)

---

## Engagement Prompts

Throughout the video, ask questions to increase engagement:

- "Have you ever wished you could just ask your database questions in plain English? That's what we're building today!"
- "Drop a comment if you're currently managing multiple database projects"
- "What database tasks do you find most tedious? Let me know!"
- "Are you using MCP servers for other services? Share in the comments!"

---

## Follow-Up Video Ideas

Mention these at the end to encourage subscription:

- "Next week: How to set up PostgreSQL MCP servers"
- "Coming soon: Advanced database querying with Claude"
- "Tutorial request: Let me know what MCP servers you want to see next"
