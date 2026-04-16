**plex-pipeline**

**What does it do?**

A Jenkins declarative pipeline that orchestrates the nightly synchronisation, testing, and mailout for the Plex automation project. It ties together the sync scripts, REST Assured test suite, and email notification into a single automated workflow.

**How does it work?**

The pipeline runs nightly at 2am and executes the following stages in sequence:

| Stage | Description |
|---|---|
| `Checkout` | Pulls the latest code from `plex-sync-scripts` and `plex-rest-assured` |
| `Upload to GDrive` | Syncs the local HDD library to Google Drive |
| `Wait for GDrive to Stabilise` | Polls Google Drive until the file count is stable |
| `Download to HDD` | Syncs Google Drive to the remote HDD library |
| `Wait for HDD to Stabilise` | Polls the HDD until the file count is stable |
| `Read Last Run Timestamp` | Reads the last successful run timestamp from disk |
| `Run Plex API Tests` | Executes the REST Assured suite and sends the mailout |

On success, the pipeline writes the current Unix timestamp to a local file. This is read at the start of the next run to determine which content is new.

**Prerequisites**

- Jenkins installed and running on the host machine
- Git installed on the agent
- Java and Maven installed on the agent
- PowerShell 5.1 or later
- Google Drive desktop client configured with access to both locations
- The following credentials configured in Jenkins:

| Credential ID | Type | Description |
|---|---|---|
| `plex-token` | Secret text | Plex API authentication token |
| `tmdb-api-key` | Secret text | TMDB API key |
| `gmail-username` | Secret text | Gmail address to send from |
| `gmail-app-password` | Secret text | Gmail app password |

**Environment Variables**

| Variable | Description |
|---|---|
| `LAST_RUN_FILE` | Path to the timestamp file on the agent |
| `LOG_DIR` | Directory for sync script log output |
| `GDRIVE_FOLDER` | Path to the Google Drive FILMS folder |
| `HDD_FOLDER` | Path to the local HDD FILMS folder |
| `PLEX_URL` | Base URL of the Plex server |
| `MAIL_RECIPIENTS` | Comma-separated list of mailout recipients |

**Relationship to the wider project**

| Repo | Purpose |
|---|---|
| `plex-api-testing` | Postman exploratory collection |
| `plex-rest-assured` | Automated REST Assured test suite |
| `plex-sync-scripts` | PowerShell sync scripts |
| `plex-pipeline` | Jenkins CI/CD pipeline (this repo) |
