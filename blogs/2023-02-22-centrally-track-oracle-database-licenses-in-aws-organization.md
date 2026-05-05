---
title: "Centrally track Oracle database licenses in AWS Organizations using AWS License Manager and AWS Systems Manager"
url: "https://aws.amazon.com/blogs/mt/centrally-track-oracle-database-licenses-in-aws-organizations-using-aws-license-manager-and-aws-systems-manager/"
date: "Wed, 22 Feb 2023 17:46:01 +0000"
author: "Praveen Bhat"
feed_url: "https://aws.amazon.com/blogs/mt/tag/aws-license-manager/feed/"
---
<p>As you continue to run your business-critical workloads in hybrid environments, you’ll most likely face the challenges of license management of products, such as Microsoft, SAP, Oracle, and IBM due to limited visibility and governance. You’ll most likely eventually over-provision licenses to avoid the headache with third-party license providers or under-provisioning licenses, only to face steep penalties.</p> 
<p><a href="https://aws.amazon.com/license-manager/">AWS License Manager</a> makes it simpler to track your licensing usage for licenses from different vendors. Here are the two main use cases:</p> 
<ul> 
 <li>AWS license included – using AWS license included instances allows you access to fully-compliant licenses, where AWS handles the tracking and management for you.</li> 
 <li>Bring Your Own License (BYOL) – License Manager&nbsp;makes it easy for you to set rules to manage, discover, and report software (BYOL) license usage.</li> 
</ul> 
<p>License Manager utilizes <a href="https://aws.amazon.com/systems-manager/">AWS Systems Manager</a> agent to discover applications and database editions. However, a few complex scenarios like discovering Oracle database edition requires user access and permissions to the Oracle database. Therefore, due to limited insights in the Oracle editions and management packs, license compliance management becomes a challenging process. In this post, I’ll show you how to build a solution using Systems Manager and License Manager to centrally discover and track your BYOL Oracle database editions and management packs across AWS accounts and Regions within an <a href="https://aws.amazon.com/organizations/">AWS Organizations</a>.</p> 
<p>Note:</p> 
<ul> 
 <li>This solution can discover and track Oracle databases from version 11.2 and later, running on <a href="https://aws.amazon.com/ec2/">Amazon Elastic Compute Cloud (Amazon EC2)</a>, Linux flavour instances.</li> 
 <li>Information derived from this solution is to be used for informational purposes only and does not represent any license entitlement or requirement.</li> 
</ul> 
<h2>Prerequisites</h2> 
<p>To deploy this solution across multiple regions and/or accounts in an organization, complete&nbsp;the following steps.</p> 
<ul> 
 <li><strong>Database Options/Packs Usage Reporting.</strong> To help identify which features and license options are used in a database, you will need the Oracle provided <a href="https://support.oracle.com/knowledge/Oracle%20Database%20Products/1317265_1.html">options_packs_usage_statistics.sql</a> script on MOS in Database Options/Management Packs Usage Reporting for Oracle Databases 11.2 and later (Doc ID 1317265.1). Please refer to MOS DOC ID 1309070.1 for additional information on DBA_FEATURE_USAGE_STATISTICS, which is the main source of information for the above script.</li> 
 <li><strong>Enable trusted access with Organizations for CloudFormation</strong>. This solution leverages <a href="https://aws.amazon.com/cloudformation/">AWS CloudFormation</a> and <a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/what-is-cfnstacksets.html">StackSets</a> to provision all of the required components. Because StackSets perform stack operations across multiple accounts and Regions, before you can create your first stack set you need the necessary permissions defined in your AWS accounts. Complete the following tasks as described in <a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/stacksets-orgs-enable-trusted-access.html">Enable trusted access with AWS Organizations</a>: 
  <ul> 
   <li>Enable all of the features in Organizations. Enabling all of the features allows the solution to create StackSets with service-managed permissions and sharing license configurations with member accounts.</li> 
   <li>Enable trusted access with Organizations. Enabling trusted access allows StackSets to create the necessary <a href="https://aws.amazon.com/iam/">AWS Identity and Access Management (IAM)</a> roles in the organization’s management account and member accounts when the solution creates stack sets with service-managed permissions.</li> 
  </ul> </li> 
 <li><strong>Use a delegated administrator.</strong> This solution uses the management account within AWS Organizations, but you can also designate an account (delegated administrator) to manage this on behalf of the organization. If you intend to use a delegated account then you will need to register it as delegated administrator for CloudFormation stack set operations as described in Register a delegated administrator.</li> 
 <li><strong>Distribute license configurations with member accounts in all target regions</strong>. To distribute self-managed licenses within your organization, from the License Manager console of the organization’s management account, choose <em>Settings</em>, and then select <em>Link AWS Organizations accounts</em>. When you select this option, we add a service-linked role to the management and member accounts. Repeat this step for all target regions. If you intend to use delegated administrator account, then from the License Manager console of the organization’s management account, choose <em>Settings</em>, and under <em>Delegated administrator</em>, choose <em>Delegate administrator</em>. Enter the account ID number for the AWS account that you want to assign, and then choose <em>Delegate</em>. You can’t use the ID for the management account. It must be a member account.</li> 
</ul> 
<p style="padding-left: 40px;">Once completed, under the Settings section you should see a link to the new Resource Share ARN (<a href="https://aws.amazon.com/ram/">AWS Resource Access Manager (AWS RAM)</a>), as shown in the following figure.</p> 
<p><img alt="" class="aligncenter wp-image-37158 size-full" height="428" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2023/02/22/Figure-1-License-Manager-Settings-with-resource-share.png" width="1192" /></p> 
<p style="text-align: center;"><em>Figure 1: License Manager Settings with resource share ARN</em></p> 
<ul> 
 <li><strong>Create license configurations</strong>. In License Manager, create license configurations for these Oracle database editions in each AWS Region where Oracle databases are deployed: 
  <ul> 
   <li><em>OracleDbEELicenseConfiguration</em> for Enterprise Edition</li> 
   <li><em>OracleDbSE2LicenseConfiguration</em> for Standard Edition 2</li> 
   <li><em>OracleDbPELicenseConfiguration</em> for Personal Edition</li> 
   <li><em>OracleDbXELicenseConfiguration</em> for Express Edition</li> 
  </ul> </li> 
</ul> 
<p style="padding-left: 40px;">A license configuration represents the licensing terms in the agreement with your software vendor. Using <a href="https://aws.amazon.com/cloudshell/">AWS CloudShell</a>, run the following command to create the following license configurations (case-sensitive), replace REGION with your target regions.</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code" style="padding-left: 40px;"><code class="lang-bash">for r in REGION-1 REGION-2; do for i in OracleDbEELicenseConfiguration OracleDbSE2LicenseConfiguration OracleDbPELicenseConfiguration OracleDbXELicenseConfiguration; do aws license-manager create-license-configuration --name "$i" --license-counting-type vCPU --region $r; done; done</code></pre> 
</div> 
<p style="padding-left: 40px;">For instructions to create these license configurations using the console, see <a href="https://docs.aws.amazon.com/license-manager/latest/userguide/create-license-configuration.html">Create a license configuration</a>&nbsp;in the License Manager User Guide.</p> 
<ul> 
 <li><strong>Share license configurations</strong>. After defining your license configurations, use Organizations or AWS RAM to share those license configurations with your member accounts. For instructions to share license configurations, see the <a href="https://aws.amazon.com/blogs/mt/tracking-software-usage-across-multiple-aws-accounts-using-aws-license-manager/">Tracking software usage across multiple AWS accounts using AWS License Manager</a> post.</li> 
</ul> 
<p style="padding-left: 40px;">After you share your principals (AWS account/Organization/Organizational unit) and resources (license configurations), you should see them in the AWS RAM console:</p> 
<p><img alt="The AWS RAM console displays shared resources and shared principals in lists organized by ID, type, and status.]" class="wp-image-34232 aligncenter" height="317" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2022/11/15/cloudops_866_3.png" width="700" /></p> 
<p style="text-align: center;"><em>Figure 2: Shared principals and resources in the AWS RAM console</em></p> 
<ul> 
 <li><strong>Manage instances using Systems Manager</strong>. A <a href="https://docs.aws.amazon.com/systems-manager/latest/userguide/managed_instances.html">managed instance</a> is an Amazon EC2 instance that is configured for use with Systems Manager. Managed instances can use Systems Manager services such as Run Command, Patch Manager, and Session Manager. You must make sure that all instances targeted for this solution meet the prerequisites to become a managed instance including configuring instance permissions for Systems Manager as described in <a href="https://docs.aws.amazon.com/systems-manager/latest/userguide/setup-instance-permissions.html">Setting up Systems Manager for EC2 instances</a>.</li> 
</ul> 
<h2>Solution overview</h2> 
<p>The following figure shows the solution architecture. In addition to License Manager, the solution uses the following Systems Manager features:</p> 
<ul> 
 <li><a href="https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-automation.html">Automation</a> to orchestrate the discovery workflow.</li> 
 <li><a href="https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-state.html">State Manager</a> to invoke the Automation document on a user-defined frequency.</li> 
 <li><a href="https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-inventory.html">Inventory</a> to maintain the information collected about the instances, the Oracle database editions and management packs running on them.</li> 
</ul> 
<p>Note that Steps 1 – 8 help you discover the Oracle editions and packs. Steps 9–11 help you query and visualize the discovered data using&nbsp;<a href="https://aws.amazon.com/athena/">Amazon Athena</a>&nbsp;and&nbsp;<a href="https://aws.amazon.com/quicksight/">Amazon QuickSight</a>. This post is focused toward the discovery and tracking of Oracle database editions and management packs, to learn more about data visualization refer to <a href="https://aws.amazon.com/blogs/mt/query-and-visualize-microsoft-sql-server-license-utilization-using-amazon-athena-and-amazon-quicksight/">Query and visualize Microsoft SQL Server license utilization using Amazon Athena and Amazon QuickSight</a>.</p> 
<p><img alt="" class="aligncenter wp-image-39221 size-large" height="573" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2023/05/15/Figure-4-Solution-Architecture-1-1024x573.png" width="1024" /></p> 
<p style="text-align: center;"><em>Figure 3: Solution architecture</em></p> 
<ol> 
 <li><strong>Invoke the OracleDbLTS-Orchestrate Automation</strong>: State Manager invokes the OracleDbLTS-Orchestrate Automation and passes the required parameters using which the solution determines the target Organizational Unit IDs/AWS accounts and Regions where your Oracle database instances are deployed.</li> 
 <li><strong>Remove old custom Inventory data</strong>: In this step, the Orchestrate Automation first invokes the OracleDbLTS-DeleteInventory Automation in the target member account to remove the old custom Inventory schema in Systems Manager Inventory, making sure that Inventory data is current. Inventory data comprises of Oracle database editions and all the packs installed and/or used.</li> 
 <li><strong>Invoke the OracleDbLTS-ManageLicenceUtilization Automation</strong>: Once the deletion has been completed, the OracleDbLTS-Orchestrate Automation invokes the OracleDbLTS-ManageLicenceUtilization Automation to initiate the discovery of Oracle databases in your account and track their utilization for license management.</li> 
 <li><strong>Remove old License Manager data</strong>: The Automation first disassociates the target instance from an existing License Configuration. This makes sure that the latest discovered licenses are available in License Manager for scenarios where changes have been made on the instance. For example, somebody deletes or installs a new edition of Oracle database on the target instance after the previous Automation run.</li> 
 <li><strong>Download license tracking solution scripts</strong>: These three scripts are required to query the database instance to determine the edition and management packs installed and used.</li> 
 <li><strong>Discovery</strong>: The Discover Automation then targets instances based on the State Manager association definition to determine the type of Oracle database running, and stores this data in the artifacts bucket under ssm-output. Instances can be targeted using ParameterValues, ResourceGroup or with tag: (default), AWS::EC2::Instance, InstanceIds, instanceids. Refer the API reference for <a href="https://docs.aws.amazon.com/systems-manager/latest/APIReference/API_Target.html">Target</a> for more details.</li> 
 <li><strong>Update Inventory</strong>: The discovered data is used to update the Systems Manager Inventory. In this step, Automation creates two new custom schemas along with the metadata to store the Oracle edition details along with the management packs.</li> 
 <li><strong>Update License Manager</strong>: Finally, the Automation updates the License Manager with the license utilization data and associates the target instance with the appropriate license specification that has been defined in License Manager. Discovered data under ssm-output is cleared for the next run.</li> 
</ol> 
<p style="padding-left: 40px;">Note that Steps 1 – 8 utilizes Systems Manager Automation to discover and track Oracle database editions and packs. Refer to Steps 9 – 11 if you want to aggregate, query, and visualize the discovered data.</p> 
<ol start="9"> 
 <li><strong>Aggregate Inventory data using resource data sync</strong>: You can use Systems Manager resource data sync to send Inventory data collected from all of your managed instances across the member accounts to a single <a href="https://aws.amazon.com/s3/">Amazon Simple Storage Service (Amazon S3)</a> bucket. Then, resource data sync automatically updates the centralized data when new Inventory data is collected. For more details, see <a href="https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-inventory-resource-data-sync.html">Walkthrough: Use resource data sync to aggregate inventory data</a>.</li> 
 <li><strong>Query the centralized Inventory data</strong>: You can use Amazon Athena which provides an interactive query service to analyze the Inventory data in Amazon S3 using standard SQL.</li> 
 <li><strong>Visualize Inventory data</strong>: With <a href="https://aws.amazon.com/quicksight/">Amazon QuickSight</a> you can create and publish interactive BI dashboards with insights powered by machine learning (ML).</li> 
</ol> 
<h2>Walkthrough</h2> 
<p>To deploy the solution, download this <a href="https://d2908q01vomqb2.cloudfront.net/artifacts/MTBlog/Cloudops-866/oracle-db-lts/CloudFormation/oracle-lts-template.yaml">CloudFormation template</a> and deploy it in the management or delegated administrator account of your organization.</p> 
<p>This template deploys the following resources required for this solution:</p> 
<ol> 
 <li><strong>Systems Manager documents</strong></li> 
</ol> 
<ul> 
 <li> 
  <ul> 
   <li>OracleDbLTS-Orchestrate includes the logic to run step 1 and 3 of the walk-through.</li> 
   <li>OracleDbLTS-DeleteInventory includes the logic to run step 2 of the walk-through.</li> 
   <li>OracleDbLTS-ManageLicenceUtilization includes the logic to run steps 4-8 of the walk-through.</li> 
  </ul> </li> 
</ul> 
<ol start="2"> 
 <li><strong>IAM roles and policy<br /> </strong></li> 
</ol> 
<ul> 
 <li> 
  <ul> 
   <li>OracleDbLTS-SystemsManagerAutomationAdministrationRole, for the administration of the Automation documents.</li> 
   <li>OracleDbLTS-SystemsManagerAutomationExecutionRole, which is deployed using StackSets across all the target accounts and regions for the execution of the Automation documents.</li> 
   <li>OracleDbLTS-SSMS3BucketPolicy, a managed policy that gets added to all the target managed instance’s IAM role(s) which allows Systems Manager to read and write from the artifacts bucket.</li> 
   <li>OracleDbLTSUtilityFunctionRole, required for the lambda function OracleDbLTS-UtilityFunction.</li> 
  </ul> </li> 
</ul> 
<ol start="3"> 
 <li><strong>S3 bucket</strong></li> 
</ol> 
<ul> 
 <li> 
  <ul> 
   <li>An artifacts bucket that will store all the required tracking scripts and (transient) output from Systems Manager.</li> 
  </ul> </li> 
</ul> 
<ol start="4"> 
 <li><strong>OracleDbLTS-UtilityFunction</strong></li> 
</ol> 
<ul> 
 <li> 
  <ul> 
   <li>A lambda utility function to 
    <ul> 
     <li>create the State Manager associations, one association for 50 child OUs under the specified TargetOUs parameter. This helps address the following limits within Automation (1) 50 organizational units/accounts as TargetLocation (TargetLocation), and (2) ability to run recursively through OUs (running automations in multiple AWS Regions and accounts).</li> 
     <li>setup the S3 bucket including copying the two scripts required for the solution.</li> 
    </ul> </li> 
  </ul> </li> 
</ul> 
<p>Once your template has been deployed, use <a href="https://aws.amazon.com/cloudshell/">AWS CloudShell</a> to upload the <a href="https://support.oracle.com/knowledge/Oracle%20Database%20Products/1317265_1.html">options_packs_usage_statistics.sql</a> script to the <em>s3://ARTIFACT-BUCKET-TARGET/scripts/</em> bucket.</p> 
<h2>Invoking the solution using a State Manager association</h2> 
<p>Association(s) created by the OracleDbLTS-UtilityFunction invokes the solution once upon creation, and then follows the user-defined cron interval for future executions. By default the cron schedule for the associations is triggered to run on the last Tuesday of every month at midnight UTC.</p> 
<h2>Validating that the execution ran successfully</h2> 
<p>After the association has triggered the Automation, open the Systems Manager console in the management account, and from the left navigation pane choose Automation. In Automation executions, you should see the status of OracleDbLTS-Orchestrate along with OracleDbLTS-DeleteInventory and OracleDbLTS-ManageLicenceUtilization, as shown in the following figure.</p> 
<p><img alt="" class="aligncenter wp-image-37138 size-full" height="888" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2023/02/22/Figure-5-Automation-executions-management-account.png" width="4596" /></p> 
<p style="text-align: center;"><em>Figure 4: Automation executions (management account)</em></p> 
<p>For more details on the status of individual instances, you can click on the Step ID of OracleDbLTS-ManageLicenceUtilization and navigate to the instance of interest, as shown in the figure below.</p> 
<p><img alt="" class="aligncenter wp-image-37139 size-full" height="2184" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2023/02/22/Figure-6-Automation-execution-detail-management-account.png" width="4594" /></p> 
<p style="text-align: center;"><em>Figure 5: Automation execution detail (management account)</em></p> 
<p>To confirm that the license utilization data has been updated in License Manager, using the management account and selected Region, open the License Manager console. Depending on the licenses consumed, the <strong>Customer managed licenses</strong> list will look something like the below figure:</p> 
<p><img alt="The customer managed licenses are displayed in a list organized by license configuration name, status, license type, licenses consumed, and account ID.] " class="wp-image-34259 aligncenter" height="247" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2022/11/15/cloudops_866_10.png" width="887" /></p> 
<p style="text-align: center;"><em>Figure 6: Customer managed licenses (management account)<br /> </em></p> 
<p>All information regarding the Oracle database, including editions and management packs utilization, is aggregated in Systems Manager Inventory in the member account. You can view this information in the Inventory (aggregated view) console or Fleet Manager (instance view) console. The previous figure and following figure show the details of an Oracle database edition and management packs being utilized in Fleet Manager.</p> 
<p><img alt="Oracle database edition details - created, instance name, database name, banner, database role, host name, DBID, version and open mode in Fleet Manager." class="wp-image-34260 aligncenter" height="447" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2022/11/15/cloudops_866_11.png" width="841" /></p> 
<p style="text-align: center;"><em>Figure 7: Oracle database edition in Fleet Manager (member account)<br /> </em></p> 
<p><img alt="Figure 11: Oracle database feature/management packs usage details in Fleet Manager" class="wp-image-34261 aligncenter" height="342" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2022/11/15/cloudops_866_12.png" width="826" /></p> 
<p style="text-align: center;"><em>Figure 8: Oracle database feature/management packs usage details in Fleet Manager (member account)<br /> </em></p> 
<p>If you want to aggregate, query, and visualize the discovered data, you can refer to steps 9 – 11 in the Solution overview.</p> 
<p>Note that Systems Manager Agent runs on Amazon EC2 instances using root permissions (Linux) or SYSTEM permissions (Windows Server). This gives it the ability to switch to the Oracle user, which then has the permissions to query the certain tables and databases to query all of the information required to track the license utilization. For more details regarding restricting the permissions of the Systems Manager Agent, you can refer to <a href="https://docs.aws.amazon.com/systems-manager/latest/userguide/ssm-agent-restrict-root-level-commands.html">Restricting access to root-level commands through SSM Agent</a>.</p> 
<h2>Adding new accounts and Regions</h2> 
<p>If you add new OUs outside of the currently targeted OUs or Regions, then you must update the CloudFormation template. However, if you are only adding accounts to the already targeted OUs then the solution will automatically target them.</p> 
<h3>CloudFormation:</h3> 
<ol> 
 <li>In the CloudFormation console, choose the original template that you deployed, and then choose <strong>Update</strong>.</li> 
 <li>Leave the <strong>Use the</strong> <strong>current template</strong> option selected.</li> 
 <li>Under <strong>Automation Documents</strong>, update the <strong>TargetRegions</strong> and/or <strong>TargetOUs</strong> parameters with the new values.</li> 
</ol> 
<h2>Clean up</h2> 
<p>If you want to remove the resources and solution after testing, then you can clean up the resources deployed by the CloudFormation template using the CloudFormation console or AWS CLI to delete the main CloudFormation stack. When you delete the CloudFormation stack, all of the solution components will be deleted.</p> 
<h2>Conclusion</h2> 
<p>In this post, I showed you how to use License Manager, Systems Manager, and Organizations to automate the tracking of your Oracle database licenses, including management packs running on Amazon EC2 instances across multiple accounts and Regions. This solution can be extended to govern other software licenses consumed not only in AWS, but also across your hybrid environments, throughout your organization to avoid any surprises during your next audit.</p> 
<p><strong>About the author:</strong></p> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="aligncenter size-full wp-image-11636" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2022/11/15/Praveen-Bhat.png" />
  </div> 
  <h3 class="lb-h4">Praveen Bhat</h3> 
  <p>Praveen Bhat is a Principal Solutions Architect with several years of experience in the technology industry. By using his passion to bridge the gap between technology and business, Praveen has helped banking, insurance, manufacturing, government, wagering, and media organizations achieve their business objectives.</p> 
 </div> 
</footer> 
<div id="aws-account-email-popup" style="color: white; width: auto; float: left; padding: 10px; font-size: 14px;"> 
 <div>
  CTI: Not available
 </div> 
</div>
