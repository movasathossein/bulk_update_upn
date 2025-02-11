# Bulk Update UserPrincipalName for Active Directory Users

This repository contains a PowerShell script designed to update the **UserPrincipalName (UPN)** for a list of users in Active Directory. The script reads user account names (sAMAccountName) from a CSV file and appends a new domain suffix to update their UPNs.

## Why Use This Script?

When internal domain names differ from external ones (e.g., `domain.local` for internal use and `domain.net` for external email addresses), users might encounter login issues. By standardizing UPNs with an external domain, not only do you simplify the login process, but you also align user credentials with email addresses, providing a unified and seamless experience.

## Features

- Bulk processing of Active Directory user accounts.
- Reads input from a CSV file for scalability.
- Updates UPNs to reflect a new domain suffix.
- Includes error handling to ensure issues are reported.
- Supports exporting enabled Active Directory users.

## Prerequisites

1. Ensure you have the **Active Directory PowerShell Module** installed on the system running this script.
2. You need appropriate permissions to update user attributes in Active Directory.
3. The domain suffix (e.g., `domain.net`) should already be added as an alternative UPN suffix in Active Directory Domains and Trusts.

### How to Add an Alternative UPN Suffix in Active Directory

1. Open **Active Directory Domains and Trusts** on a domain controller.
2. Right-click on **Active Directory Domains and Trusts** in the left pane and select **Properties**.
3. In the **UPN Suffixes** tab, click **Add** and enter the new domain suffix (e.g., `domain.net`).
4. Click **OK** to save the changes.

Once the new domain suffix is added to Active Directory, you can proceed with updating the UPNs for users.

## Usage

### Exporting Enabled Users

Run the following PowerShell commands to generate a CSV file containing active users:

```powershell
# Import Active Directory module
Import-Module ActiveDirectory

# Get all enabled users and export to CSV
Get-ADUser -Filter {Enabled -eq $true} -Property SamAccountName | 
    Select-Object SamAccountName |
    Export-Csv -Path "C:\UserData\EnabledUsers.csv" -NoTypeInformation -Encoding UTF8

Write-Host "CSV file created at C:\UserData\EnabledUsers.csv"
```

This will generate a CSV file named `EnabledUsers.csv` at the specified location.

### Input File

Prepare a CSV file with the following format:

```csv
sAMAccountName
user1
user2
user3
```

Save the file at an accessible location, e.g., `C:\UserData\ADUsersList.csv`.

### Script

Modify the script with the correct path to your CSV file and the desired domain suffix:

```powershell
# Path to your CSV file
$csvPath = "C:\UserData\ADUsersList.csv"

# New domain suffix
$newDomain = "domain.net"

# Import the CSV file
$users = Import-Csv -Path $csvPath

# Iterate through each user
foreach ($user in $users) {
    $samAccountName = $user.sAMAccountName
    $newUPN = "$samAccountName@$newDomain"

    # Update the UserPrincipalName
    try {
        Set-ADUser -Identity $samAccountName -UserPrincipalName $newUPN
        Write-Host "Updated: $samAccountName to $newUPN" -ForegroundColor Green
    } catch {
        Write-Host "Failed to update: $samAccountName" -ForegroundColor Red
    }
}
```

Run the script in PowerShell:

```powershell
powershell -ExecutionPolicy Bypass -File .\Update-UPN.ps1
```

### Example Output

```
Updated: user1 to user1@domain.net
Updated: user2 to user2@domain.net
Failed to update: user3
```

## Error Handling

If the script encounters an issue with updating a user, it logs the failure to the console with the relevant username.

## Contributions

Feel free to contribute to this repository by submitting issues or pull requests.

---

**Disclaimer**: Use this script with caution in a production environment. Always test in a controlled setup first.
