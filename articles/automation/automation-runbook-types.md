---
title: Azure Automation runbook types
description: This article describes the types of runbooks that you can use in Azure Automation and considerations for determining which type to use.
services: automation
ms.subservice: process-automation
ms.date: 11/17/2021
ms.topic: conceptual 
ms.custom: devx-track-azurepowershell
---

# Azure Automation runbook types

The Azure Automation Process Automation feature supports several types of runbooks, as defined in the following table. To learn about the process automation environment, see [Runbook execution in Azure Automation](automation-runbook-execution.md).

| Type | Description |
|:--- |:--- |
| [Graphical](#graphical-runbooks)|Graphical runbook based on Windows PowerShell and created and edited completely in the graphical editor in Azure portal. |
| [Graphical PowerShell Workflow](#graphical-runbooks)|Graphical runbook based on Windows PowerShell Workflow and created and edited completely in the graphical editor in Azure portal. |
| [PowerShell](#powershell-runbooks) |Textual runbook based on Windows PowerShell scripting. |
| [PowerShell Workflow](#powershell-workflow-runbooks)|Textual runbook based on Windows PowerShell Workflow scripting. |
| [Python](#python-runbooks) |Textual runbook based on Python scripting. |

Take into account the following considerations when determining which type to use for a particular runbook.

* You can't convert runbooks from graphical to text type, or the other way around.
* There are limitations when using runbooks of different types as child runbooks. For more information, see [Child runbooks in Azure Automation](automation-child-runbooks.md).

## Graphical runbooks

You can create and edit graphical and graphical PowerShell Workflow runbooks using the graphical editor in the Azure portal. However, you can't create or edit this type of runbook with another tool. Main features of graphical runbooks:

* Exported to files in your Automation account and then imported into another Automation account.
* Generate PowerShell code.
* Converted to or from graphical PowerShell Workflow runbooks during import.

### Advantages

* Use visual insert-link-configure authoring model.
* Focus on how data flows through the process.
* Visually represent management processes.
* Include other runbooks as child runbooks to create high-level workflows.
* Encourage modular programming.

### Limitations

* Can't create or edit outside the Azure portal.
* Might require a code activity containing PowerShell code to execute complex logic.
* Can't convert to one of the [text formats](automation-runbook-types.md), nor can you convert a text runbook to graphical format. 
* Can't view or directly edit PowerShell code that the graphical workflow creates. You can view the code you create in any code activities.
* Can't run runbooks on a Linux Hybrid Runbook Worker. See [Automate resources in your datacenter or cloud by using Hybrid Runbook Worker](automation-hybrid-runbook-worker.md).
* Graphical runbooks can't be digitally signed.

## PowerShell runbooks

PowerShell runbooks are based on Windows PowerShell. You directly edit the code of the runbook using the text editor in the Azure portal. You can also use any offline text editor and [import the runbook](manage-runbooks.md) into Azure Automation.


The PowerShell version is determined by the **Runtime version** specified (that is version 7.1 preview or 5.1). The Azure Automation service supports the latest PowerShell runtime.
 
The same Azure sandbox and Hybrid Runbook Worker can execute **PowerShell 5.1** and **PowerShell 7.1** runbooks side by side.   

> [!NOTE]
>  At the time of runbook execution, if you select **Runtime Version** as **7.1 (preview)**, PowerShell modules targeting 7.1 runtime version is used and if you select **Runtime Version** as **5.1**, PowerShell modules targeting 5.1 runtime version are used.
 
Ensure that you select the right Runtime Version for modules.

For example : if you are executing a runbook for a Sharepoint automation scenario in **Runtime version** *7.1 (preview)*, then import the module in **Runtime version** **7.1 (preview)**; if you are executing a runbook for a Sharepoint automation scenario in **Runtime version** **5.1**, then import the module in **Runtime version** *5.1*. In this case, you would see two entries for the module, one for **Runtime Version** **7.1(preview)** and other for **5.1**.

:::image type="content" source="./media/automation-runbook-types/runbook-types.png" alt-text="runbook Types.":::


Currently, PowerShell 5.1 and 7.1 (preview) are supported.


### Advantages

* Implement all complex logic with PowerShell code without the other complexities of PowerShell Workflow.
* Start faster than PowerShell Workflow runbooks, since they don't need to be compiled before running.
* Run in Azure and on Hybrid Runbook Workers for both Windows and Linux.

### Limitations - version 5.1

* You must be familiar with PowerShell scripting.
* Runbooks can't use [parallel processing](automation-powershell-workflow.md#use-parallel-processing) to execute multiple actions in parallel.
* Runbooks can't use [checkpoints](automation-powershell-workflow.md#use-checkpoints-in-a-workflow) to resume runbook if there's an error.
* You can include only PowerShell, PowerShell Workflow runbooks, and graphical runbooks as child runbooks by using the [Start-AzAutomationRunbook](/powershell/module/az.automation/start-azautomationrunbook) cmdlet, which creates a new job.
* Runbooks can't use the PowerShell [#Requires](/powershell/module/microsoft.powershell.core/about/about_requires) statement, it is not supported in Azure sandbox or on Hybrid Runbook Workers and might cause the job to fail.

### Known issues - version 5.1

The following are current known issues with PowerShell runbooks:

* PowerShell runbooks can't retrieve an unencrypted [variable asset](./shared-resources/variables.md) with a null value.
* PowerShell runbooks can't retrieve a variable asset with `*~*` in the name.
* A [Get-Process](/powershell/module/microsoft.powershell.management/get-process) operation in a loop in a PowerShell runbook can crash after about 80 iterations.
* A PowerShell runbook can fail if it tries to write a large amount of data to the output stream at once. You can typically work around this issue by having the runbook output just the information needed  to work with large objects. For example, instead of using `Get-Process` with no limitations, you can have the cmdlet output just the required parameters as in `Get-Process | Select ProcessName, CPU`.

### Limitations - 7.1 (preview)

-  The Azure Automation internal PowerShell cmdlets are not supported on a Linux Hybrid Runbook Worker. You must import the `automationassets` module at the beginning of your Python runbook to access the Automation account shared resources (assets) functions. 
-  For the PowerShell 7 runtime version, the module activities are not extracted for the imported modules.
-  *PSCredential* runbook parameter type is not supported in PowerShell 7 runtime version.
-  PowerShell 7.x does not support workflows. See [this](/powershell/scripting/whats-new/differences-from-windows-powershell?view=powershell-7.1#powershell-workflow&preserve-view=true) for more details.
-  PowerShell 7.x currently does not support signed runbooks.
-  Source control integration doesn't support PowerShell 7.1. Also, PowerShell 7.1 runbooks in source control gets created in Automation account as Runtime 5.1.

### Known Issues - 7.1 (preview)

- Executing child scripts using `.\child-runbook.ps1` is not supported in this preview. 
  **Workaround**: Use `Start-AutomationRunbook` (internal cmdlet) or `Start-AzAutomationRunbook` (from *Az.Automation* module) to start another runbook from parent runbook.
- Runbook properties defining logging preference is not supported in PowerShell 7 runtime.  
  **Workaround**: Explicitly set the preference at the start of the runbook as below -
  ```
      $VerbosePreference = "Continue"

      $ProgressPreference = "Continue"
  ```
- Avoid importing `Az.Accounts` module to version 2.4.0 version for PowerShell 7 runtime as there can be an unexpected behavior using this version in Azure Automation. 
- You might encounter formatting problems with error output streams for the job running in PowerShell 7 runtime.
- When you import a PowerShell 7.1 module that’s dependent on other modules, you may find that the import button is gray even when PowerShell 7.1 version of the dependent module is installed. For example, Az.Compute version 4.20.0, has a dependency on Az.Accounts being >= 2.6.0. This issue occurs when an equivalent dependent module in PowerShell 5.1 doesn't meet the version requirements. For example, 5.1 version of Az.Accounts was < 2.6.0.
- When you start PowerShell 7 runbook using the webhook, it auto-converts the webhook input parameter to an invalid JSON.

## PowerShell Workflow runbooks

PowerShell Workflow runbooks are text runbooks based on [Windows PowerShell Workflow](automation-powershell-workflow.md). You directly edit the code of the runbook using the text editor in the Azure portal. You can also use any offline text editor and [import the runbook](manage-runbooks.md) into Azure Automation. 

>[!NOTE]
> PowerShell 7.1 does not support workflow runbooks.

### Advantages

* Implement all complex logic with PowerShell Workflow code.
* Use [checkpoints](automation-powershell-workflow.md#use-checkpoints-in-a-workflow) to resume operation if there's an error.
* Use [parallel processing](automation-powershell-workflow.md#use-parallel-processing) to do multiple actions in parallel.
* Can include other graphical runbooks and PowerShell Workflow runbooks as child runbooks to create high-level workflows.

### Limitations

* You must be familiar with PowerShell Workflow.
* Runbooks must deal with the additional complexity of PowerShell Workflow, such as [deserialized objects](automation-powershell-workflow.md#deserialized-objects).
* Runbooks take longer to start than PowerShell runbooks since they must be compiled before running.
* You can only include PowerShell runbooks as child runbooks by using the `Start-AzAutomationRunbook` cmdlet.
* Runbooks can't run on a Linux Hybrid Runbook Worker.

## Python runbooks

Python runbooks compile under Python 2 and Python 3. Python 3 runbooks are currently in preview. You can directly edit the code of the runbook using the text editor in the Azure portal. You can also use an offline text editor and [import the runbook](manage-runbooks.md) into Azure Automation.

Python 3 runbooks are supported in the following Azure global infrastructures:

* Azure global
* Azure Government

### Advantages

* Use the robust Python libraries.
* Can run in Azure or on Hybrid Runbook Workers.
* For Python 2, Windows Hybrid Runbook Workers are supported with [python 2.7](https://www.python.org/downloads/release/latest/python2) installed.
* For Python 3 Cloud Jobs, Python 3.8 version is supported. Scripts and packages from any 3.x version might work if the code is compatible across different versions.  
* For Python 3 Hybrid jobs on Windows machines, you can choose to install any 3.x version you may want to use.  
* For Python 3 Hybrid jobs on Linux machines, we depend on the Python 3 version installed on the machine to run DSC OMSConfig and the Linux Hybrid Worker. We recommend installing 3.6 on Linux machines. However, different versions should also work if there are no breaking changes in method signatures or contracts between versions of Python 3.

### Limitations

* You must be familiar with Python scripting.
* To use third-party libraries, you must [import the packages](python-packages.md) into the Automation account.
* Using **Start-AutomationRunbook** cmdlet in PowerShell/PowerShell Workflow to start a Python 3 runbook (preview) doesn't work. You can use **Start-AzAutomationRunbook** cmdlet from Az.Automation module or **Start-AzureRmAutomationRunbook** cmdlet from AzureRm.Automation module to work around this limitation.  
* Azure Automation doesn't support **sys.stderr**.
* The Python **automationassets** package is not available on pypi.org, so it's not available for import onto a Windows machine.

### Multiple Python versions

For a Windows Runbook Worker, when running a Python 2 runbook it looks for the environment variable `PYTHON_2_PATH` first and validates whether it points to a valid executable file. For example, if the installation folder is `C:\Python2`, it would check if `C:\Python2\python.exe` is a valid path. If not found, then it looks for the `PATH` environment variable to do a similar check.

For Python 3, it looks for the `PYTHON_3_PATH` env variable first and then falls back to the `PATH` environment variable.

When using only one version of Python, you can add the installation path to the `PATH` variable. If you want to use both versions on the Runbook Worker, set `PYTHON_2_PATH` and `PYTHON_3_PATH` to the location of the module for those versions.

### Known issues

For cloud jobs, Python 3 jobs sometimes fail with an exception message `invalid interpreter executable path`. You might see this exception if the job is delayed, starting more than 10 minutes, or using **Start-AutomationRunbook** to start Python 3 runbooks. If the job is delayed, restarting the runbook should be sufficient. Hybrid jobs should work without any issue if using the following steps:

1. Create a new environment variable called `PYTHON_3_PATH` and specify the installation folder. For example, if the installation folder is `C:\Python3`, then this path needs to be added to the variable.
1. Restart the machine after setting the environment variable.

## Next steps

* To learn about PowerShell runbooks, see [Tutorial: Create a PowerShell runbook](./learn/powershell-runbook-managed-identity.md).
* To learn about PowerShell Workflow runbooks, see [Tutorial: Create a PowerShell Workflow runbook](learn/automation-tutorial-runbook-textual.md).
* To learn about graphical runbooks, see [Tutorial: Create a graphical runbook](./learn/powershell-runbook-managed-identity.md).
* To learn about Python runbooks, see [Tutorial: Create a Python runbook](./learn/automation-tutorial-runbook-textual-python-3.md).
