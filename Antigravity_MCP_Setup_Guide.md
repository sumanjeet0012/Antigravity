# Antigravity CLI MCP Setup Guide

## Overview

This document describes the MCP (Model Context Protocol) servers
currently installed and working in my Antigravity CLI setup.

Current working servers:

-   ✅ GitHub (Official)
-   ✅ Filesystem (Official)
-   ✅ Playwright (Official)
-   ✅ AWS API (AWS Labs)

------------------------------------------------------------------------

# What is an MCP Server?

An MCP server is a bridge between Antigravity CLI and an external
system.

    Antigravity CLI
            │
            ▼
        MCP Server
            │
            ▼
    External System

Examples:

-   GitHub MCP → GitHub API
-   Filesystem MCP → Local files
-   Playwright MCP → Browser automation
-   AWS MCP → AWS services

The servers are **not always running**. Antigravity starts them when
needed and stops them when the session ends.

------------------------------------------------------------------------

# Global MCP Configuration

Configuration file:

``` text
~/.gemini/config/mcp_config.json
```

Current configuration:

``` json
{
  "mcpServers": {
    "github": {
      "command": "github-mcp-server",
      "args": ["stdio"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "<YOUR_TOKEN>"
      }
    },
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/Users/sumanjeet/code",
        "/Users/sumanjeet/Documents"
      ]
    },
    "playwright": {
      "command": "npx",
      "args": [
        "-y",
        "@playwright/mcp@latest"
      ]
    },
    "aws": {
      "command": "uvx",
      "args": [
        "awslabs.aws-api-mcp-server"
      ]
    }
  }
}
```

------------------------------------------------------------------------

# 1. GitHub MCP (Official)

Purpose:

-   Read repositories
-   Create branches
-   Read/Create Issues
-   Review Pull Requests
-   GitHub Actions
-   Repository search

## Install Go

``` bash
brew install go
```

## Install GitHub MCP

``` bash
go install github.com/github/github-mcp-server/cmd/github-mcp-server@latest
```

## Verify

``` bash
which github-mcp-server
github-mcp-server --version
```

## Authenticate

``` bash
gh auth login
gh auth status
gh auth token
```

Add your token as:

``` text
GITHUB_PERSONAL_ACCESS_TOKEN
```

------------------------------------------------------------------------

# 2. Filesystem MCP

Purpose:

-   Read files
-   Write files
-   Search directories
-   Refactor projects

Requirements:

``` bash
brew install node
```

No installation is required.

Antigravity runs:

``` bash
npx -y @modelcontextprotocol/server-filesystem
```

------------------------------------------------------------------------

# 3. Playwright MCP

Purpose:

-   Browser automation
-   Screenshots
-   UI testing
-   Web scraping

Install browsers:

``` bash
npx playwright install
```

Antigravity runs:

``` bash
npx -y @playwright/mcp@latest
```

------------------------------------------------------------------------

# 4. AWS API MCP

Purpose:

-   EC2
-   S3
-   IAM
-   CloudWatch
-   Lambda
-   Other AWS APIs

Install:

``` bash
uv tool install awslabs.aws-api-mcp-server
```

or

``` bash
uvx awslabs.aws-api-mcp-server
```

Configure AWS credentials:

``` bash
aws configure
```

or

``` bash
aws sso login
```

------------------------------------------------------------------------

# Verify Installed MCP Servers

Launch Antigravity:

``` bash
agy
```

Open the MCP panel:

    /mcp

Expected status:

    ✓ github
    ✓ filesystem
    ✓ playwright
    ✓ aws

------------------------------------------------------------------------

# Useful Prompts

## GitHub

-   Review PR #123
-   List open issues
-   Create a branch
-   Show CI failures

## Filesystem

-   Find every TODO
-   Open all GitHub Actions workflows
-   Search for a symbol

## Playwright

-   Open localhost:3000
-   Click Login
-   Take a screenshot
-   Fill the signup form

## AWS

-   List EC2 instances
-   List S3 buckets
-   Show Lambda logs
-   Inspect CloudWatch

------------------------------------------------------------------------

# Notes

-   Git does not require an MCP server because Antigravity can invoke
    the local `git` CLI.
-   MCP servers are started by Antigravity on demand and are not
    permanent background services.
-   The GitHub MCP server requires `stdio` mode and a
    `GITHUB_PERSONAL_ACCESS_TOKEN`.
