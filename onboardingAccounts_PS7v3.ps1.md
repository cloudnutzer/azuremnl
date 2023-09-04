# Set error handling preference to stop on error
$ErrorActionPreference = "Stop"

# Set variables for tenant ID, subscription ID, application name, and custom role file name
$TENANT_ID = "<tenant_id>"
$SUBSCRIPTION_ID = "<subscription_id>"
$APPLICATION_NAME = "PrismaCloudEnterprise"
$CUSTOM_ROLE_FILENAME = "customRolePermission.json"

# Install and import AzureAD PowerShell module
Install-Module AzureAD
Import-Module AzureAD -UseWindowsPowerShell

# Connect to Azure AD using the provided tenant ID
Connect-AzureAD -TenantId $TENANT_ID

# Create a new Azure AD application registration with the provided display name and homepage
$azureADAppReg = New-AzureADApplication -DisplayName $APPLICATION_NAME -AvailableToOtherTenants $false -HomePage 'https://www.paloaltonetworks.com/prisma/cloud'

# Create a new application secret (password) for the application
$appSecret = New-AzureADApplicationPasswordCredential -CustomKeyIdentifier PrimarySecret -ObjectId $azureADAppReg.ObjectId -EndDate ((Get-Date).AddMonths(6))

# Create a new custom role definition using the provided JSON file
$customRoleDef = New-AzRoleDefinition -InputFile $CUSTOM_ROLE_FILENAME

# Create a new service principal for the application
$sp = New-AzureADServicePrincipal -DisplayName $APPLICATION_NAME -AccountEnabled $true -AppId $azureADAppReg.AppId

# Assign the custom role and "Reader" role to the service principal at the subscription scope
New-AzRoleAssignment -ObjectId $sp.ObjectId -RoleDefinitionName $customRoleDef.Name -Scope "/subscriptions/$SUBSCRIPTION_ID"
New-AzRoleAssignment -ObjectId $sp.ObjectId -RoleDefinitionName Reader -Scope "/subscriptions/$SUBSCRIPTION_ID"

# Output various information about the setup to a file named "out.txt"
$azureIDAppRegAppId = $azureADAppReg.AppId
$appSecretValue = $appSecret.Value
$spObjectId = $sp.ObjectId

Write-Output "Directory (Tenant) ID = $TENANT_ID"
Write-Output "Subscription ID = $SUBSCRIPTION_ID"
Write-Output "Application (Client) ID = $azureIDAppRegAppId"
Write-Output "Application Client Secret = $appSecretValue"
Write-Output "Enterprise Application Object ID = $spObjectId"

Write-Output @"
Directory (Tenant) ID = $TENANT_ID
Subscription ID = $SUBSCRIPTION_ID
Application (Client) ID = $azureIDAppRegAppId
Application Client Secret = $appSecretValue
Enterprise Application Object ID = $spObjectId
"@ | Out-File -FilePath "out.txt"
