# Azure Automation Runbook for Object Replication

This tutorial will guide you through the process of setting up an Azure Automation Runbook for object replication between two storage accounts.

## Prerequisites

- Active subscription.
- Two Azure Storage accounts.
- Contributor permissions.

## Introduction

This PowerShell script is used to set up object replication between two Azure Storage accounts located in different regions. Object replication is a feature of Azure Storage that allows you to replicate blobs from a source container to a destination container within or across storage accounts.

Here's a brief overview of what the script does:

1. **Imports the Azure Storage module**: This module contains the cmdlets required to interact with Azure Storage.

2. **Connects to your Azure account**: The script uses a managed identity to authenticate to Azure.

3. **Defines the source and destination storage accounts and containers**: These are the storage accounts and containers between which the objects will be replicated.

4. **Gets the current date and time in UTC**: This is used to set the minimum creation time for the object replication policy rule.

5. **Retrieves the storage accounts**: The script retrieves the details of the source and destination storage accounts.

6. **Creates a new object replication policy rule**: This rule specifies that blobs created in the source container after a certain time should be replicated to the destination container.

7. **Sets the object replication policy**: The script applies the object replication policy rule to the source and destination storage accounts.

8. **Waits for 20 minutes**: This is to allow time for the object replication policy to take effect. You need to carefully consider how long time the script should wait before the objects have completely replicated over to the destination account.

9. **Deletes the object replication policy**: After the replication has taken place, the script deletes the object replication policy.

This script is a basic example of how to set up object replication between two Azure Storage accounts. Depending on your specific requirements, you may need to modify the script to suit your needs.

## Step 1: Create an Automation Account and a Runbook

1. In the Azure portal, search for and select **Automation Accounts**.
2. Click on **Add** to create a new Automation Account.
3. Fill in the details for your new Automation Account and click **Create**.
4. Once your Automation Account is created, go to it and select **Runbooks** under **Process Automation**.
5. Click on **Create a runbook**, give it a name, select **PowerShell** as the runbook type, and set Version to **7.2 (Recommended)** and click **Create**.

## Step 2: Enable Managed Identity

1. In your Automation Account, go to **Identity** under **Account Settings**.
2. Switch the **Status** of **System assigned** to **On** and click **Save**.

## Step 3: Assign Managed Identity to the Storage Accounts

1. Go to each of your Storage Accounts.
2. Select **Access control (IAM)**.
3. Click on **Add** > **Add role assignment**.
4. In the **Add role assignment** pane, select **Storage Account Contributor** as the role.
5. In the **Assign access to** dropdown, select **Managed Identity**.
6. In the **Select** field, choose the name of your Automation Account, and click **Save**.

## Step 4: Add the Code to the Runbook

1. Go back to your Runbook in the Automation Account.
2. Click on **Edit**.
3. Paste the provided PowerShell script into the editor.
4. Click on **Save**.

```powershell
# Import the module
Import-Module Az.Storage
Write-Output "Module imported"

# Connect to your Azure account
Connect-AzAccount -Identity
Write-Output "Connected to Azure account"

# Source and destination storage account names
$srcStorageAccountName = "storwestacc"
$destStorageAccountName = "storeastacc"

# Resource group names
$srcResourceGroupName = "storage-west-rg"
$destResourceGroupName = "storage-east-rg"

# Container names
$srcContainerName = "data"
$destContainerName = "data"


# The script is working with UTC times to avoid issues with time zone differences.

# Let's Get the current date and time in UTC
$currentDateTime = Get-Date
$currentDateTimeUtc = $currentDateTime.ToUniversalTime()

# Format the current date and time
$formattedCurrentDateTime = $currentDateTimeUtc.ToString("yyyy-MM-ddTHH:mm:ssZ")

Write-Output $formattedCurrentDateTime

# Get the date and time 24 hours ago in UTC
$dateTime24HoursAgoUtc = $currentDateTimeUtc.AddHours(-24)

# Format the date and time 24 hours ago
$formattedDateTime24HoursAgo = $dateTime24HoursAgoUtc.ToString("yyyy-MM-ddTHH:mm:ssZ")

Write-Output $formattedDateTime24HoursAgo

# Get the storage accounts
$srcStorageAccount = Get-AzStorageAccount -ResourceGroupName $srcResourceGroupName -Name $srcStorageAccountName
$destStorageAccount = Get-AzStorageAccount -ResourceGroupName $destResourceGroupName -Name $destStorageAccountName
Write-Output "Storage accounts retrieved"

# Create a new object replication policy rule, and set the minimum creation time to 24 hours ago,blobs created after this time will be replicated.
$rule = New-AzStorageObjectReplicationPolicyRule -SourceContainer $srcContainerName -DestinationContainer $destContainerName -MinCreationTime $formattedDateTime24HoursAgo

# Get the source storage account
$srcStorageAccount = Get-AzStorageAccount -ResourceGroupName $srcResourceGroupName -Name $srcStorageAccountName

# Set the object replication policy
Set-AzStorageObjectReplicationPolicy -ResourceGroupName $destResourceGroupName -AccountName $destStorageAccountName -PolicyId default -SourceAccount $srcStorageAccount.Id -Rule $rule

#Get Object Replication Policy ID
$policy = Get-AzStorageObjectReplicationPolicy -ResourceGroupName $destResourceGroupName -AccountName $destStorageAccountName

Set-AzStorageObjectReplicationPolicy -ResourceGroupName $srcResourceGroupName -AccountName $srcStorageAccountName -InputObject $policy

Write-Output "Object replication policy rule applied"

# Wait for 20 minute
Start-Sleep -Seconds 1200

#Get Object Replication Policy ID
$policy = Get-AzStorageObjectReplicationPolicy -ResourceGroupName $destResourceGroupName -AccountName $destStorageAccountName

# Delete the Object Replication Policy
Remove-AzStorageObjectReplicationPolicy -ResourceGroupName $destResourceGroupName -AccountName $destStorageAccountName -PolicyId $policy.PolicyId
Remove-AzStorageObjectReplicationPolicy -ResourceGroupName $srcResourceGroupName -AccountName $srcStorageAccountName -PolicyId $policy.PolicyId

Write-Output "Object replication policy rule deleted"

```

## Step 5: Link a Schedule to Your Runbook

1. In your Runbook, click on **Link to Schedule**.
2. Click on **Link a schedule to your runbook**.
3. Click on **Create a new schedule**, fill in the details for your schedule, and click **Create**.
4. Click **OK** to link the schedule to your runbook.

Now, your runbook will automatically run according to the schedule, replicating objects between your two storage accounts.
## 
