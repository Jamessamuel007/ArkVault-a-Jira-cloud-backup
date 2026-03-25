ArkVault — Full Documentation
Overview
ArkVault is a Jira Cloud backup and restore app built on Atlassian Forge. It automatically backs up your Jira issues, comments, attachments, and custom fields to your own cloud storage — and lets you restore them at any time with full control over scope and granularity.

Your data is never stored on ArkVault's servers. All backups go directly to your configured storage provider.

Getting Started
1. Install the app
Install ArkVault from the Atlassian Marketplace. Once installed, navigate to:
Jira Settings → Apps → ArkVault

2. Configure storage
Before running a backup, connect a storage provider. ArkVault supports:

AWS S3
Azure Blob Storage
Google Cloud Storage (via S3-compatible HMAC)
Wasabi
Backblaze B2
Supabase
Go to the Storage Settings card and enter your credentials. ArkVault never exposes your secret keys — they are stored encrypted within Atlassian Forge Storage.

3. Run your first backup
Click Run Full Backup in the Backup tab. This captures all issues across your configured projects and saves them to Forge Storage. Once complete, click Export to S3 (or your provider) to push the snapshot to your storage bucket.

Backup
Full Backup
A full backup captures all issues across all selected projects at a point in time. Run this first before enabling incremental backups.

Incremental Backup
After a full backup exists, ArkVault captures only issues that changed since the last run — much faster and storage-efficient. Click Run Incremental Now or let the schedule handle it automatically.

When you click Sync to S3, ArkVault merges all incremental changes on top of the full backup and uploads a single up-to-date snapshot — no fragmented delta files in your storage bucket.

Schedules
Option	Behavior
None	Manual only
Hourly	Runs every hour
Daily	Runs once per 24 hours
Backup Settings
Issue Age Filter
Limit which issues are backed up by creation date. Options:

3 months, 6 months, 1 year, 2 years, All time
Or pick a specific date
Project Scope

All projects — backs up every project in your Jira instance
Select projects — choose specific projects to include
Content Backup

Backup comments — stores the full comment thread per issue (all pages, not just Jira's default 20)
Backup attachment metadata — stores filenames, sizes, MIME types, and content URLs (file binaries are not stored)
Both options can be scoped to all backed-up projects or specific projects only.

Restore
Loading Snapshots
Go to the Restore tab. Select a storage connection from the dropdown and click Load Snapshots. ArkVault reads the manifest index from your storage bucket and lists all available snapshots.

You can load snapshots from any connected storage provider — including previous providers you've since switched away from.

Restore Scopes
Scope	Behavior
Full instance	Restores all issues from the snapshot
Selected projects	Restores issues from chosen projects only
Specific issues	Restores a comma-separated list of issue keys
JQL	Restores issues matching a JQL query
Note: JQL restore works even for deleted issues that are no longer visible in live Jira search. ArkVault extracts the project keys from your JQL and matches against the backup snapshot directly.

As-of Date Filter
Optionally filter the restore to only issues created on or before a specific date — useful for point-in-time recovery.

How Restore Works (3-Step Upsert)
For each issue, ArkVault tries in order:

Update by original key — if the issue still exists, update it
Search by summary — if the key is gone but the issue was recreated under a new key, find and update it
Create new — if the issue is genuinely gone, recreate it (Jira assigns a new key)
This makes restores fully idempotent — running restore multiple times never creates duplicates.

Smart Deduplication
When ArkVault creates a new issue during restore (e.g. A-1 → A-2), it remembers this mapping. On the next restore run, it goes directly to A-2 and updates it instead of creating A-3. This mapping persists across all future restores.

Restore Log
After every restore, ArkVault generates a downloadable log listing every issue processed:

Updated — issue was found and updated
Created — issue was recreated with a new key
Failed — issue could not be restored (error included)
Download the log as JSON or CSV for audit and compliance purposes.

Storage Providers
AWS S3
Field	Description
Bucket	Your S3 bucket name
Region	e.g. us-east-1
Access Key ID	IAM user access key
Secret Access Key	IAM user secret key
Prefix	Folder path inside the bucket e.g. jira-backups
Minimum IAM permissions required: s3:PutObject, s3:GetObject

Azure Blob Storage
Field	Description
Account Name	Your storage account name
Account Key	Storage account access key
Container Name	Blob container name
Prefix	Folder path inside the container
Google Cloud Storage
Use the S3-Compatible provider option with:

Endpoint: storage.googleapis.com
Access Key ID: GCS HMAC Access ID
Secret: GCS HMAC Secret
Create HMAC keys in GCP Console → Cloud Storage → Settings → Interoperability.

Wasabi
Use S3-Compatible with endpoint: s3.us-east-1.wasabisys.com (replace region as needed)

Backblaze B2
Use S3-Compatible with endpoint: s3.us-west-004.backblazeb2.com (replace region as needed)

Supabase
Field	Description
Project URL	Your Supabase project URL
Service Role Key	Found in Project Settings → API
Bucket	Storage bucket name
Prefix	Folder path inside the bucket
Multiple Storage Connections
ArkVault supports multiple storage connections simultaneously. You can:

Switch your active storage provider at any time
Restore from any previous connection's snapshots
Mix restores across providers — restore some issues from one snapshot, others from another
Previous connections remain readable even after switching to a new active provider.

Permissions & Security
ArkVault requests the following Jira scopes:

read:jira-work — read issues and projects for backup
read:jira-user — read user/reporter fields
write:jira-work — update and create issues during restore
storage:app — store backup metadata and configuration within Forge
Your data never leaves your control. ArkVault writes backups directly from your Jira instance to your storage provider. No data is routed through or stored on ArkVault servers.

FAQ
Q: Will restore create duplicate issues if I run it twice?
A: No. ArkVault tracks all previously restored issues and updates them on subsequent runs rather than creating new ones.

Q: Does restore delete the backup from storage?
A: No. The backup file in your storage bucket is never modified or deleted by the restore process. You can restore from the same snapshot as many times as you need.

Q: Can I restore issues that were deleted from Jira?
A: Yes. ArkVault matches against the backup snapshot directly, not live Jira search — deleted issues are fully restorable.

Q: What happens to custom fields during restore?
A: ArkVault automatically detects which custom fields your Jira instance accepts and skips any that are read-only or managed by other systems (e.g. Sprint, Rank, Epic Link). No manual configuration needed.

Q: Does ArkVault back up attachment files?
A: ArkVault backs up attachment metadata (filename, size, MIME type, URL). Actual file binaries are not stored — this keeps backup sizes manageable and avoids Forge storage limits.

Q: Can I back up only specific projects?
A: Yes. In Backup Settings, switch from "All projects" to "Select projects" and choose exactly which ones to include.

Support
For bug reports, feature requests, or questions:

GitHub Issues: github.com/edurself/arkvault/issues
Email: support@edurself.com
ArkVault is built and maintained by EdurSelf. Running on Atlassian Forge — secure, scalable, and fully within the Atlassian ecosystem.

