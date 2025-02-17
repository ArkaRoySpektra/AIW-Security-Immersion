# Module 5 - Improving your Secure Posture


## Overview

This module will walk you through the process to use the vulnerability assessment for Virtual Machines and Containers, along with the usage of Workflow Automation and data query.

### Exercise 1: Vulnerability assessment for VMs

With Azure Defender for servers, you can quickly deploy the integrated vulnerability assessment solution (powered by Qualys) with no additional configuration or extra costs. Once the vulnerability assessment scanner is deployed, it continually assesses all the installed applications on a virtual machine to find vulnerabilities and presents its findings in the Azure Security Center console. When a machine is found that doesn't have a vulnerability assessment solution deployed, Azure Security Center generates a recommendation that looks like this: _A vulnerability assessment solution should be enabled on your virtual machines._ To remediate a resource, you can click on the **Quick Fix** button to deploy the necessary VM extension.

**Explore vulnerability assessment recommendations:**

1.	Launch **Azure Portal** using the desktop icon on the **JumpVM** and login with the Azure credentials from the Lab **Environment Details** tab, if you are not logged in already.

2.	Type **Security Center** in the search box located on the top of the **Azure Portal** page and click on it. Next, click on **Recommendations** from the left sidebar.

3.	Go to bottom of page and expand **Remediate vulnerabilities** security control (which contains all recommendations related to security vulnerabilities).

4.	Make sure you have **A vulnerability assessment solution should be enabled on your virtual machines** recommendation listed here. If you don’t, you will need **24 hours** to have the recommendation with the assessment.

![](../Images/remediate-blade.png)

5.	Click on **A vulnerability assessment solution should be enabled on your virtual machines** recommendation and open it.

6.	Click to expand **Remediation steps** – then click on **View recommendation logic** option to expose an automatic remediation script content (ARM template). Once done, **Close** this window.

![](../Images/remediation-logic.png)

7.	From the unhealthy tab, select both *asclab-win* and *aslab-linux* virtual machines. Click on  **Remediate**.

![](../Images/remediate-asclab-win.png)

8.	On the **Choose a vulnerability assessment solution** select **Recommended: Deploy ASC integrated vulnerability scanner powered by Qualys (included in Azure Defender for servers)**. Click on  **Proceed**.

![](../Images/proceed.png)

9.	A window will open, on this page review the list of VMs and click the **Remediate 2 resource** button.

10.	Remediation is now in process. Azure Security Center will deploy the Qualys VM extension on the selected VMs, so you track the status using the notification area or by using Azure activity log. Wait for 5-10 minutes for the process to complete.

> Note: You can find a list of supported operating systems [here](https://docs.microsoft.com/en-us/azure/security-center/deploy-vulnerability-assessment-vm#deploy-the-integrated-scanner-to-your-azure-and-hybrid-machines).

10.	Ensure the VM extension is deployed on the relevant machines:
    - Type **Virtual Machines** in the search box located on the top of the **Azure Portal** page and click on it.
    - Select **asclab-win**.
    - From the sidebar, click on **Extensions**.
    - Make sure to have `WindowsAgent.AzureSecurityCenter` extension installed and successfully provisioned.
    - Repeat the process for **asclab-linux** – you should expect to see a different name for the extension on Linux platform: `LinuxAgent.AzureSecurityCenter`.

![](../Images/win-ext.png)


![](../Images/linux-ext.png)


> Note: There are multiple ways you can automate the process where you need to achieve at scale deployment. More details are available on our [documentation](https://docs.microsoft.com/en-us/azure/security-center/deploy-vulnerability-assessment-vm#automate-at-scale-deployments) and on [blog](https://techcommunity.microsoft.com/t5/azure-security-center/built-in-vulnerability-assessment-for-vms-in-azure-security/ba-p/1577947).

11.	The VA agent will now collect all required artifacts, send them to Qualys Cloud and findings will be presented back on ASC console within 24 hours.

### Exercise 2: Vulnerability assessment for Containers

Azure Security Center scans images in your Azure Container Registry (ACR) that are pushed and imported into the registry, it also contains any other images pulled within the last 30 days. Then, it exposes detailed findings per image. All vulnerabilities can be found in the following recommendation: **Vulnerabilities in Azure Container Registry images should be remediated (powered by Qualys).**

To simulate a container registry image with vulnerabilities, we will use ACR tasks commands and sample image:

1. Type **Container registries** in the search box located on the top of the **Azure Portal** page and click on it, or click [here](https://portal.azure.com/#blade/HubsExtension/BrowseResource/resourceType/Microsoft.ContainerRegistry%2Fregistries).

2. Copy the name or your container registry, for example: *asclabcrktfvrxcne4kki*

![acr-select](../Images/acr-select.png)

3. In the Azure portal, open Cloud Shell pane by selecting on the toolbar icon directly to the right of the search textbox or click on [Azure Cloud Shell](https://shell.azure.com/), If prompted to select either Bash or PowerShell, select Bash.

4. When prompted, click **Create storage**, and wait for the Azure Cloud Shell to initialize. 

5.	Build a Linux container image from the hello-world image hosted at Microsoft Container Registry and push it to the existing Azure Container Registry instance on your subscription:

Run the the following two script blocks:

```
echo FROM mcr.microsoft.com/azuredocs/aci-helloworld > Dockerfile
```

Modify the following script to include your container registry name:

```
az acr build --image sample/hello-world:v1 --registry <your container registry name> --file Dockerfile .
```

![Build Linux container in Cloud Shell](../Images/asc-build-linux-container-cloud-shell.gif?raw=true)

6. Wait for a successful execution message to appear. For example: Run ID: cb1 was successful after 23s

7.	The scan completes typically within few minutes, but it might take up to 15 minutes for the vulnerabilities/security findings to appear on the Recommendations page.

8.	Type **Security Center** in the search box located on the top of the **Azure Portal** page and click on it, then click on **Recommendations** from the left sidebar.

9.	Expand **Remediate vulnerabilities** security control and select **Vulnerabilities in Azure Container Registry images should be remediated (powered by Qualys)**.

![](../Images/acr.png)

10.	On the recommendation page, notice the following details at the upper section:

    - Unhealthy registries: *1/1*
    - Severity of recommendation: *High*
    - Total vulnerabilities: *expect to see more than 2 vulnerabilities*

![](../Images/acr2.png)

11.	Expand the **Affected resources** section and notice the **Unhealthy registries** count which shows **1 container registry** (asclab**xxx** here xxx is unique ID).

12.	On the **Security Checks** section, notice the number of vulnerabilities.

13.	Click on the first security check to open the right pane.

![](../Images/acr3.png)

Notice the vulnerability description, general information (containing the Cvss 2.0 base score, etc.), remediation steps/workaround, additional information, and the affected (vulnerable) image. **Close this window.**

### Exercise 3: Automate recommendations with workflow automation

Every security program includes multiple workflows for incident response. The process might include notifying relevant stakeholders, launching a change management process, and applying specific remediation steps. With the help of workflow automation, you can trigger logic apps to automate processes in real-time with Security Center events (security alerts or recommendations). In this lab, you will create a new Logic App and then trigger it automatically using workflow automation feature when there is a change with a specific recommendation.

**Create a new Logic App:**

1.	Type **Logic Apps** in the search box located on the top of the **Azure Portal** page and click on it, or [click here](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResource/resourceType/Microsoft.Logic%2Fworkflows).

2.	Click **Add** and **Consumption** to create a new Logic App.

3.	On the Basics tab, select your subscription and resource group **asclab**.

4.	On the Logic app name field enter *Send-RecommendationsChanges*.

5.	Select location, for example: **East US** (it’s recommended to use the same region as used in the previous exercises).

6.	Keep Log Analytics option as **Off**.

7.	Select **Review + Creation** and then click on **Create**.

8.	When the **Deployment** is completed click on **Go to resource**

![](../Images/logic-app.png)

8.	After the Logic Apps Designer opens, select **Blank Logic App**.

![](../Images/open-logic-app.png)

9.	Type **Security Center** in the search box and select **When an Azure Security Center Recommendation is created or triggered**.

![](../Images/triggered.png)

10.	Click on the **new step** button and type **Outlook send**.

![](../Images/newstep.png)




11. Scroll down the list, and click on **Send an email (V2)** action to add it to the Designer.

> Note: you will need to sign into your Outlook.com (Use Odl user from Environment details) and grant permissions for the Logic App to send email using your account.

12.	In the Send an email (V2), add your email address in the **To** field.

> Later, you will use the same email address to check if you have received an email using workflow automation feature.

13.	Click in the **Subject box**, then type: **Recommendation changed:**

14.	Click just after Recommendation changed: to get your cursor in the right place. In the dynamic content box, click on **Dynamic content** and then select `Properties Display Name` in the list (click Add dynamic content if it doesn’t pop out automatically).

15.	Click into the Body text box and type the following:

**The following recommendation has been changed**</br>
**Recommendation:**</br>
**Description:**</br>
**Status:**</br>
**Link to recommendation:**</br>

16.	Click just after each section, to get your cursor in the right place. In the **dynamic content box**, match each line to the following content by selecting in the list:

Recommendation: `Properties Display Name`</br>
Description: `Properties Metadata Description`</br>
Status: `Properties Status Code`</br>
Link to recommendation: `Properties Links Azure Portal Uri`</br>

17.	Verify that Your Logic App looks like the below screenshot and then click on **Save** in the Logic App Designer.

![Logic App worklfow](../Images/outlook-send.png)

**Create a new workflow automation instance**

1.	Type **Security Center** in the search box located on the top of the **Azure Portal** page and click on it, then select **Workflow automation** under **Management** section from the left sidebar.

2.	Click on **Add workflow automation**.

3.	A pane appears on the right side. Enter the following for each field:
    - General:
        - Name: *Send-RecommendationsChanges*
        - Description: *Send email message when a recommendation is created or triggered*
        - Subscription: *Your Subscription*
        - Resource group: *asclab*
    - Trigger conditions:
        - Select Security Center data types: *Security Center recommendations*
        - Recommendations name: *All recommendations selected*
        - Recommendation severity: *All severities selected*
        - Recommendation state: *All states selected*
    - Actions:
        - Show Logic App instances from the following subscriptions: *Your Subscription*
        - Logic App name: *Send-RecommendationsChanges*
    Click **Create** to complete the task.

![](../Images/workflow-automation.png)

4.	Wait for the message *"Workflow automation created successfully. Changes may take up to 5 minutes to be reflected"* to appear. From now on, you will get email notifications for recommendations.
Once you start to get email notifications, you can disable the automation by selecting the workflow and clicking on **Disable**.

> Please be aware that if your trigger is a recommendation that has "sub-recommendations” / “nested recommendations”, the logic app will not trigger for every new security finding; only when the status of the parent

5. Once the automation is automatically triggered, you should expect the email message to look like the screenshot below:

![Workflow automation generated email message](../Images/asc-workflow-automation-automated-email.gif?raw=true)

6.	Test/trigger your automation manually:
    - On Security Center sidebar, click on **Recommendations**.
    - Look for recommendation **Azure Defender for SQL should be enabled on your SQL servers** under **Remediate vulnerabilities** click on it.

    ![](../Images/trigger-logic-app.png)


    - Select resource **asclab-sql-xxx** (here xxx is unique ID) and then click on the **Trigger Logic App** button.
    - In the Logic App Trigger blade, select the Logic App you created in the previous step (Send-RecommendationsChanges) then click on **Trigger**.
    - You should receive an email, verify in your inbox.


![](../Images/trigger-logic-app1.png)

### Exercise 4: Accessing your secure score via ARG
Azure Resource Graph (ARG) provide an efficient and performant resource exploration with the ability to query at scale across a given set of subscriptions.
Azure Secure Score data is available in ARG so you can query and calculate your score for the security controls and accurately calculate the aggregated score across multiple subscription.

1.	Type **arg** in the search box located on the top of the **Azure Portal** page and click on **Resource Graph Explorer**.

![Resource Graph Explorer](../Images/asc-resource-graph-explorer.gif?raw=true)

2.	Paste the following KQL query and then select **Run query**.

```
SecurityResources
| where type == 'microsoft.security/securescores'
| extend current = properties.score.current, max = todouble(properties.score.max)
| project subscriptionId, current, max, percentage = ((current / max)*100)
```

![](../Images/run-query1.png)

3.	You should now see your subscription ID listed here along with the current score (in points), the max score and the score in percentage.

4.	To return the status of all the security controls, select **New query**. Next, paste the following KQL query and click on **Run query**:

```
SecurityResources
| where type == 'microsoft.security/securescores/securescorecontrols'
| extend SecureControl = properties.displayName, unhealthy = properties.unhealthyResourceCount, currentscore = properties.score.current, maxscore = properties.score.max
| project SecureControl , unhealthy, currentscore, maxscore
```

![](../Images/run-query2.png)

More details on the [official article](https://docs.microsoft.com/en-us/azure/security-center/secure-score-security-controls) or on the [blog post](https://techcommunity.microsoft.com/t5/azure-security-center/querying-your-secure-score-across-multiple-subscriptions-in/ba-p/1749193)


### Summary

  * In this module, you have completed exploring more **Security Center** features - **Vulnerability assessment for VMs**, **Vulnerability assessment for Containers**, **Automated recommendations with workflow automation** and **Accessed your secure score via ARG**.

Now you can move on to the next module by clicking on the Next button at the bottom right of the screen.
