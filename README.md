![Lacework Monthly Alert Chart](lw-alerts-prev-month.png)
---

# Lacework Threat Center Monthly Alert Report

PowerShell 7 Stacked Severity Chart Generator

## Overview

This script connects to the Lacework API, retrieves all Threat Center alerts for the previous calendar month, and generates a stacked bar chart showing daily alert volume by severity. The output is a PNG image suitable for executive reporting, SOC reviews, audits, or long term trend tracking.

The script is designed for reliability in real environments. It respects Lacework API constraints, enforces rate limits, handles schema drift, and avoids common PowerShell path resolution issues that cause automation failures.

## Key Features

* Retrieves Threat Center alerts for the **previous calendar month**
* Automatically chunks API requests to comply with **7 day query limits**
* Enforces **configurable rate limiting** up to 400 API calls per hour
* Generates a **stacked bar chart** with:

  * Date on the X axis
  * Alert count on the Y axis
  * Severity stacked per day
* Applies consistent **severity colors**:

  * Critical: Dark Red
  * High: Red
  * Medium: Orange
  * Low: Yellow
  * Info: Grey
* Orders severity consistently:

  * Graph stack bottom to top: Info, Low, Medium, High, Critical
  * Legend top to bottom: Critical, High, Medium, Low, Info
* Displays **monthly totals per severity in the legend**
* Anchors all relative paths to the script directory
* Fails fast with a clear message if PowerShell 7 is not used

## Example Output

The generated PNG shows daily alert volume with severity stacked visually. This allows quick identification of:

* Alert spikes
* Severity distribution trends
* Days dominated by Critical or High alerts
* Overall alert posture month over month

## Requirements

* PowerShell 7 or later
* Windows system with .NET charting support
* Lacework API access with:

  * Key ID
  * Secret key
  * Permissions to query Alerts

PowerShell 5.1 is not supported and will exit with an informational message.

## Installation

1. Clone the repository

```
git clone https://github.com/your-org/lacework-monthly-alert-chart.git
cd lacework-monthly-alert-chart
```

2. Ensure PowerShell 7 is installed

```
pwsh --version
```

If not installed, download from:
[https://aka.ms/powershell](https://aka.ms/powershell)

3. Review the script and confirm execution policy allows local scripts

## Usage

Run the script from PowerShell 7:

```
pwsh
```

Then execute:

```
.\Get-LaceworkAlertsPrevMonthChart.ps1 `
  -LaceworkAccount "your-account-name" `
  -KeyId "YOUR_KEY_ID" `
  -SecretKey "YOUR_SECRET_KEY" `
  -OutputPngPath ".\lw-alerts-prev-month.png" `
  -StackBy "severity" `
  -MaxApiCallsPerHour 400
```

### Parameters

| Parameter          | Description                                                            |
| ------------------ | ---------------------------------------------------------------------- |
| LaceworkAccount    | Lacework tenant name, for example `acme`                               |
| KeyId              | Lacework API Key ID                                                    |
| SecretKey          | Lacework API Secret                                                    |
| OutputPngPath      | Output image path. Relative paths are resolved to the script directory |
| StackBy            | Field used for stacking. Default is `severity`                         |
| MaxApiCallsPerHour | API rate limit. Default and recommended value is 400                   |
| TokenExpirySeconds | OAuth token lifetime. Default 3600                                     |

## Rate Limiting Behavior

The script enforces a sliding window rate limit. If the maximum calls per hour is reached, execution pauses automatically until additional calls are allowed. This prevents accidental API abuse and avoids Lacework throttling.

## API Query Strategy

Lacework enforces a maximum difference of 7 days between `startTime` and `endTime` for alert searches. The script automatically:

* Divides the previous month into 7 day windows
* Queries each window sequentially
* Aggregates results into a single dataset
* Deduplicates implicitly by date grouping

No manual intervention is required.

## Security Considerations

* API credentials are passed as parameters only
* No credentials are written to disk
* No data is transmitted outside Lacework APIs
* Output contains only aggregate alert counts, not raw alert payloads

For automation, consider injecting credentials via environment variables or a secure secret manager.

## Automation and Scheduling

This script is safe to run unattended and is suitable for:

* Windows Task Scheduler
* CI pipelines
* Monthly SOC reporting jobs
* Compliance evidence generation

Because paths are resolved relative to the script location, it behaves predictably regardless of execution context.

## Troubleshooting

### Script exits immediately

Ensure you are running PowerShell 7:

```
pwsh
```

### Access denied writing PNG

Use a path where the current user has write permissions. Relative paths resolve to the script directory.

### Empty chart

Confirm that alerts exist in the previous calendar month and that the API key has alert read permissions.

### API errors

Reduce `MaxApiCallsPerHour` if running alongside other Lacework automation using the same credentials.

## Roadmap Ideas

Possible future enhancements:

* CSV export alongside PNG
* Per account or per resource grouping
* Severity threshold highlighting
* Multi month trend overlays
* Headless chart rendering for Linux

## License

MIT License. Use, modify, and distribute freely.

## Disclaimer

This script is not affiliated with or endorsed by Lacework. It is provided as is, without warranty. Always test in a non production environment before operational use.

