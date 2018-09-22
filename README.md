[![Build status](https://ci.appveyor.com/api/projects/status/4jmq6scy093kf5hi/branch/master?svg=true)](https://ci.appveyor.com/project/PowerShell/xjea/branch/master)

# xJea

In the current world of Information Technology, protective measures do not stop at the network edge. Recent news reports based on security breach post-mortems indicate the need to protect assets using measures that reduce administrative access. While the principle of least privilege has always been known to IT Security professionals, there is a need in the industry for a standardized method of constructing an operator experience that reduces access with a more sophisticated level of granularity than what is available in many traditional access control models.

Just Enough Administration (JEA) is a solution designed to help protect Server systems. This is accomplished by allowing specific users to perform administrative tasks on servers without giving them administrator privileges and auditing all actions that these users performed. JEA is based on PowerShell constrained runspaces, a technology that is already being used to secure administrative tasks in environments such as Exchange online.

We are focusing on fortifying your existing environment allowing you to adopt at your own pace a solution that reduces administrator exposure and protects against lateral movement from breached machines

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.


## Contributing
Please check out common DSC Resources [contributing guidelines](https://github.com/PowerShell/DscResource.Kit/blob/master/CONTRIBUTING.md).

## Resources

* **xJeaEndPoint** Describes an administration point for users on the Windows Server
* **xJeaToolKit** Describes a set of authorized tasks that can be accomplished

### xJeaEndPoint

A JEA Endpoint represents an administration point for users on a Windows Server. A JEA Endpoint is composed from:
1. One or more JEA Toolkits (e.g.: “Auditor” toolkit)
2. An Access Control List (ACL) representing the users that can access the JEA Endpoint (e.g.: Auditors_Security_Group) NOTE: these users should not be administrators on the server

* **Name**: Sets a name for the registered endpoint. This will be used by operators to choose which endpoint they should connect to.
* **Toolkit**: The JEA toolkits that should be available from this endpoint.
* **SecurityDescriptorSddl**: An SDDL that defines access for the session.
* **Group**: The machine local group that the account created to host the session should be a member of. If not provided, defaults to ‘Administrators’
* **Ensure**: Defines whether the endpoint should be created or removed.
* **CleanAll**: Boolean value that when set to True will remove all endpoint configurations from the endpoint server.

### xJeaToolKit

A JEA Toolkit represents a set of tasks (e.g.: auditor tasks, SQL admin tasks…) that a user can perform on the server. This is configured through the set of PowerShell cmdlets and parameters that the user can run in order to accomplish that task. For example: an “Auditor” JEA Toolkit will include the “Get-Events” and “Get-Logs” cmdlets. A “SQL admin” JEA Toolkit will include the “Stop-service SQL” and “Start-service SQL” cmdlets and parameters

* **Name**: Sets a name for the toolkit that the registered endpoint will use.
* **CommandSpecs**: A CSV formated list of command specifications (Name,Parameter,ValidateSet,ValidatePattern).
* **ScriptDirectory**: An array of script directories that can be run.
* **Applications**: An array of executables that are allowed to run.
* **Ensure**: Defines whether the toolkit should be created or removed.

## Versions

### Unreleased

* Updated appveyor.yml to use the default template.
* Added unit test template.
* Activate the GitHub App Stale on the GitHub repository
* Added default template files .codecov.yml, .gitattributes, and .gitignore, and
  .vscode folder.

### 0.3.0.0

* Fixing tests

## Examples
### Deploy JEA

Create a JEA endpoint with the toolkit specification hard coded into the configuration.

```
Configuration JEAAuditors
{
    Import-DscResource -module xjea

    xJeaToolkit AuditorToolkit
    {
        Name         = "Auditor toolkit"
        CommandSpecs = @"
Name,Parameter,ValidateSet,ValidatePattern
Get-Process,,,,
Get-Service,,,,
"@
        applications = "ipconfig"
    }

    xJeaEndPoint AuditorEndPoint
    {
        Name      = 'Auditor EndPoint'
        Ensure    = 'Present'
        Toolkit   = 'AuditorToolkit'
        DependsOn = '[xJeaToolkit]AuditorToolkit'
    }
}
```

### Deploy JEA from external CSV

In this example the toolkit is provided to the resource from an external CSV file. This can be helpful as the number of toolkits grow to centrally mange them.

```
Configuration JEAAuditors
{
    Import-DscResource -module xjea

    xJeaToolkit AuditorToolkit
    {
        Name         = "Auditor toolkit"
        CommandSpecs = Get-Content “C:\AuditorToolkit\Toolkit.csv” -Raw
        applications = "ipconfig"
    }

    xJeaEndPoint AuditorEndPoint
    {
        Name      = 'Auditor EndPoint'
        Ensure    = 'Present'
        Toolkit   = 'AuditorToolkit'
        DependsOn = '[xJeaToolkit]AuditorToolkit'
        SecurityDescriptorSddl = 'O:NSG:BAD:P(A;;GX;;;WD)S:P(AU;FA;GA;;;WD)(AU;SA;GXGW;;;WD)'
    }
}
```
### Deploy JEA multiple toolkits from external CSV's

It is also possible to define an endpoint which has multiple toolkits available. This example is an extension of the previous in that it simply adds a second toolkit from a different csv to the configuration.

```
Configuration JEAAuditors
{
    Import-DscResource -module xjea

    xJeaToolkit AuditorToolkit
    {
        Name         = "Auditor toolkit"
        CommandSpecs = Get-Content “C:\AuditorToolkit\Toolkit.csv” -Raw
        applications = "ipconfig"
        Ensure       = 'Present'
    }

    xJeaToolkit Auditor2Toolkit
    {
        Name         = "Auditor 2 toolkit"
        CommandSpecs = Get-Content “C:\AuditorToolkit\Toolkit2.csv” -Raw
        Ensure       = 'Present'
    }

    xJeaEndPoint AuditorEndPoint
    {
        Name      = 'Auditor EndPoint'
        Ensure    = 'Present'
        Toolkit   = 'AuditorToolkit','Auditor2Toolkit'
        DependsOn = '[xJeaToolkit]AuditorToolkit','[xJeaToolkit]Auditor2Toolkit'
        SecurityDescriptorSddl = 'O:NSG:BAD:P(A;;GX;;;WD)S:P(AU;FA;GA;;;WD)(AU;SA;GXGW;;;WD)'
    }
}
```
