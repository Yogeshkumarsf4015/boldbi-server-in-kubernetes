# Upgrade Center User Guide

## Overview

Upgrade Center is an in-application upgrade management feature that helps administrators upgrade a Bold BI Kubernetes deployment from the currently installed version to a newer supported version.

## Purpose and Benefits

Upgrade Center is designed to reduce manual upgrade effort and provide a safer, more transparent upgrade experience for Bold BI.

Key benefits:

- View available product versions from within the application.
- Validate the environment before upgrade.
- Identify database tables affected by schema changes.
- Back up affected database tables before upgrade.
- Upgrade application deployments in a controlled sequence.
- Monitor upgrade progress stage by stage.
- View validation results, operation logs, and history.
- Roll back to the previous version when needed.

## Supported Deployment Environments

Upgrade Center is currently supported only for Kubernetes-based deployments.

Supported environments may include:

- Kubernetes
- Google Kubernetes Engine
- Azure Kubernetes Service
- Amazon Elastic Kubernetes Service
- Rancher-managed Kubernetes
- Other supported Kubernetes distributions

Availability depends on the deployment configuration, image registry access, database support, and the namespace-scoped Kubernetes permissions provided to Upgrade Center.

## How to Enable Upgrade Center in Kubernetes

Follow the deployment steps available in the Upgrade Center deployment documentation:

- Using kubectl: [upgrade-center-deployment.md](./upgrade-center-deployment.md#using-kubectl)
- Using Helm: [upgrade-center-deployment.md](./upgrade-center-deployment.md#using-helm)

The Upgrade Center Kubernetes permissions are namespace scoped. It uses Role and RoleBinding resources in the namespace where Bold BI is deployed, and it does not require cluster-wide permissions for normal operation.

## Accessing Upgrade Center

Administrators can access Upgrade Center from the product menu using the **Check for Updates** option.

The Upgrade Center page displays:

- Currently installed version
- Latest available version
- Available releases
- Custom patch option
- Upgrade history
- Rollback availability

The currently installed version is retrieved from the product version source used by the application. If the installed version is already the latest available version, the available release list is hidden and the page shows a message that no newer versions are available.

(Need screenshot)

## Standard Release Upgrade

Use the **Available releases** tab to upgrade to an officially available product version.

Steps:

1. Open Upgrade Center.
2. Go to **Available releases**.
3. Review the available versions.
4. Select the required target version.
5. Click **Upgrade**.
6. Review the confirmation dialog.
7. Check the affected database tables and warnings.
8. Click **Start Upgrade** to begin.

(Need screenshot)

## Custom Patch Upgrade

Use the **Custom patch** tab when Bold BI Support provides a custom image tag or patch version.

Steps:

1. Open Upgrade Center.
2. Go to **Custom patch**.
3. Enter the custom patch version or image tag.
4. Optionally enter a custom image repository.
5. Click **Validate**.
6. Review generated image references and validation results.
7. Proceed only if all required images are valid.
8. Confirm and start the upgrade.

Notes:

- The image tag must be valid and accessible.
- The image repository must be allowed by the configured registry validation rules.
- The patch version must be greater than the currently installed version.
- The custom patch tag may include additional text after the product version, such as `17.1.11_patch`; Upgrade Center uses the product version portion for version comparison.
- Custom patch upgrades update Kubernetes images and use the same monitoring, validation, cleanup, and rollback views as standard upgrades.
- Database schema backup is skipped for custom patch upgrades.

(Need screenshot)

## Upgrade Confirmation

Before the upgrade starts, the confirmation dialog may show:

- Current version
- Target version
- Database schema impact
- Affected table count
- Affected table list
- Important warnings
- Upgrade restrictions, if any

The upgrade starts only after the administrator confirms the operation.

The **Start Upgrade** button is disabled when the selected workflow is not supported for the current deployment. For example, Bold BI upgrades are blocked for currently unsupported shared database configurations and Oracle database configurations.

(Need screenshot)

## Upgrade Workflow

The standard upgrade process follows this sequence:

1. Identify current and target versions.
2. Analyze database schema changes.
3. Back up affected database tables.
4. Run pre-upgrade validation.
5. Update Kubernetes deployment images.
6. Wait for deployment rollout completion.
7. Run product health checks.
8. Run post-upgrade validation.
9. Run cleanup tasks, where applicable.
10. Mark the upgrade as completed.

For custom patch upgrades, the schema backup stage is skipped because custom patches are image-only updates in the current workflow.

## Database Backup

Before changing application images, Upgrade Center identifies affected database tables and backs them up.

Backup behavior:

- Only existing affected tables are backed up.
- Missing affected tables are skipped and logged.
- Duplicate tables are backed up only once per database.
- Newly created tables from upgrade scripts are tracked for rollback cleanup.
- A backup database is created only when at least one existing affected table requires backup.
- All discovered databases are checked uniformly.
- Tables are resolved using configured schema and table prefix where applicable.
- Tenant database discovery processes only active tenants.

The database backup performed by Upgrade Center is limited to affected tables identified for the upgrade. It is not a replacement for a full customer-managed database backup.

## Pre-Upgrade Validation

Pre-upgrade validation checks whether the existing environment is ready for upgrade.

Depending on configuration, validation may include:

- Product health checks
- Playwright-based automated validation
- Required service availability checks

If pre-upgrade validation fails, the upgrade does not proceed to image update.

When the Playwright Kubernetes runner is disabled, Upgrade Center uses product-level health checks instead of the Playwright validation job.

## Kubernetes Upgrade

During the upgrade stage, Upgrade Center updates Kubernetes deployment images and monitors rollout status.

Deployments are upgraded in predefined groups. Deployments within a group may be updated in parallel. The next group starts only after the current group is healthy.

If a required deployment fails, the upgrade stops and rollback may be triggered.

After Kubernetes rollout succeeds, Upgrade Center performs product-level health checks for the configured Bold BI services. These checks help confirm that the upgraded pods are not only ready in Kubernetes, but also responding through the product health endpoints.

## Post-Upgrade Validation

Post-upgrade validation confirms that the upgraded application is working as expected.

Depending on configuration, validation may include:

- Product-level health checks
- Playwright-based automated validation
- Service availability checks

If post-upgrade validation fails, automatic rollback may be triggered.

When the Playwright Kubernetes runner is disabled, post-upgrade validation is based on product-level health checks.

## Monitoring an Upgrade

After the upgrade begins, the monitoring page displays stages such as:

- Schema Backup
- Pre-Upgrade Validation
- Upgrade
- Post-Upgrade Validation
- Cleanup Job, where applicable

(Need screenshot)

Each stage may show:

- Status
- Progress percentage
- Duration
- View Details
- Complete operation logs
- HTML validation report, where available

Common status values:

- **Pending**: Stage has not started.
- **Running**: Stage is currently in progress.
- **Passed**: Stage completed successfully.
- **Failed**: Stage failed and requires attention.
- **Skipped**: Stage was not required or disabled.
- **Rolled back**: Upgrade was reverted to the previous version.
- **Cancelled**: Operation was cancelled by the administrator.

## Validation Reports and Logs

Upgrade Center provides logs and validation results to help administrators understand upgrade progress.

Stage details contain concise progress information relevant to each stage.

Complete operation logs provide detailed operation-level information, including:

- Database backup and restore messages
- Kubernetes rollout progress
- Validation execution details
- Warnings
- Errors
- Final operation status

When Playwright validation is enabled and the report is available, the monitoring page may display a **View HTML Report** option for the pre-upgrade and post-upgrade validation stages.

Playwright result handling:

- Validation runs in a Kubernetes Job using the configured Playwright runner image.
- Runtime credentials and database values are supplied through Kubernetes Secrets.
- A shared Playwright state volume is used where validation cleanup or post-upgrade validation requires files produced by earlier validation steps.
- The pass-rate threshold is configurable. If it is not configured, Upgrade Center uses 100%.
- Stage summaries show total, passed, failed, skipped, pass percentage, configured threshold, and final validation status when these values are available.
- HTML reports are uploaded back to Upgrade Center and served through an authenticated Upgrade Center endpoint.
- Reports are stored separately by upgrade job and validation stage.
- Reports are retained for the latest seven upgrade jobs. Older local report files are removed during report cleanup.
- If the configured application data storage is not persistent, reports can be lost when the Upgrade Center pod is recreated.

The **Download Complete Logs** option provides the full operation log for troubleshooting.
(Need screenshot)

## Manual Rollback

Manual rollback is available from the Rollback or History section when a valid rollback point exists.

Steps:

1. Open Upgrade Center.
2. Go to the Rollback or History section.
3. Select the available rollback point.
4. Review the rollback confirmation dialog.
5. Confirm rollback.
6. Monitor rollback progress.
(Need screenshot)

During rollback, Upgrade Center restores backed-up affected database tables where a backup exists and reverts Kubernetes deployment images to the previously recorded image references.

## Automatic Rollback

Automatic rollback may be triggered when a failure occurs after upgrade changes have been applied, such as:

- Kubernetes rollout failure
- Product health-check failure
- Post-upgrade validation failure

If failure occurs before any application changes are applied, rollback may not be required.

Automatic rollback uses the same rollback workflow as manual rollback, but it is started by Upgrade Center after a qualifying failure.

## Cancellation and Recovery

An administrator can cancel an active operation when cancellation is still safe for the current stage.

Cancellation behavior:

- If cancellation happens before application images are changed, Upgrade Center stops the remaining upgrade stages.
- If cancellation happens after application images have been changed, Upgrade Center may trigger rollback to restore the previous version.
- Running Playwright validation jobs and cleanup jobs are stopped or cleaned up where possible.
- The monitoring page and History section show the final cancelled, failed, or rolled-back state.
- If the Upgrade Center pod is recreated during an operation, Upgrade Center reconciles persisted job state and Kubernetes state before allowing another operation to start.

## Cleanup Job

Where configured, Cleanup Job removes validation-created resources such as temporary test users, sites, or data created during automated validation.

Cleanup Job uses the Playwright Kubernetes runner. If the Kubernetes runner is disabled, cleanup is skipped and shown as skipped in the monitoring page.

Cleanup failure does not necessarily mean the upgrade failed, but manual cleanup may be required.

## Upgrade History

The History section shows previous upgrade and rollback operations.

History may include:

- Source version
- Target version
- Operation type
- Status
- Initiated user
- Open/details option where available

Use History to review completed, failed, cancelled, and rolled-back operations.
(Need screenshot)

History entries include an **Open** option when job details are available, including failed and rolled-back jobs.

## Best Practices

Before upgrading:

- Review release notes.
- Ensure recent infrastructure/database backups are available.
- Confirm cluster health.
- Confirm image registry access.
- Avoid starting upgrades during peak usage.
- Ensure no manual deployment changes are running.
- Keep the browser monitoring page open where possible.
- Review logs and reports after completion.
- Ensure the configured Upgrade Center administrator credentials are stored securely in Kubernetes Secrets.
- Ensure the Playwright runner image and allowed image repositories are configured before using validation or custom patch workflows.

## Limitations

- Only supported Kubernetes deployments are handled.
- Upgrade Center must have required Kubernetes permissions.
- Custom patch images must be accessible from the configured registry.
- Upgrade Center supports upgrading existing configured services. If a new Bold BI service or a new environment-specific deployment is introduced, update the Upgrade Center configuration before using this workflow.

The following scenarios may not be supported unless explicitly enabled:
- Non-Kubernetes deployments
- Unsupported database engines, such as Oracle for the current Bold BI Playwright validation workflow
- Shared database configurations that are currently blocked by Upgrade Center validation
- Missing Kubernetes permissions
- Invalid or inaccessible image repositories
- Multiple simultaneous upgrades
- Manual changes to deployments during an active upgrade
- Expired or deleted rollback backup data

## Required Configuration

Before using Upgrade Center, verify that the deployment provides:

- Upgrade Center feature enablement configuration.
- Kubernetes hosting environment configuration.
- IDP configuration endpoint access.
- Namespace-scoped Kubernetes RBAC for deployments, jobs, pods, pod logs, secrets, and related Upgrade Center resources.
- Allowed image repositories and registry hosts for standard and custom patch image validation.
- Playwright runner image, timeout, pass-rate threshold, and resource settings when Playwright validation is enabled.
- Secure Kubernetes Secret values for required administrator credentials used by validation.
- Supported database connection details through the existing Bold ID configuration and encrypted connection-string workflow.

Do not place passwords, API keys, database credentials, or customer-specific values directly in this guide, Helm values committed to source control, or public documentation.

## Screenshot Checklist

Add screenshots for the following areas:

- Upgrade Center landing page
- Available releases tab
- Custom patch tab
- Upgrade confirmation dialog
- Database impact section
- Upgrade monitoring page
- Stage details expanded view
- Complete operation logs
- HTML validation report link
- Successful upgrade status
- Failed upgrade status
- Rollback confirmation dialog
- Rollback monitoring page
- Upgrade history tab
- No rollback point available state
- Already latest version state
