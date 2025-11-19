# MySQL MCP Server Setup with SSH Tunnels

Complete guide for setting up MySQL MCP servers in Claude Code with secure SSH tunnel connections to remote databases. This enables Claude to directly query, update, and manage your MySQL databases through encrypted connections.

---

## What is MySQL MCP?

MySQL MCP (Model Context Protocol) server allows Claude Code to interact directly with MySQL databases. Instead of manually writing SQL queries, Claude can:

- **Query databases** - Fetch data, analyze tables, examine schemas
- **Update records** - Modify existing data with natural language commands
- **Insert data** - Add new records to tables
- **Execute DDL** - Create tables, modify schemas, manage indexes
- **Database analysis** - Understand relationships, identify issues, suggest optimizations

**Example interactions:**
- "Show me all users registered in the last 30 days"
- "Update the price of product ID 123 to $49.99"
- "Create a new table for storing customer reviews"
- "Find all orders with status 'pending' older than 7 days"

---

## Why SSH Tunnels?

SSH tunnels provide secure, encrypted connections to remote databases without exposing them to the internet.

**Security Benefits:**
- All database traffic is encrypted
- No need to open MySQL port (3306) to public internet
- Uses existing SSH authentication
- No additional firewall rules required
- Protects against man-in-the-middle attacks

**How it works:**
```
Claude Code → MySQL MCP → SSH Tunnel → Remote Server → MySQL
(Local)       (127.0.0.1:3307)  (Encrypted)  (gold.superhosting.bg)  (localhost:3306)
```

---

## Prerequisites

### Required Software

- **Claude Code** installed and running
- **Node.js** v20.19 or higher (for MCP server)
- **SSH access** to your remote server
- **MySQL client** (optional, for testing)

### Required Access

- SSH credentials for remote server
- MySQL database credentials (username, password, database name)
- Ability to establish SSH connections from your machine

---

## Architecture Overview

### Centralized Tunnel Management

This guide uses a centralized SSH tunnel management system that:

1. **Manages multiple projects** - Run tunnels for all your databases simultaneously
2. **Auto-starts with Claude Code** - Tunnels start when Claude launches
3. **Handles port conflicts** - Each project gets its own port
4. **Provides unified control** - Single script to start/stop/check all tunnels

**Directory structure:**
```
~/Dropbox/others/mcp-tunnels/
├── manage_ssh_tunnels.sh    # Main control script
└── CLAUDE.md                 # Documentation
```

---

## Installation Part 1: SSH Tunnel Setup

### Step 1: Create Tunnel Management Directory

```bash
mkdir -p ~/Dropbox/others/mcp-tunnels
cd ~/Dropbox/others/mcp-tunnels
```

### Step 2: Create Tunnel Management Script

Create `manage_ssh_tunnels.sh`:

```bash
#!/bin/bash

# SSH Tunnel Manager for MCP MySQL Servers

# Server configuration
SSH_USER="your_ssh_username"
SSH_HOST="your.server.com"
SSH_PORT="22"  # or 1022, 2222, etc.
REMOTE_MYSQL_PORT="3306"

# Project configurations (format: port:database_name)
PROJECT1_CONFIG="3307:myapp_production"
PROJECT2_CONFIG="3308:analytics_db"

# Colors for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

# Function to check if a tunnel is running
check_tunnel() {
    local port=$1
    local project=$2

    if pgrep -f "ssh.*-L ${port}:localhost:${REMOTE_MYSQL_PORT}.*${SSH_HOST}" > /dev/null; then
        echo -e "${GREEN}✓${NC} $project tunnel (port $port) is running"
        return 0
    else
        echo -e "${RED}✗${NC} $project tunnel (port $port) is not running"
        return 1
    fi
}

# Function to start a tunnel
start_tunnel() {
    local port=$1
    local project=$2

    echo "Starting SSH tunnel for $project on port $port..."
    ssh -f -N -L ${port}:localhost:${REMOTE_MYSQL_PORT} -p ${SSH_PORT} ${SSH_USER}@${SSH_HOST}

    sleep 2
    if check_tunnel $port $project > /dev/null; then
        echo -e "${GREEN}✓${NC} Successfully started $project tunnel"
    else
        echo -e "${RED}✗${NC} Failed to start $project tunnel"
    fi
}

# Function to stop a tunnel
stop_tunnel() {
    local port=$1
    local project=$2

    echo "Stopping SSH tunnel for $project on port $port..."
    pkill -f "ssh.*-L ${port}:localhost:${REMOTE_MYSQL_PORT}.*${SSH_HOST}"
    echo -e "${YELLOW}✓${NC} Stopped $project tunnel"
}

# Function to show status of all tunnels
status() {
    echo "SSH Tunnel Status:"
    echo "=================="

    IFS=':' read -r port db_name <<< "$PROJECT1_CONFIG"
    check_tunnel "$port" "project1"

    IFS=':' read -r port db_name <<< "$PROJECT2_CONFIG"
    check_tunnel "$port" "project2"
}

# Function to start all tunnels
start_all() {
    echo "Starting all SSH tunnels..."
    echo "=========================="

    IFS=':' read -r port db_name <<< "$PROJECT1_CONFIG"
    if ! check_tunnel "$port" "project1" > /dev/null; then
        start_tunnel "$port" "project1"
    else
        echo -e "${YELLOW}!${NC} project1 tunnel already running"
    fi

    IFS=':' read -r port db_name <<< "$PROJECT2_CONFIG"
    if ! check_tunnel "$port" "project2" > /dev/null; then
        start_tunnel "$port" "project2"
    else
        echo -e "${YELLOW}!${NC} project2 tunnel already running"
    fi
}

# Function to stop all tunnels
stop_all() {
    echo "Stopping all SSH tunnels..."
    echo "========================="

    IFS=':' read -r port db_name <<< "$PROJECT1_CONFIG"
    stop_tunnel "$port" "project1"

    IFS=':' read -r port db_name <<< "$PROJECT2_CONFIG"
    stop_tunnel "$port" "project2"
}

# Main script logic
case "$1" in
    "status")
        status
        ;;
    "start")
        start_all
        ;;
    "stop")
        stop_all
        ;;
    "restart")
        stop_all
        sleep 3
        start_all
        ;;
    *)
        echo "Usage: $0 {status|start|stop|restart}"
        exit 1
        ;;
esac
```

### Step 3: Make Script Executable

```bash
chmod +x ~/Dropbox/others/mcp-tunnels/manage_ssh_tunnels.sh
```

### Step 4: Configure Your Projects

Edit the script and update these sections:

**SSH Server Configuration:**
```bash
SSH_USER="your_ssh_username"     # Your SSH username
SSH_HOST="your.server.com"        # Your server hostname/IP
SSH_PORT="22"                     # SSH port (often 22, 1022, or 2222)
REMOTE_MYSQL_PORT="3306"          # MySQL port on remote server
```

**Project Configuration:**
```bash
# Add your projects here (format: local_port:database_name)
PROJECT1_CONFIG="3307:myapp_production"
PROJECT2_CONFIG="3308:analytics_db"
PROJECT3_CONFIG="3309:staging_db"
```

**Port Assignment Guidelines:**
- Start at 3307 (3306 is usually taken by local MySQL)
- Each project needs a unique port
- Increment by 1 for each new project

### Step 5: Test SSH Tunnel

```bash
# Start all tunnels
~/Dropbox/others/mcp-tunnels/manage_ssh_tunnels.sh start

# Check status
~/Dropbox/others/mcp-tunnels/manage_ssh_tunnels.sh status
```

Expected output:
```
SSH Tunnel Status:
==================
✓ project1 tunnel (port 3307) is running
✓ project2 tunnel (port 3308) is running
```

### Step 6: Test Database Connection (Optional)

```bash
mysql -h 127.0.0.1 -P 3307 -u your_db_user -p your_database
```

If you see the MySQL prompt, the tunnel is working correctly!

---

## Installation Part 2: MySQL MCP Server Setup

### Step 1: Install MySQL MCP Package

```bash
npm install -g @bulgariamitko/mcp-server-mysql-write
```

### Step 2: Find Installation Path

```bash
npm list -g @bulgariamitko/mcp-server-mysql-write
```

Example output:
```
/Users/username/.nvm/versions/node/v22.11.0/lib
└── @bulgariamitko/mcp-server-mysql-write@1.0.0
```

The full path will be:
```
/Users/username/.nvm/versions/node/v22.11.0/lib/node_modules/@bulgariamitko/mcp-server-mysql-write/dist/index.js
```

### Step 3: Configure Claude Code

You can configure MySQL MCP servers **globally** (available everywhere) or **project-specific** (available only in certain directories).

#### Option A: Global Configuration

Edit `~/.claude.json` and add to the global `mcpServers` section:

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

**When to use global:**
- Development database used across multiple projects
- Shared analytics database
- Central configuration/settings database

#### Option B: Project-Specific Configuration

Edit `~/.claude.json` and add to the `projects` section:

```json
{
  "projects": {
    "/Users/username/path/to/your-project": {
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
            "MYSQL_USER": "project_db_user",
            "MYSQL_PASS": "project_db_password",
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

**When to use project-specific:**
- Production databases (isolation for safety)
- Client-specific databases (prevent cross-contamination)
- Staging environments (separate from production)
- Multiple projects with different databases

**Benefits of project-specific:**
- ✅ Appears only when working in that directory
- ✅ Prevents accidental database operations on wrong database
- ✅ Keeps credentials isolated per project
- ✅ Cleaner global MCP server list

### Step 4: Get Your Exact Project Path

```bash
cd /path/to/your/project
pwd
```

Copy the exact output and use it as the key in your `projects` configuration.

### Step 5: Verify MCP Connection

```bash
claude mcp list
```

**For global configuration:**
You should see your MySQL server from any directory:
```
myapp-mysql: node /path/to/index.js - ✓ Connected
```

**For project-specific configuration:**
```bash
# Inside project directory
cd /Users/username/path/to/your-project
claude mcp list
# Should show: myapp-mysql: ... - ✓ Connected

# Outside project directory
cd ~
claude mcp list
# Should NOT show myapp-mysql
```

---

## Configuration Reference

### Environment Variables

| Variable | Description | Required | Example |
|----------|-------------|----------|---------|
| `MYSQL_HOST` | MySQL host (always 127.0.0.1 for tunnels) | Yes | `127.0.0.1` |
| `MYSQL_PORT` | Local tunnel port | Yes | `3307` |
| `MYSQL_USER` | Database username | Yes | `myapp_user` |
| `MYSQL_PASS` | Database password | Yes | `secure_password` |
| `MYSQL_DB` | Database name | Yes | `myapp_production` |
| `ALLOW_INSERT_OPERATION` | Enable INSERT queries | No | `true` / `false` |
| `ALLOW_UPDATE_OPERATION` | Enable UPDATE queries | No | `true` / `false` |
| `ALLOW_DELETE_OPERATION` | Enable DELETE queries | No | `true` / `false` |
| `ALLOW_DDL_OPERATION` | Enable CREATE/ALTER/DROP | No | `true` / `false` |
| `MYSQL_DISABLE_READ_ONLY_TRANSACTIONS` | Allow write operations | No | `true` / `false` |

### Permission Levels

**Read-Only (Safest):**
```json
"env": {
  "MYSQL_HOST": "127.0.0.1",
  "MYSQL_PORT": "3307",
  "MYSQL_USER": "readonly_user",
  "MYSQL_PASS": "password",
  "MYSQL_DB": "database",
  "ALLOW_INSERT_OPERATION": "false",
  "ALLOW_UPDATE_OPERATION": "false",
  "ALLOW_DELETE_OPERATION": "false",
  "ALLOW_DDL_OPERATION": "false"
}
```

**Full Access (Development):**
```json
"env": {
  "MYSQL_HOST": "127.0.0.1",
  "MYSQL_PORT": "3307",
  "MYSQL_USER": "admin_user",
  "MYSQL_PASS": "password",
  "MYSQL_DB": "database",
  "ALLOW_INSERT_OPERATION": "true",
  "ALLOW_UPDATE_OPERATION": "true",
  "ALLOW_DELETE_OPERATION": "true",
  "ALLOW_DDL_OPERATION": "true",
  "MYSQL_DISABLE_READ_ONLY_TRANSACTIONS": "true"
}
```

---

## Real-World Example: Multiple Projects

**Scenario:** You have three projects:
1. **E-commerce app** - Production database (project-specific)
2. **Analytics dashboard** - Shared analytics DB (global)
3. **Blog platform** - Staging database (project-specific)

### SSH Tunnel Configuration

Edit `~/Dropbox/others/mcp-tunnels/manage_ssh_tunnels.sh`:

```bash
# Project configurations
ECOMMERCE_CONFIG="3307:ecommerce_production"
ANALYTICS_CONFIG="3308:analytics_shared"
BLOG_CONFIG="3309:blog_staging"
```

### MCP Configuration

Edit `~/.claude.json`:

```json
{
  "mcpServers": {
    "analytics-mysql": {
      "type": "stdio",
      "command": "node",
      "args": ["/path/to/mcp-server-mysql-write/dist/index.js"],
      "env": {
        "MYSQL_HOST": "127.0.0.1",
        "MYSQL_PORT": "3308",
        "MYSQL_USER": "analytics_user",
        "MYSQL_PASS": "analytics_pass",
        "MYSQL_DB": "analytics_shared",
        "ALLOW_INSERT_OPERATION": "false",
        "ALLOW_UPDATE_OPERATION": "false",
        "ALLOW_DELETE_OPERATION": "false",
        "ALLOW_DDL_OPERATION": "false"
      }
    }
  },
  "projects": {
    "/Users/username/work/ecommerce-app": {
      "mcpServers": {
        "ecommerce-mysql": {
          "type": "stdio",
          "command": "node",
          "args": ["/path/to/mcp-server-mysql-write/dist/index.js"],
          "env": {
            "MYSQL_HOST": "127.0.0.1",
            "MYSQL_PORT": "3307",
            "MYSQL_USER": "ecommerce_admin",
            "MYSQL_PASS": "ecommerce_pass",
            "MYSQL_DB": "ecommerce_production",
            "ALLOW_INSERT_OPERATION": "true",
            "ALLOW_UPDATE_OPERATION": "true",
            "ALLOW_DELETE_OPERATION": "true",
            "ALLOW_DDL_OPERATION": "true",
            "MYSQL_DISABLE_READ_ONLY_TRANSACTIONS": "true"
          }
        }
      }
    },
    "/Users/username/work/blog-platform": {
      "mcpServers": {
        "blog-mysql": {
          "type": "stdio",
          "command": "node",
          "args": ["/path/to/mcp-server-mysql-write/dist/index.js"],
          "env": {
            "MYSQL_HOST": "127.0.0.1",
            "MYSQL_PORT": "3309",
            "MYSQL_USER": "blog_user",
            "MYSQL_PASS": "blog_pass",
            "MYSQL_DB": "blog_staging",
            "ALLOW_INSERT_OPERATION": "true",
            "ALLOW_UPDATE_OPERATION": "true",
            "ALLOW_DELETE_OPERATION": "false",
            "ALLOW_DDL_OPERATION": "true",
            "MYSQL_DISABLE_READ_ONLY_TRANSACTIONS": "true"
          }
        }
      }
    }
  }
}
```

**Result:**
- **Analytics DB** available everywhere (read-only)
- **E-commerce DB** only in ecommerce-app directory (full access)
- **Blog DB** only in blog-platform directory (no delete)

---

## Automatic Tunnel Management with Claude Code

### Setup Auto-Start on Claude Launch

Create a Claude Code hook to start tunnels automatically:

1. Edit `~/.claude.json`
2. Add to the hooks section:

```json
{
  "hooks": {
    "onStartup": "~/Dropbox/others/mcp-tunnels/manage_ssh_tunnels.sh start"
  }
}
```

### Setup Auto-Stop on Claude Exit

Add to hooks:

```json
{
  "hooks": {
    "onStartup": "~/Dropbox/others/mcp-tunnels/manage_ssh_tunnels.sh start",
    "onShutdown": "~/Dropbox/others/mcp-tunnels/manage_ssh_tunnels.sh stop"
  }
}
```

Now tunnels will automatically:
- ✅ Start when Claude Code launches
- ✅ Stop when Claude Code exits
- ✅ Remain running throughout your session

---

## Usage Examples

### Querying Data

```
You: Show me all users who registered in the last 30 days

Claude: [Uses MySQL MCP to execute]
SELECT * FROM users WHERE created_at >= DATE_SUB(NOW(), INTERVAL 30 DAY);

Results:
- user_id: 123, email: john@example.com, created_at: 2024-01-15
- user_id: 124, email: jane@example.com, created_at: 2024-01-18
...
```

### Updating Records

```
You: Update the price of product ID 456 to $79.99

Claude: [Uses MySQL MCP to execute]
UPDATE products SET price = 79.99 WHERE product_id = 456;

✓ Updated 1 row
```

### Database Analysis

```
You: What tables exist in this database and what are their relationships?

Claude: [Analyzes schema via MySQL MCP]
Found 8 tables:
- users (primary key: user_id)
- orders (foreign key: user_id → users)
- products (primary key: product_id)
- order_items (foreign keys: order_id → orders, product_id → products)
...
```

### Creating Tables

```
You: Create a table for storing customer reviews with ratings 1-5

Claude: [Uses MySQL MCP to execute DDL]
CREATE TABLE customer_reviews (
  review_id INT AUTO_INCREMENT PRIMARY KEY,
  user_id INT NOT NULL,
  product_id INT NOT NULL,
  rating TINYINT NOT NULL CHECK (rating BETWEEN 1 AND 5),
  review_text TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(user_id),
  FOREIGN KEY (product_id) REFERENCES products(product_id)
);

✓ Table created successfully
```

---

## Troubleshooting

### SSH Tunnel Issues

**Problem: "Connection refused" when starting tunnel**

Solutions:
1. Verify SSH credentials:
   ```bash
   ssh -p 1022 your_user@your.server.com
   ```

2. Check if port is already in use:
   ```bash
   lsof -i :3307
   ```

3. Try different port in configuration

**Problem: Tunnel shows as running but database connection fails**

Solutions:
1. Verify MySQL is running on remote server:
   ```bash
   ssh your_user@your.server.com "systemctl status mysql"
   ```

2. Check remote MySQL is listening on localhost:
   ```bash
   ssh your_user@your.server.com "netstat -tlnp | grep 3306"
   ```

3. Test tunnel manually:
   ```bash
   ssh -L 3307:localhost:3306 -p 1022 your_user@your.server.com
   # In another terminal:
   mysql -h 127.0.0.1 -P 3307 -u db_user -p
   ```

### MCP Server Issues

**Problem: MCP server shows as "Disconnected"**

Solutions:
1. Check Node.js version:
   ```bash
   node --version  # Should be v20.19+
   ```

2. Verify MCP package installation:
   ```bash
   npm list -g @bulgariamitko/mcp-server-mysql-write
   ```

3. Check database credentials in `~/.claude.json`

4. Test direct connection:
   ```bash
   mysql -h 127.0.0.1 -P 3307 -u your_user -p your_database
   ```

**Problem: "Permission denied" errors in Claude**

Solutions:
1. Check permission flags in `~/.claude.json`:
   ```json
   "ALLOW_INSERT_OPERATION": "true",
   "ALLOW_UPDATE_OPERATION": "true",
   "ALLOW_DELETE_OPERATION": "true",
   "ALLOW_DDL_OPERATION": "true",
   "MYSQL_DISABLE_READ_ONLY_TRANSACTIONS": "true"
   ```

2. Verify MySQL user has proper grants:
   ```sql
   SHOW GRANTS FOR 'your_user'@'localhost';
   ```

### Port Conflicts

**Problem: "Address already in use"**

Solutions:
1. Find process using the port:
   ```bash
   lsof -i :3307
   ```

2. Kill the process:
   ```bash
   kill -9 <PID>
   ```

3. Or use different port in configuration

### Project Isolation Issues

**Problem: MCP server appears in wrong project**

Solutions:
1. Verify exact path match:
   ```bash
   cd /your/project
   pwd  # Copy exact output to ~/.claude.json
   ```

2. Check `~/.claude.json` structure:
   ```json
   {
     "projects": {
       "/exact/path/from/pwd": {
         "mcpServers": {
           // Your config
         }
       }
     }
   }
   ```

3. Restart Claude Code after configuration changes

---

## Security Best Practices

### 1. Use SSH Key Authentication

Instead of password authentication, use SSH keys:

```bash
# Generate SSH key
ssh-keygen -t ed25519 -C "your_email@example.com"

# Copy to server
ssh-copy-id -p 1022 your_user@your.server.com

# Test connection
ssh -p 1022 your_user@your.server.com
```

### 2. Limit Database User Permissions

Create MySQL users with minimal required permissions:

```sql
-- Read-only user for analytics
CREATE USER 'analytics_ro'@'localhost' IDENTIFIED BY 'password';
GRANT SELECT ON analytics_db.* TO 'analytics_ro'@'localhost';

-- Limited write user for app
CREATE USER 'app_user'@'localhost' IDENTIFIED BY 'password';
GRANT SELECT, INSERT, UPDATE ON app_db.* TO 'app_user'@'localhost';
FLUSH PRIVILEGES;
```

### 3. Use Different Credentials per Environment

```json
{
  "projects": {
    "/path/to/production": {
      "mcpServers": {
        "prod-mysql": {
          "env": {
            "MYSQL_USER": "prod_readonly",
            "ALLOW_UPDATE_OPERATION": "false",
            "ALLOW_DELETE_OPERATION": "false"
          }
        }
      }
    },
    "/path/to/development": {
      "mcpServers": {
        "dev-mysql": {
          "env": {
            "MYSQL_USER": "dev_admin",
            "ALLOW_UPDATE_OPERATION": "true",
            "ALLOW_DELETE_OPERATION": "true"
          }
        }
      }
    }
  }
}
```

### 4. Never Commit Credentials

Add to `.gitignore`:
```
.claude.json
**/mcp-tunnels/**
*.sql
*.dump
```

### 5. Rotate Passwords Regularly

Update passwords in:
1. MySQL server
2. `~/.claude.json` configuration
3. Any backup scripts or documentation

### 6. Monitor Database Activity

Enable MySQL query logging:
```sql
SET GLOBAL general_log = 'ON';
SET GLOBAL general_log_file = '/var/log/mysql/query.log';
```

Review logs regularly for suspicious activity.

---

## Port Allocation Strategy

### Standard Port Assignment

| Port Range | Usage |
|------------|-------|
| 3306 | Local MySQL (avoid) |
| 3307-3320 | Production databases |
| 3321-3340 | Staging databases |
| 3341-3360 | Development databases |
| 3361-3380 | Testing databases |

### Port Documentation Template

Create `~/Dropbox/others/mcp-tunnels/PORTS.md`:

```markdown
# Port Allocation

| Port | Project | Environment | Database | User |
|------|---------|-------------|----------|------|
| 3307 | E-commerce | Production | ecommerce_prod | ecom_admin |
| 3308 | Analytics | Shared | analytics | analytics_ro |
| 3309 | Blog | Staging | blog_staging | blog_user |
| 3310 | Mobile App | Production | mobile_prod | mobile_admin |
```

---

## Performance Optimization

### Connection Pooling

The MySQL MCP server handles connection pooling automatically, but you can optimize:

1. **Keep tunnels running** - Starting/stopping adds latency
2. **Use persistent SSH connections** - Add to `~/.ssh/config`:
   ```
   Host gold.superhosting.bg
     ServerAliveInterval 60
     ServerAliveCountMax 3
     ControlMaster auto
     ControlPath ~/.ssh/control-%r@%h:%p
     ControlPersist 600
   ```

3. **Monitor tunnel health** - Run periodic checks:
   ```bash
   */5 * * * * ~/Dropbox/others/mcp-tunnels/manage_ssh_tunnels.sh status
   ```

### Query Optimization

When asking Claude to query databases:

**Good:**
```
Show me the top 10 products by sales in the last month
```

**Better:**
```
Show me the top 10 products by sales in the last month,
limiting to products in category 'Electronics'
```

This helps Claude write more efficient queries with proper WHERE clauses.

---

## Backup and Recovery

### Backup Tunnel Configuration

```bash
# Backup SSH tunnel configuration
cp ~/Dropbox/others/mcp-tunnels/manage_ssh_tunnels.sh \
   ~/Dropbox/others/mcp-tunnels/manage_ssh_tunnels.sh.backup

# Backup Claude MCP configuration
cp ~/.claude.json ~/.claude.json.backup
```

### Restore Configuration

```bash
# Restore from backup
cp ~/.claude.json.backup ~/.claude.json

# Restart Claude Code to apply changes
```

---

## Advanced Configuration

### Multiple SSH Servers

If you have databases on different servers:

```bash
# In manage_ssh_tunnels.sh
start_tunnel_server1() {
    ssh -f -N -L 3307:localhost:3306 -p 1022 user@server1.com
}

start_tunnel_server2() {
    ssh -f -N -L 3308:localhost:3306 -p 22 user@server2.com
}
```

### Custom SSH Options

Add SSH options to tunnel script:

```bash
ssh -f -N \
  -L ${port}:localhost:${REMOTE_MYSQL_PORT} \
  -p ${SSH_PORT} \
  -o ServerAliveInterval=60 \
  -o ServerAliveCountMax=3 \
  -o ExitOnForwardFailure=yes \
  -o StrictHostKeyChecking=no \
  ${SSH_USER}@${SSH_HOST}
```

### Environment-Specific Configurations

Use different Claude config files:

```bash
# Development
export CLAUDE_CONFIG=~/.claude.dev.json
claude

# Production
export CLAUDE_CONFIG=~/.claude.prod.json
claude
```

---

## Migration Guide

### From Direct MySQL Connection

If you're currently connecting directly to MySQL without tunnels:

**Before:**
```json
"env": {
  "MYSQL_HOST": "remote.server.com",
  "MYSQL_PORT": "3306",
  ...
}
```

**After:**
1. Set up SSH tunnel on port 3307
2. Update configuration:
   ```json
   "env": {
     "MYSQL_HOST": "127.0.0.1",
     "MYSQL_PORT": "3307",
     ...
   }
   ```

### From Other MCP Database Tools

If migrating from different MCP packages:

1. Uninstall old package:
   ```bash
   npm uninstall -g old-mcp-package
   ```

2. Install new package:
   ```bash
   npm install -g @bulgariamitko/mcp-server-mysql-write
   ```

3. Update `~/.claude.json` with new paths

---

## Quick Start Checklist

- [ ] Install Node.js v20.19+
- [ ] Install MySQL MCP package globally
- [ ] Create `~/Dropbox/others/mcp-tunnels` directory
- [ ] Create `manage_ssh_tunnels.sh` script
- [ ] Make script executable (`chmod +x`)
- [ ] Configure SSH credentials in script
- [ ] Add project configurations (ports + databases)
- [ ] Test SSH tunnel: `./manage_ssh_tunnels.sh start`
- [ ] Check tunnel status: `./manage_ssh_tunnels.sh status`
- [ ] Test database connection with MySQL client
- [ ] Find MCP package installation path
- [ ] Add MCP configuration to `~/.claude.json`
- [ ] Verify with `claude mcp list`
- [ ] (Optional) Add Claude Code hooks for auto-start
- [ ] Test with Claude: "Show me all tables in the database"

---

## Additional Resources

- [MySQL MCP Package on npm](https://www.npmjs.com/package/@bulgariamitko/mcp-server-mysql-write)
- [Model Context Protocol Documentation](https://modelcontextprotocol.io/)
- [Claude Code MCP Documentation](https://docs.anthropic.com/en/docs/claude-code/mcp)
- [SSH Tunneling Guide](https://www.ssh.com/academy/ssh/tunneling)
- [MySQL Documentation](https://dev.mysql.com/doc/)

---

## Support

For issues or questions:
- MySQL MCP Package: [GitHub Issues](https://github.com/bulgariamitko/mcp-server-mysql-write/issues)
- Claude Code: [Documentation](https://docs.anthropic.com/en/docs/claude-code)
- SSH Tunneling: Your server hosting provider support

---

## Changelog

### Version 1.0 (2025-01-20)
- Initial release
- Centralized tunnel management system
- Support for multiple projects
- Global and project-specific MCP configurations
- Automatic tunnel management with Claude Code hooks
