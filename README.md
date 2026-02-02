# postman-report

CLI tool to fetch Postman team user reports using the Postman CLI authentication.

## Prerequisites

- [Postman CLI](https://learning.postman.com/docs/postman-cli/postman-cli-installation/) (`npm install -g postman-cli`)
- `jq` for JSON processing
- `curl`

## Installation

```bash
# Clone the repo
git clone https://github.com/postman-cs/user-activity.git
cd user-activity

# Make executable
chmod +x postman-report

# Optionally add to PATH
ln -s $(pwd)/postman-report /usr/local/bin/postman-report
```

## Usage

```bash
postman-report [report_type] [--format json|table|csv]
postman-report --login
postman-report --help
```

### Report Types

| Type | Description |
|------|-------------|
| `users` | User details: name, email, join date, last active, roles, invite info (default) |
| `activity` | User activity: API requests, collection runs |
| `roles` | User roles summary |

### Output Formats

| Format | Description |
|--------|-------------|
| `json` | Raw JSON response (default) |
| `table` | Formatted table for terminal |
| `csv` | CSV for spreadsheets |

## Examples

```bash
# Fetch users report as JSON
postman-report

# Fetch as formatted table
postman-report --format table

# Export to CSV
postman-report --format csv > users.csv

# Fetch activity report
postman-report activity

# Fetch roles as table
postman-report roles --format table

# Count users with jq
postman-report | jq '.data | length'

# Re-authenticate
postman-report --login
```

## Authentication

The tool uses the Postman CLI's authentication. On first run (or if not logged in), it will prompt you to authenticate via browser.

Credentials are stored in `~/.postman/postmanrc` by the Postman CLI.

To force re-authentication:
```bash
postman-report --login
```

## Sample Output

### Table Format
```
NAME              EMAIL                         JOINED      LAST_ACTIVE  ROLES
John Smith        john@example.com              2023-01-15  2024-01-20   Admin, Developer
Jane Doe          jane@example.com              2023-03-22  2024-01-19   Developer
```

### JSON Format
```json
{
  "data": [
    {
      "measures": {},
      "dimensions": {
        "userName": "John Smith",
        "email": "john@example.com",
        "userAddedInTeam": "2023-01-15",
        "lastActiveDate": "2024-01-20",
        "userRole": "Admin, Developer",
        "invitedBy": "admin@example.com",
        "inviteType": "Email Invite",
        "publicProfileEnabled": "Yes"
      }
    }
  ]
}
```

## License

MIT
