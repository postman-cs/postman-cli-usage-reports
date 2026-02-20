# Usage Reports via Postman CLI

A command-line tool for extracting Postman team analytics and usage data. Generate reports on users, API inventory, workspace activity, test coverage, collaboration trends, and Private API Network metrics.

**Features:**
- 28 report commands covering users, content, coverage, trends, and team metadata
- Output formats: JSON (for scripting), table (for terminal), CSV (for spreadsheets)
- Leverages Postman CLI authentication - no API keys to manage
- Exportable data for dashboards, audits, and capacity planning

**Use cases:**
- Track user activity and identify inactive accounts
- Audit API coverage (mocks, monitors, tests, documentation)
- Monitor workspace and API growth trends
- Export collection/environment/monitor inventories
- Report on Private API Network adoption

## Prerequisites

- [Postman CLI](https://learning.postman.com/docs/postman-cli/postman-cli-installation/) (`npm install -g postman-cli`)
- `jq` for JSON processing
- `curl`
- `bc` for percentage calculations (coverage reports)

## Installation

```bash
# Clone the repo
git clone https://github.com/postman-cs/postman-cli-usage-reports.git
cd postman-cli-usage-reports

# Make executable
chmod +x postman-report

# Optionally add to PATH
ln -s $(pwd)/postman-report /usr/local/bin/postman-report
```

## Usage

```bash
postman-report [report] [options]
postman-report --login
postman-report --help
```

## Reports

### User Reports

| Report | Description |
|--------|-------------|
| `users` | User details: name, email, join date, last active, roles, invite info (default) |
| `activity` | User activity: API requests, collection runs per user |
| `roles` | User roles summary |

### Content Reports

| Report | Description |
|--------|-------------|
| `summary` | Dashboard with key metrics (APIs, workspaces, entities) |
| `apis` | API distribution by workspace type (personal, private, team, public, PAN) |
| `workspaces` | Workspace counts by visibility (Partner, Personal, Private, Public, Team) |
| `schemas` | Schema format distribution (OpenAPI, GraphQL, AsyncAPI, RAML, WSDL, protobuf) |
| `entities` | Entity counts in team workspaces (Collections, APIs, Envs, Monitors, Mocks) |

### Coverage Reports

| Report | Description |
|--------|-------------|
| `coverage` | API coverage summary for mocks, monitors, tests, documentation |
| `coverage mocks` | Mock coverage only |
| `coverage monitors` | Monitor coverage only |
| `coverage tests` | Test coverage only |
| `coverage docs` | Documentation coverage only |

### Trend Reports

| Report | Description |
|--------|-------------|
| `trends` | APIs and workspaces created over time |
| `api-health` | Test results (pass/fail), uptime, response time |
| `api-updates` | API updates over time |
| `engagement` | Watches and comments over time |
| `forks-prs` | Forks and pull requests over time |

### List Reports

| Report | Description |
|--------|-------------|
| `collections` | List all collections in team workspaces |
| `workspace-list` | List all team workspaces with IDs |
| `apis-list` | List all APIs in team workspaces |
| `environments` | List all environments in team workspaces |
| `mocks-list` | List all mocks in team workspaces |
| `monitors-list` | List all monitors in team workspaces |

### Private API Network

| Report | Description |
|--------|-------------|
| `pan` | Private API Network metrics: total APIs, schema types, coverage (mocks, monitors, tests, docs), watches, comments |

### Team Reports

| Report | Description |
|--------|-------------|
| `team` | Team info: name, plan, billing frequency, renewal date, member count, SSO provider |

### System Reports

| Report | Description |
|--------|-------------|
| `freshness` | Data freshness timestamp (when reports were last updated) |
| `filters` | Available filters for dynamic queries (counts of all APIs, team APIs, PAN APIs) |

## Options

| Option | Description |
|--------|-------------|
| `--format json\|table\|csv` | Output format (default: json) |
| `--limit N` | Limit results for time series reports (default: 10) |
| `--raw` | Output raw API response without formatting |
| `--login` | Re-authenticate with Postman |
| `-h, --help` | Show help |

## Examples

### User Reports

```bash
# Default: user details as JSON
postman-report

# User list as table
postman-report users --format table

# Export activity to CSV
postman-report activity --format csv > activity.csv

# User roles
postman-report roles --format table
```

### Content Reports

```bash
# Quick summary dashboard
postman-report summary --format table

# API distribution
postman-report apis --format table

# Workspace breakdown
postman-report workspaces --format csv

# Schema types in use
postman-report schemas --format table
```

### Coverage Reports

```bash
# Full coverage summary
postman-report coverage --format table

# Just test coverage
postman-report coverage tests --format table

# Export all coverage to CSV
postman-report coverage --format csv > coverage.csv
```

### Trend Reports

```bash
# APIs/workspaces created over time (last 10 periods)
postman-report trends --format table

# More history
postman-report trends --format table --limit 30

# API health metrics
postman-report api-health --format table

# Forks and PRs
postman-report forks-prs --format table

# Engagement trends
postman-report engagement --format table
```

### List Reports

```bash
# List all collections
postman-report collections --format table

# Export collection inventory
postman-report collections --format csv > collections.csv

# List all team workspaces
postman-report workspace-list --format table

# List all APIs
postman-report apis-list --format csv > apis.csv

# List environments, mocks, monitors
postman-report environments --format table
postman-report mocks-list --format table
postman-report monitors-list --format table
```

### Private API Network

```bash
# PAN metrics and coverage
postman-report pan --format table
```

### Team Info

```bash
# Team details including SSO
postman-report team --format table
```

### System Reports

```bash
# Check when data was last refreshed
postman-report freshness --format table

# See available filters for queries
postman-report filters --format table
```

### JSON Output (Default)

JSON is the default output format. Omit `--format` or pass `--format json` explicitly.

```bash
# User details (default format is JSON)
postman-report users

# Team info as JSON
postman-report team --format json

# Summary dashboard as JSON
postman-report summary

# Coverage as JSON (aggregated across all types)
postman-report coverage

# Single coverage type as JSON (raw API response)
postman-report coverage tests

# Trends as JSON (multiple series in one object)
postman-report trends

# List reports as JSON
postman-report collections
```

Some reports compose JSON from multiple API calls. For example, `summary` merges four endpoints into one object, while `collections` passes the raw API response through.

### Data Processing with jq

```bash
# Count users
postman-report users | jq '.data | length'

# Find inactive users (no activity in 30 days)
postman-report users | jq '.data[] | select(.dimensions.lastActiveDate < "2024-01-01")'

# Get APIs with tests
postman-report coverage tests | jq '.data.dataset[0].values[0].x'

# Export all API IDs from filters
postman-report filters | jq -r '.apis.allApis[].id'

# Extract entity counts as key:value pairs
postman-report summary | jq '{totalAPIs, teamAPIs, totalWorkspaces}'

# List all collection names
postman-report collections | jq -r '.data.dataset[].value.collectionName'

# Get workspace types and counts
postman-report workspaces | jq '[.data.dataset[].values[] | {type: .x, count: .y}]'
```

## Authentication

The tool uses the Postman CLI's authentication. On first run (or if not logged in), it will prompt you to authenticate via browser.

Credentials are stored in `~/.postman/postmanrc` by the Postman CLI.

To force re-authentication:
```bash
postman-report --login
```

## Sample Output

### Team (JSON)
```json
{
  "teamName": "Postman Customer Org",
  "teamCreated": "2017-11-05T17:11:09.000Z",
  "currentPlan": "Enterprise",
  "billingFrequency": "annual",
  "nextRenewal": "0000-00-00 00:00:00",
  "memberCount": 123,
  "ssoProvider": "saml"
}
```

### Summary (JSON)
```json
{
  "totalAPIs": 2193,
  "teamAPIs": 411,
  "totalWorkspaces": 1027,
  "entities": {
    "labels": null,
    "dataset": [
      { "type": "bar", "legend": "Environments", "values": [{ "x": 0, "y": 657 }] },
      { "type": "bar", "legend": "Monitors", "values": [{ "x": 0, "y": 87 }] },
      { "type": "bar", "legend": "Mocks", "values": [{ "x": 0, "y": 214 }] },
      { "type": "bar", "legend": "Collections", "values": [{ "x": 0, "y": 2010 }] },
      { "type": "bar", "legend": "APIs", "values": [{ "x": 0, "y": 411 }] }
    ]
  }
}
```

### Users (JSON)
```json
{
  "data": [
    {
      "measures": {},
      "dimensions": {
        "userName": "Jane Smith",
        "email": "jane.smith@example.com",
        "userAddedInTeam": "2024-03-15",
        "lastActiveDate": "2026-02-19",
        "userRole": "Admin, Developer",
        "invitedBy": "admin@example.com",
        "inviteType": "Invite Link",
        "publicProfileEnabled": "Yes"
      }
    }
  ],
  "meta": { ... },
  "numRows": 123
}
```

### Coverage (JSON)
```json
{
  "mocks": {
    "id": "APIsWithMocks",
    "label": "API mock coverage",
    "data": {
      "dataset": [
        { "legend": "With mocks", "values": [{ "x": 43 }] },
        { "legend": "Without mocks", "values": [{ "x": 368 }] }
      ]
    }
  },
  "monitors": { ... },
  "tests": { ... },
  "documentation": { ... }
}
```

### Freshness (JSON)
```json
{
  "lastRefreshedAt": "2026-02-20T02:47:33.000Z"
}
```

### Summary (Table)
```
METRIC                              VALUE
Total APIs                           2193
APIs in Team Workspaces               411
Total Workspaces                     1027
Environments                          657
Monitors                               87
Mocks                                 214
Collections                          2010
APIs                                  411
```

### Coverage (Table)
```
TYPE                 WITH    WITHOUT   COVERAGE
Mocks                  43        368      10.4%
Monitors               11        400       2.6%
Tests                  18        393       4.3%
Documentation          18        393       4.3%
```

### Team (Table)
```
FIELD                VALUE
Team Name            Postman Customer Org
Created              2017-11-05T17:11:09.000Z
Plan                 Enterprise
Billing              annual
Next Renewal         0000-00-00 00:00:00
Members              123
SSO Provider         saml
```

### Freshness (Table)
```
FIELD                VALUE
Last Refreshed       2026-02-04T08:44:08.000Z
```

### Filters (Table)
```
FILTER_TYPE                    COUNT
All APIs                         806
Team APIs                        451
Private Network APIs               0
```

## API Endpoints Used

This tool uses Postman's reporting API:

### v3 Views (flexible queries)
- `/v3/views/BillingUsersDimension` - User data with dimensions/measures
- `/v3/views/TeamBillingDimension` - Team/billing info
- `/v3/filter-listing` - Available filters for dynamic queries

### v2 Stored Reports
- `/v2/reports/stored/totalAPIs` - Total API count
- `/v2/reports/stored/teamTotalApisInTeamWorkspace` - Team workspace APIs
- `/v2/reports/stored/totalWorkspaces` - Total workspaces
- `/v2/reports/stored/distributionOfApis` - API distribution
- `/v2/reports/stored/workspacesByType` - Workspace types
- `/v2/reports/stored/teamSchemasType` - Schema formats
- `/v2/reports/stored/teamWorkspaceEntities` - Entity counts
- `/v2/reports/stored/teamApiWith*` - Coverage reports (Mocks, Monitors, Test, Documentation)
- `/v2/reports/stored/apisCreatedOverTime` - API trends
- `/v2/reports/stored/workspacesCreatedOverTime` - Workspace trends
- `/v2/reports/stored/activeWorkspacesOverTime` - Active workspace trends
- `/v2/reports/stored/teamTestResult` - Test results
- `/v2/reports/stored/teamApiUptime` - API uptime
- `/v2/reports/stored/teamResponseTime` - Response times
- `/v2/reports/stored/teamApiUpdatedOvertime` - API updates
- `/v2/reports/stored/teamWatchesOvertime` - Watch activity
- `/v2/reports/stored/teamCommentsOvertime` - Comment activity
- `/v2/reports/stored/teamWSForksPullRequestsOverTime` - Forks and PRs
- `/v2/reports/stored/teamSSO` - SSO provider
- `/v2/reports/stored/*Drill` - Drill-down reports (collections, workspaces, APIs, environments, mocks, monitors)

### v2 Report Groups
- `/v2/reports/group/privateNetworkApis` - Private API Network metrics

### v1 Simple Endpoints
- `/v1/last-refreshed` - Data freshness timestamp

## License

MIT
