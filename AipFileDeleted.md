## AipFileDeleted

Azure Information Protection is a service that allows organizations to classify and label sensitive data, and apply policies to control how that data is accessed and shared. The AipFileDeleted event can be useful for tracking and monitoring the deletion of AIP labeled files, and ensuring that only authorized users are able to delete such files.

AipFileDeleted is a type of event that is recorded in the Office 365 Unified Audit Log. It represents a successful attempt to delete an Azure Information Protection (AIP) labeled file. This event is typically logged when a user with the appropriate permissions uses the AIP client to delete a file that has been labeled with AIP.

Some common attributes of the AipFileDeleted event type in the Unified Audit Log include:

1. CreationTime: The date and time when the event was logged.
2.  UserId: The user who performed the delete action.
3. ClientIP: The IP address of the client that was used to perform the delete action.
4. Operation: The type of operation that was performed, in this case "AipFileDeleted".
5. ResultStatus: The result of the operation, either "Success" or "Failure".
6. ObjectId: The unique identifier of the file that was deleted.

Before connecting to Unified Audit Log - we need to connect with Exchange Online using PowerShell, you can use the following steps:

# Establishing remote Powershell session

This will establish a remote PowerShell session with Exchange Online.Once the connection is established, you can run Exchange Online cmdlets to manage your Exchange Online environment. 

Open a PowerShell window and run the Install-Module -Name ExchangeOnlineManagement command to install the Exchange Online Management module. This module provides cmdlets that can be used to manage Exchange Online.

1. Connect-IPPSSession is a PowerShell cmdlet used to create a remote connection to an Exchange Online PowerShell session.
2. Import-Module ExchangeOnlineManagement is a PowerShell cmdlet used to import the Exchange Online Management module into the current PowerShell session.

```powershell
# Import the PSSSession and Exchange Online cmdlets
Connect-IPPSSession
Import-Module ExchangeOnlineManagement
```

## Option 1 :- If you want to connect with a specific user. 

Command to prompt for a specific user for  your Exchange Online credentials.
```powershell
$UserCredential = Get-Credential 
```
Command to connect to Exchange Online using the provided credentials. 
```powershell
 Connect-ExchangeOnline -Credential $UserCredential -ShowProgress $true 
```
## Option 2 :- If you want to connect with credentials in the current session
Connect to Exchange Online using the credentials in the current session
```powershell
Connect-ExchangeOnline
```
## Finding AipFileDeleted events 
You can use the Search-UnifiedAuditLog PowerShell cmdlet to search for AipFileDeleted events in the Unified Audit Log, and use the Select-Object cmdlet to specify which attributes you want to include in the results.

```powershell
$Operations = "FileDeleted, FileDeletedFirstStageRecycleBin, FileDeletedSecondStageRecycleBin" 
$StartDate = (Get-Date).AddDays(-90); $EndDate = (Get-Date) 
Search-UnifiedAuditLog -Operations $Operations -StartDate $StartDate -EndDate $EndDate -ResultSize 5000 -Formatted
```
### Get the AipFileDeleted events from the last 24 hours
```powershell
$events = Search-UnifiedAuditLog -StartDate (Get-Date).AddHours(-24) -EndDate (Get-Date) -RecordType AipFileDeleted
```
### Extract the information you want from the events
```powershell
$events | Select-Object UserId, FileName, FileType, FilePath, FileDeletionTime
```

### Disconnect from Exchange Online
```powershell
Disconnect-ExchangeOnline
```

## Additional samples

1. This script first sets the date range for the search using the Get-Date cmdlet, and then uses the Search-UnifiedAuditLog cmdlet to search for AipFileDeleted events within that date range. 
2. Next, the script uses the Select-Object cmdlet to extract the relevant information from each event, including the creation time, user ID, client IP, operation type, result status, and file name. 
3. Finally, the script exports the results to a CSV file using the Export-Csv cmdlet.

### Set the date range for the search
```powershell
$startDate = Get-Date -Year 2020 -Month 1 -Day 1
$endDate = Get-Date
```

### Search the Unified Audit Log for AipFileDeleted events in the specified date range
```powershell
$auditEvents = Search-UnifiedAuditLog -StartDate $startDate -EndDate $endDate -Operations AipFileDeleted
```

### Extract the relevant information from each event
```powershell
$results = $auditEvents | Select-Object CreationTime, UserId, ClientIP, Operation, ResultStatus, FileName
```

###  Write the results to a CSV file
```powershell
$results | Export-Csv -Path C:\AipFileDeleted-Audit.csv -NoTypeInformation
```

This is just one example of how the Search-UnifiedAuditLog cmdlet can be used to extract information from the Unified Audit Log. You may need to adjust the script and specify additional parameters based on your specific requirements. For more information about the cmdlets and their parameters, you can refer to the Microsoft documentation or use the PowerShell Get-Help command.

## Managing Large Amounts of Audit Data

To manage large amounts of audit data from the Search-UnifiedAuditLog cmdlet, you can follow these steps:

1. Use the SessionId parameter to identify a search session and specify the number of pages you want to retrieve. This will allow the cmdlet to fetch multiple pages of data and return them to you.

2. The SessionId parameter is used when you want to search for a large amount of audit data using the Search-UnifiedAuditLog cmdlet. The cmdlet will return a maximum of 5000 records per page, so if you want to search for more than that, you will need to use the SessionId parameter to identify a search session and specify the number of pages you want to retrieve. The cmdlet will then use the session identifier to fetch the additional pages of data and return them to you.

3. If you need to search for more than 50,000 records, split the work across multiple searches and use different criteria for each search.

4. Store the results of the searches in an external repository, such as Azure log analytics , Azure Dataexplorer , for easy access and analysis.
 
5. Regularly review and update your search criteria and audit data management processes to ensure that you are capturing the right data and efficiently managing it. This will help you stay on top of any potential security issues and improve the overall security of your organization
 
 
 Search-UnifiedAuditLog has two parameters to support the retrieval of large data sets:
 
 > The **SessionId** parameter holds a string value to identify a search session. You can use any value you like from a simple number to a GUID generated with the New-Guid cmdlet. The presence of a session identifier tells Search-UnifiedAuditLog that it might need to fetch several pages of data.

 > The **SessionCommand** parameter tells Search-UnifiedAuditLog how to handle large amounts of audit data. The returned data might contain duplicate records. This parameter can be set to:
 
 > **ReturnLargeSet** : The audit records returned are unsorted. You can fetch up to 50,000 audit records using this method but must remember to sort the data once it is all fetched.

 > **ReturnNextPreviewPage** : Search-UnifiedAuditLog returns audit records sorted by date. However, you can fetch only a maximum of 5,000 records using this method. If more matching records exist, attempts to fetch the data will result in an er

### SessionId
```powershell
# Import the Exchange Online Management module
Import-Module ExchangeOnlineManagement
# Create a remote connection to Exchange Online
$UserCredential = Get-Credential
Connect-IPPSSession
# Search the Unified Audit Log for entries with the specified SessionID
$SessionId = "5b5a5a5a-5b5b-5c5c-5d5d-5e5e5e5e5e5e"
$StartDate = (Get-Date).AddDays(-90)
$EndDate = (Get-Date).AddDays(1)
[Array]$Records = Search-UnifiedAuditLog -StartDate $StartDate -EndDate $EndDate -SessionId $SessionId

# Scan through the array of records
foreach ($Record in $Records)
{
    # Process each record as needed
    # For example, you could print the operation name and user identity
    Write-Host "Operation: $($Record.RecordType)"
    Write-Host "User Identity: $($Record.Operations)"
    Write-Host "User Identity: $($Record.AuditData)"
}
```
| Event              | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| :----------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| PSComputerName     | Computer name                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| RunspaceId         | The Runspace is a specific instance of PowerShell which contains modifiable collections of commands, providers, variables, functions, and language elements that are available to the command line user.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| PSShowComputerName | The value is false for a documented edited in Office 365.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| RecordType         | Shows the value of Label Action. The operation type indicated by the record. Here we are only listing the relevant MIP Record types. For more information, see the [full list of record types](/office/office-365-management-api/office-365-management-activity-api-schema#auditlogrecordtype).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| CreationTime       | The date and time in Coordinated Universal Time (UTC) in ISO8601 format when the user performed the activity.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| UserId             | The User Principal Name (UPN) of the user who performed the action (specified in the Operation property) that resulted in the record being logged. For example, my_name@my_domain_name. <br><br>Note that records for activity performed by system accounts (such as SHAREPOINT\system or NT AUTHORITY\SYSTEM) are also included. In SharePoint, another value display in the UserId property is app@sharepoint. This indicates that the "user" who performed the activity was an application that has the necessary permissions in SharePoint to perform organization-wide actions (such as search a SharePoint site or OneDrive account) on behalf of a user, admin, or service. For more information, see the [app@sharepoint user in audit records](/microsoft-365/compliance/search-the-audit-log-in-security-and-compliance?view=o365-worldwide#the-appsharepoint-user-in-audit-records). |
| Operation          | The operation type for the audit log. The name of the user or admin activity. For a description of the most common operations/activities.<br> SensitivityLabelApplied<br>SensitivityLabelUpdated<br>SensitivityLabelRemoved<br>SensitivityLabelPolicyMatched<br>SensitivityLabeledFileOpened.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| Identity           | The identity of the user or service to be authenticated.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| ObjectState        | State of the Object after the current event.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| ApplicationId      | The application where the activity happened and displayed in GUID.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| ApplicationName    | Application friendly name of the application performing the operation.Outlook (for email), OWA (for email), Word (for file), Excel (for file), PowerPoint (for file).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| ProcessName        | Process name of the Office application.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| Platform           | The platform on which the activity happened. For example, Windows.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| DeviceName         | Device the event was recorded on.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| ProductVersion     | Version of the Azure Information Protection client that performed the audit action.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| UserId             | The UPN of the user who performed the action (specified in the Operation property) that resulted in the record being logged; for example, my_name@my_domain_name. <br><br>Note that records for activity performed by system accounts (such as SHAREPOINT\system or NT AUTHORITY\SYSTEM) are also included. In SharePoint, another value display in the UserId property is app@sharepoint. This indicates that the "user" who performed the activity was an application that has the necessary permissions in SharePoint to perform organization-wide actions (such as search a SharePoint site or OneDrive account) on behalf of a user, admin, or service. For more information, see the [app@sharepoint user in audit records](/microsoft-365/compliance/search-the-audit-log-in-security-and-compliance?view=o365-worldwide#the-appsharepoint-user-in-audit-records).                       |
| ClientIP           | The IP address of the device that was used when the activity was logged. The IP address is displayed in either an IPv4 or IPv6 address format. For some services, the value displayed in this property might be the IP address for a trusted application (for example, Office on the web apps) calling into the service on behalf of a user and not the IP address of the device used by person who performed the activity. Also, for Azure Active Directory-related events, the IP address isn't logged and the value for the ClientIP property is null. The IP address is displayed in either an IPv4 or IPv6 address format.                                                                                                                                                                                                                                                                 |
| Id                 | GUID of the current record.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| RecordType         | Shows the value of Label Action. The operation type indicated by the record. Here we are only listing the relevant MIP Record types.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| CreationTime       | The date and time in Coordinated Universal Time (UTC) in ISO8601 format when the user performed the activity.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| Operation          | The name of the user or admin activity. For a description of the most common operations/activities, see [Search the audit log in the Office 365 Protection Center](/microsoft-365/compliance/search-the-audit-log-in-security-and-compliance?view=o365-worldwide).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| OrganizationId     | The GUID for your organization's Office 365 tenant. This value will always be the same for your organization, regardless of the Office 365 service in which it occurs.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| UserType           | The type of user that performed the operation. See the UserType table for details on the types of users.</br>0 = Regular</br>1 = Reserved</br>2 = Admin </br>3 = DcAdmin</br>4 = Systeml</br>5 = Application</br>6 = ServicePrincipal</br>7 = CustomPolicy</br>8 = SystemPolicy                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| UserKey            | An alternative ID for the user identified in the UserId property. This property is populated with the passport unique ID (PUID) for events performed by users in SharePoint, OneDrive for Business, and Exchange.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| Workload           | Stores the Office 365 service where the activity occurred.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| Version            | Version of the Azure Information Protection client that performed the audit action                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| Scope              | Specifies scope.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
