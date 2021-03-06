# Getting started using SpecFlow

This chapter goes through the setup and the synchronization steps for SpecFlow projects. For non-SpecFlow projects, like Cucumber, please check page [Getting started using Cucumber](getting-started-cucumber.md).

SpecSync is a synchronization tool that can be invoked from the command line. For SpecFlow projects, there is also a SpecFlow plugin that enables synchronizing automated test cases if that is necessary. This guide shows you step-by-step how the synchronization tool and the SpecFlow plugin can be configured.

## Preparation

For setting up SpecSync for Azure DevOps, you need a SpecFlow project and a Azure DevOps project. For the supported Azure DevOps versions, please check the [Compatibility](../compatibility.md) list.

In our guide, we will use a calculator example \(MyCalculator\) that uses SpecFlow v2.3 with MsTest. The SpecFlow project is called `MyCalculator.Specs`. The sample project can be downloaded from [GitHub](https://github.com/gasparnagy/specsync-basic-calculator-specflow).

For a synchronization target we use an Azure DevOps project: `https://specsyncdemo.visualstudio.com/MyCalculator`. \(An Azure DevOps project for testing SpecSync can be created for free from the [Azure DevOps website](https://azure.microsoft.com/en-us/services/devops/)\).

## Installation

The SpecSync for Azure DevOps synchronization tool can be installed by adding the [`SpecSync.AzureDevOps`](https://www.nuget.org/packages/SpecSync.AzureDevOps) package from NuGet.org:

```text
PM> Install-Package SpecSync.AzureDevOps
```

The package contains the synchronization command line tool \(`tools\SpecSync4AzureDevOps.exe`\) and some documentation files \(`docs` folder\).

It also adds a `specsync4azuredevops.cmd` script file to the project for calling the SpecSync command line tool conveniently. SpecSync can also be used without the script file, but in this case you have to provide the full path of `SpecSync4AzureDevOps.exe` downloaded into the NuGet packages folder.

## Basic configuration

The NuGet package has added a configuration file \(`specsync.json`\) to your project that contains all SpecSync related settings. Before the first synchronization we have to review and change a few settings in this file.

1. Open the `specsync.json` file in Visual Studio from your project folder.
2. Set the value of the `remote/projectUrl` setting to the **project URL** of your Azure DevOps project. The project URL is usually in `https://server-name/project-name` or in `http://server-name:8080/tfs/project-name` form and it is not necessarily the URL of the dashboard you open normally. See [What is my Azure DevOps project URL](../important-concepts/what-is-my-tfs-project-url.md) for more details.
3. Optionally you can set your [personal access token](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=vsts) \(PAT\) as user name \(`remote/user` setting\) or choose one of the other [Azure DevOps authentication options](../important-concepts/tfs-authentication-options.md). If you don't specify credentials here, SpecSync will show an interactive authentication prompt.

The `specsync.json` after basic configuration has been set

```javascript
{
  "$schema": "http://schemas.specsolutions.eu/specsync4azuredevops-config-latest.json",

  // See configuration options and samples at http://speclink.me/specsyncconfig.
  // You can also check the 'specsync-sample.json' file in the 'docs' folder of the NuGet package.

  "remote": {
    "projectUrl": "https://specsyncdemo.visualstudio.com/MyCalculator",
    "user": "52yny........................................ycsetda"
  }
}
```

## First synchronization

1. Make sure your project compiles. 
2. We recommend starting from a state where 
   * all tests pass,
   * the modified files are checked in to source control.
3. Open a command line prompt and navigate to the SpecFlow project folder \(`MyCalculator.Specs`\)
4. Call `specsync4azuredevops.cmd push` to invoke the synchronization.
5. If you haven't specified any credentials in the configuration file, an authentication dialog will popup, where you have to specify your credentials for accessing the Azure DevOps project.

As a result, the scenarios from the project will be linked to newly created Azure DevOps test cases, and you will see a result like this.

![Result of the first synchronization](../.gitbook/assets/getting-started-specflow-first-sync.png)

_Note: Scenarios are synchronized to normal, Scenario Outlines to parametrized test cases._

_Useful hint for testing:_ Normally you cannot delete work items from Azure DevOps, so testing the initial linking is harder. If you have Visual Studio installed, there is a tool called `witadmin` available from the VS command prompt. With the `destroywi` command of this tool you can delete work items. See `witadmin help destroywi` for details, and use it carefully.

## Check Test Case in Azure DevOps

1. Find one of the created test case in Azure DevOps. The easiest way to do this is to open the Azure DevOps URL in a browser and specify the test case ID \(e.g. `#12294)` in the "Search" text box in the upper right corner of the web page.

You should see something like this.

![A newly created test case in Azure DevOps](../.gitbook/assets/getting-started-specflow-new-test-case.png)

There are a couple of things you can note here.

* The name of the scenario has been synchronized as the title of the test case. \(The "Scenario:" prefix can be omitted by changing the [synchronization format configurations](../configuration/configuration-synchronization/configuration-synchronization-format.md).\)
* The tags of the scenario have been synchronized as test case tags.
* The steps of the scenario have been synchronized as test case steps. \(The _Then_ steps can also be synchronized into the _Expected result_ column of the test case step list and you can [change a couple of other formatting options](../configuration/configuration-synchronization/configuration-synchronization-format.md) as well.\)

## Verify feature file and commit changes

1. Open one of the feature files from the SpecFlow project in Visual Studio. SpecSync modified the file and added a few tags.
2. Each scenario and scenario outline has been tagged with a `@tc:...` tag making the link between the scenario and the created test case.

```text
@tc:12294
@important
Scenario: Add two positive numbers
```

_Note: The feature files are changed only when synchronizing new scenarios \(linking\). To avoid file changes \(e.g. when running the synchronization from a build server\) the_ `--buildServerMode` _command line switch can be used. See_ [_Synchronizing test cases from build_](../important-concepts/synchronizing-test-cases-from-build.md) _for details._

Verify if the project still compiles and the tests pass \(they should, since we have only added tags\), and commit \(check-in\) your changes.

## Synchronize an update

Now let's make a change in one of the scenarios and synchronize the changes to the related test case.

1. Update the title and the steps of the scenario, for example change the scenario `Add two positive numbers` to `Multiply two positive numbers`, change `add` to `multiply` in the _When_ step and update the expected result to `377`:

   ```text
   @tc:12294
   Scenario: Multiply two positive numbers
     Given I have entered the following numbers
        | number |
        | 29     |
        | 13     |
     When I choose multiply
     Then the result should be 377
   ```

2. Make sure it still compiles and the test passes.
3. Run the synchronization again:

   ```text
   specsync4azuredevops.cmd push
   ```

The result shows that the test case for the scenario has been updated, but the other test cases have remained unchanged \(_up-to-date_\).

![Result of synchronizing an update](../.gitbook/assets/getting-started-specflow-sync-updates.png)

1. Refresh the test case in your browser to see the changed title and steps.

   ![Updated test case in Azure DevOps](../.gitbook/assets/getting-started-specflow-updated-test-case.png)

_Note: For executing complex test cases, further verification and planning steps might be required after the test case has been changed. SpecSync can reset the test case state to a configured value \(e.g._ `Design`_\) in order to ensure that these steps are not forgotten. For more information on this, check the_ [_synchronization state configuration_](../configuration/configuration-synchronization/configuration-synchronization-state.md) _documentation._

## Group synchronized test cases to a test suite

We have seen already how to synchronize scenarios to test cases. To be able to easily find these test cases in Azure DevOps, they can be added to test suites. SpecSync can automatically add/remove the synchronized test cases to a test suite. For this you have to specify the name or the ID or the name of the test suite in the configuration.

1. Create a "Static suite" \(e.g. "BDD Scenarios"\) in Azure DevOps. \(For that you have to navigate to "Test plans" and create and select a test plan first.\)
2. Specify the name of the test suite in the `remote/testSuite/name` entry of the `specsync.json` file. \(Alternatively you can specify the ID of the suite in `remote/testSuite/id`. The suite names are not unique in Azure DevOps!\)

   ```javascript
   {
     "$schema": "http://schemas.specsolutions.eu/specsync4azuredevops-config-latest.json",

     // See configuration options and samples at http://speclink.me/specsyncconfig.
     // You can also check the 'specsync-sample.json' file in the 'docs' folder of the NuGet package.

     "remote": {
       "projectUrl": "https://specsyncdemo.visualstudio.com/MyCalculator",
       "user": "52yny4a......................................ycsetda",
       "testSuite": {
         "name": "BDD Scenarios"
       }
     }
   }
   ```

3. Make sure that the project compiles and the tests pass.
4. Run the synchronization again:

   ```text
   specsync4azuredevops.cmd push
   ```

The synchronization will proceed with the result similar to this.

![Synchronize scenarios to test suite](../.gitbook/assets/getting-started-specflow-sync-test-suite.png)

SpecSync has added the test cases to the test suite.

![Test cases were added to the test suite](../.gitbook/assets/getting-started-specflow-updated-test-suite.png)

_Note: Since the test suite names are not unique in Azure DevOps, you can also specify the test suite ID in the_ `remote/testSuite/id` _setting._

## Synchronizing automated test cases \(optional\)

So far the test cases we synchronized from the scenarios were marked as "Not Automated". This means that although it is possible to execute the SpecFlow scenarios both locally and on the build server \(from the assembly built from the SpecFlow project\), the synchronized tests cases were not linked to the test method generated by SpecFlow.

If the team needs the Azure DevOps test cases for documentation and traceability and runs the scenarios from assembly, then we have already reached the desired outcome. But if the test cases have to be executed as automated test cases, we need to perform a few further steps and you have to choose a test execution strategy.

For more information on test execution strategies and a step-by-step instruction on how they can be configured, please check the [Synchronizing automated test cases](../important-concepts/synchronizing-automated-test-cases.md) article.

