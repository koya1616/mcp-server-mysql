# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Model Context Protocol (MCP) server** that provides MySQL database connectivity with comprehensive security controls. It acts as a secure bridge between MCP clients and MySQL databases, supporting both direct connections and SSH tunneling.

## Architecture

The server follows a layered architecture:

- **Main Server (`index.ts`)**: Core MCP server implementation using `@modelcontextprotocol/sdk`
- **Database Layer**: MySQL connection pooling with `mysql2` and transaction management  
- **Security Layer**: Permission-based access control with schema-specific restrictions
- **Transport Layer**: SSH tunneling support for secure remote connections
- **Utilities (`utils/index.ts`)**: Logging infrastructure with configurable output

## Key Features

- **Permission System**: Granular control over INSERT, UPDATE, DELETE, and DDL operations
- **Schema-Specific Permissions**: Different permission levels per database schema
- **SSH Tunneling**: Secure connections through SSH with private key authentication
- **SQL Query Parsing**: Uses `node-sql-parser` to analyze and validate queries before execution
- **Transaction Safety**: Read-only queries use read-only transactions, write operations use proper commit/rollback

## Common Commands

### Build and Development
```bash
# Build the TypeScript project
npm run build

# Run the compiled server
node dist/index.js

# Run with ts-node for development
npx ts-node index.ts
```

### Environment Configuration
The server requires environment variables for database and SSH configuration:

- `MYSQL_HOST`, `MYSQL_PORT`, `MYSQL_USER`, `MYSQL_PASS`, `MYSQL_DB`
- `SSH_HOST`, `SSH_USER`, `SSH_KEY` (for SSH tunneling)
- Permission flags: `ALLOW_INSERT_OPERATION`, `ALLOW_UPDATE_OPERATION`, etc.
- Schema-specific permissions: `SCHEMA_INSERT_PERMISSIONS`, `SCHEMA_UPDATE_PERMISSIONS`, etc.
- `ENABLE_LOGGING` for debug output

## Permission System Architecture

The server implements a two-tier permission system:
1. **Global defaults**: `ALLOW_*_OPERATION` environment variables
2. **Schema overrides**: `SCHEMA_*_PERMISSIONS` with format `"schema1:true,schema2:false"`

Permission checking occurs in `executeReadOnlyQuery()` at index.ts:476-538, with schema extraction at index.ts:112-133.

## Database Connection Management

- Connection pooling via `mysql2.createPool()` with lazy initialization
- SSH tunnel creation at index.ts:711-773 for secure remote access
- Transaction management separates read-only (index.ts:460-606) from write operations (index.ts:609-699)

## MCP Integration

The server exposes:
- **Tool**: `mysql_query` for executing SQL queries
- **Resources**: Database table schemas via information_schema queries
- **Capabilities**: Dynamic tool description based on enabled permissions

Schema resource handling occurs at index.ts:294-317, with table information exposed through MCP resource URIs.