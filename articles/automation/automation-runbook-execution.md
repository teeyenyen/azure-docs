---
title: Runbook execution in Azure Automation | Microsoft Docs
description: Describes the details of how a runbook in Azure Automation is processed.
services: automation
documentationcenter: ''
author: eslesar
manager: jwhit
editor: tysonn

ms.assetid: d10c8ce2-2c0b-4ea7-ba3c-d20e09b2c9ca
ms.service: automation
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 11/15/2017
ms.author: magoedte;bwren

---
# Runbook execution in Azure Automation
When you start a runbook in Azure Automation, a job is created. A job is a single execution instance of a runbook. An Azure Automation worker is assigned to run each job. While workers are shared by multiple Azure accounts, jobs from different Automation accounts are isolated from one another. You do not have control over which worker services the request for your job. A single runbook can have multiple jobs running at one time.  The execution environment for jobs from the same Automation Account may be reused. When you view the list of runbooks in the Azure portal, it lists the status of all jobs that were started for each runbook. You can view the list of jobs for each runbook in order to track the status of each. For a description of the different job statuses [Job Statuses](#job-statuses).

The following diagram shows the lifecycle of a runbook job for [Graphical runbooks](automation-runbook-types.md#graphical-runbooks) and [PowerShell Workflow runbooks](automation-runbook-types.md#powershell-workflow-runbooks).

![Job Statuses - PowerShell Workflow](./media/automation-runbook-execution/job-statuses.png)

The following diagram shows the lifecycle of a runbook job for [PowerShell runbooks](automation-runbook-types.md#powershell-runbooks).

![Job Statuses - PowerShell Script](./media/automation-runbook-execution/job-statuses-script.png)

Your jobs have access to your Azure resources by making a connection to your Azure subscription. They only have access to resources in your data center if those resources are accessible from the public cloud.

## Job statuses
The following table describes the different statuses that are possible for a job.

| Status | Description |
|:--- |:--- |
| Completed |The job completed successfully. |
| Failed |For [Graphical and PowerShell Workflow runbooks](automation-runbook-types.md), the runbook failed to compile.  For [PowerShell Script runbooks](automation-runbook-types.md), the runbook failed to start or the job encountered an exception. |
| Failed, waiting for resources |The job failed because it reached the [fair share](#fair-share) limit three times and started from the same checkpoint or from the start of the runbook each time. |
| Queued |The job is waiting for resources on an Automation worker to come available so that it can be started. |
| Starting |The job has been assigned to a worker, and the system is in the process of starting it. |
| Resuming |The system is in the process of resuming the job after it was suspended. |
| Running |The job is running. |
| Running, waiting for resources |The job has been unloaded because it reached the [fair share](#fair-share) limit. It resumes shortly from its last checkpoint. |
| Stopped |The job was stopped by the user before it was completed. |
| Stopping |The system is in the process of stopping the job. |
| Suspended |The job was suspended by the user, by the system, or by a command in the runbook. A job that is suspended can be started again and resume from its last checkpoint or from the beginning of the runbook if it doesn't have checkpoints. The runbook will only be suspended by the system when an exception occurs. By default, ErrorActionPreference is set to **Continue**, meaning that the job keeps running on an error. If this preference variable is set to **Stop**, then the job suspends on an error.  Applies to [Graphical and PowerShell Workflow runbooks](automation-runbook-types.md) only. |
| Suspending |The system is attempting to suspend the job at the request of the user. The runbook must reach its next checkpoint before it can be suspended. If it already passed its last checkpoint, then it completes before it can be suspended.  Applies to [Graphical and PowerShell Workflow runbooks](automation-runbook-types.md) only. |

## Viewing job status from the Azure portal
You can view a summarized status of all runbook jobs or drill into details of a specific runbook job in the Azure portal or by configuring integration with your Microsoft Operations Management Suite (OMS) Log Analytics workspace to forward runbook job status and job streams.  For more information about integrating with OMS Log Analytics, see [Forward job status and job streams from Automation to Log Analytics (OMS)](automation-manage-send-joblogs-log-analytics.md).  

### Automation runbook jobs summary
On the right of your selected Automation account, you can see a summary of all of the runbook jobs for a selected Automation account under **Job Statistics** tile.<br><br> ![Job Statistics tile](./media/automation-runbook-execution/automation-account-job-status-summary.png).<br> This tile displays a count and graphical representation of the job status for all jobs executed.  

Clicking on the tile presents the **Jobs** blade, which includes a summarized list of all jobs executed, with status, job execution, and start and completion times.<br><br> ![Automation account Jobs blade](./media/automation-runbook-execution/automation-account-jobs-status-blade.png)<br><br>  You can filter the list of jobs by selecting **Filter jobs**  and filter on a specific runbook, job status, or from the drop-down list, the date/time range to search within.<br><br> ![Filter Job status](./media/automation-runbook-execution/automation-account-jobs-filter.png)

Alternatively, you can view job summary details for a specific runbook by selecting that runbook from the **Runbooks** blade in your Automation account, and then select the **Jobs** tile.  This presents the **Jobs** blade, and from there you can click on the job record to view its detail and output.<br><br> ![Automation account Jobs blade](./media/automation-runbook-execution/automation-runbook-job-summary-blade.png)<br> 

### Job Summary
You can view a list of all of the jobs that have been created for a particular runbook and their most recent status. You can filter this list by job status and the range of dates for the last change to the job. To view its detailed information and output, click on the name of a job. The detailed view of the job includes the values for the runbook parameters that were provided to that job.

You can use the following steps to view the jobs for a runbook.

1. In the Azure portal, select **Automation** and then select the name of an Automation account.
2. From the hub, select **Runbooks** and then on the **Runbooks** blade select a runbook from the list.
3. On the blade for the selected runbook, click the **Jobs** tile.
4. Click on one of the jobs in the list and on the runbook job details blade you can view its detail and output.

## Retrieving job status using Windows PowerShell
You can use the [Get-AzureRmAutomationJob](https://msdn.microsoft.com/library/mt619440.aspx) to retrieve the jobs created for a runbook and the details of a particular job. If you start a runbook with Windows PowerShell using [Start-AzureRmAutomationRunbook](https://msdn.microsoft.com/library/mt603661.aspx), then it returns the resulting job. Use [Get-AzureRmAutomationJob](https://msdn.microsoft.com/library/mt619440.aspx)Output to get a job’s output.

The following sample commands retrieve the last job for a sample runbook and displays its status, the values provided for the runbook parameters, and the output from the job.

    $job = (Get-AzureRmAutomationJob –AutomationAccountName "MyAutomationAccount" `
    –RunbookName "Test-Runbook" -ResourceGroupName "ResourceGroup01" | sort LastModifiedDate –desc)[0]
    $job.Status
    $job.JobParameters
    Get-AzureRmAutomationJobOutput -ResourceGroupName "ResourceGroup01" `
    –AutomationAccountName "MyAutomationAcct" -Id $job.JobId –Stream Output

## Fair share
In order to share resources among all runbooks in the cloud, Azure Automation will temporarily unload any job after it has been running for three hours.  During this time, jobs for [PowerShell-based runbooks](automation-runbook-types.md#powershell-runbooks) are stopped and will not be restarted.  The job status shows **Stopped**.  This type of runbook is always restarted from the beginning since they don't support checkpoints.  

[PowerShell-Workflow-based runbooks](automation-runbook-types.md#powershell-workflow-runbooks) will be resumed from their last [checkpoint](https://docs.microsoft.com/system-center/sma/overview-powershell-workflows#bk_Checkpoints).  After running three hours, the runbook job will be suspended by the service and its status shows **Running, waiting for resources**.  When a sandbox becomes available, the runbook will be automatically restarted by the Automation service and resumes from the last checkpoint.  This is normal PowerShell-Workflow behavior for suspend/restart.  If the runbook again exceeds three hours of runtime, the process repeats, up to three times.  After the third restart, if the runbook still has not completed in three hours, then the runbook job is failed, and the job status shows **Failed, waiting for resources**.  In this case, you receive the following exception with the failure.

*The job cannot continue running because it was repeatedly evicted from the same checkpoint. Please make sure your Runbook does not perform lengthy operations without persisting its state.*

This is to protect the service from runbooks running indefinitely without completing, as they are not able to make it to the next checkpoint without being unloaded again.

If the runbook has no checkpoints or the job had not reached the first checkpoint before being unloaded, then it restarts from the beginning.  

When you create a runbook, you should ensure that the time to run any activities between two checkpoints does not exceed three hours. You may need to add checkpoints to your runbook to ensure that it does not reach this three hour limit or break up long running operations. For example, your runbook might perform a reindex on a large SQL database. If this single operation does not complete within the fair share limit, then the job are unloaded and restarted from the beginning. In this case, you should break up the reindex operation into multiple steps, such as reindexing one table at a time, and then insert a checkpoint after each operation so that the job could resume after the last operation to complete.

## Next steps
* To learn more about the different methods that can be used to start a runbook in Azure Automation, see [Starting a runbook in Azure Automation](automation-starting-a-runbook.md)

