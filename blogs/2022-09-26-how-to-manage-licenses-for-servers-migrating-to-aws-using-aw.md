---
title: "How to Manage Licenses for Servers Migrating to AWS using AWS License Manager"
url: "https://aws.amazon.com/blogs/mt/how-to-manage-licenses-for-servers-migrating-to-aws-using-aws-license-manager/"
date: "Mon, 26 Sep 2022 16:42:56 +0000"
author: "Anutosh"
feed_url: "https://aws.amazon.com/blogs/mt/tag/aws-license-manager/feed/"
---
<p>We often see large enterprises migrating their workloads to AWS, reaping the benefits of the state-of-the-art migration tool <a href="https://aws.amazon.com/application-migration-service/">AWS Application Migration Service</a>, and they prefer migrating their Microsoft workloads along with licenses. This post will show how we can lift and shift large enterprise workloads with Windows Bring Your Own Licenses (BYOL) using Application Migration Service and <a href="https://aws.amazon.com/license-manager/">AWS License Manager</a>. License Manager helps you gain control over license usage, reduce costs, and reduce the risk of non-compliance.</p> 
<p>This post will cover two setup scenarios. The first will be a Single AWS Account Setup where we deploy all the workloads in a single AWS account, and the other will be a Multi-Account Setup where we deploy workloads across multiple AWS accounts. AWS <a href="https://aws.amazon.com/organizations/getting-started/best-practices/">recommends</a> that you set up multiple accounts as your workloads grow in size and complexity.</p> 
<p>Your existing Microsoft licenses may be used on AWS with&nbsp;<a href="https://aws.amazon.com/ec2/dedicated-hosts/">Amazon EC2 Dedicated Hosts</a>,&nbsp;<a href="https://aws.amazon.com/ec2/purchasing-options/dedicated-instances/">Amazon EC2 Dedicated Instances</a>, using <a href="https://aws.amazon.com/windows/resources/licensemobility/">Microsoft License Mobility through Software Assurance</a>.</p> 
<h2>Solution walkthrough</h2> 
<h3>Scenario 1 – Single Account Setup</h3> 
<p>In the Single Account Setup, all the required AWS resources and services will run off of a single AWS account.</p> 
<p>The following diagram shows the solution architecture for a single account setup where on-premises servers are migrating over to AWS using Application Migration Service that utilizes Amazon EC2 Launch Templates with predefined launch parameters to launch the BYOL test and cutover servers. License Manager will use the Host Resource Group of Dedicated Hosts to track the BYOL servers license usage based on the license configuration rules specified.</p> 
<div class="wp-caption aligncenter" id="attachment_32978" style="width: 710px;">
 <img alt="Figure 1. Single Account Setup Architecture" class="wp-image-32978" height="443" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2022/09/24/AWS-MGN-License-Manager-1-Single-Account.drawio_Fig_1.png" width="700" />
 <p class="wp-caption-text" id="caption-attachment-32978">Figure 1. Single Account Setup Architecture</p>
</div> 
<h2>Prerequisites</h2> 
<p>The following prerequisites are required for following along with this post:</p> 
<ol> 
 <li>An AWS Account with <a href="https://docs.aws.amazon.com/mgn/latest/ug/first-time-setup-gs.html">Application Migration Service setup</a>. If you do not have this already setup, please do the following for the first time setup:</li> 
</ol> 
<ol> 
 <li> 
  <ol type="a"> 
   <li>From the AWS Management Console, go to Application Migration Service, select Get started, and then setup Application Migration Service replication settings.</li> 
   <li>Add a Source Server, Install the AWS Replication Agent and then wait for the initial sync to complete.</li> 
  </ol> </li> 
</ol> 
<ol start="2"> 
 <li>A <a href="https://docs.aws.amazon.com/mgn/latest/ug/source-servers.html">Source Server</a> with the <a href="https://docs.aws.amazon.com/mgn/latest/ug/agent-installation-instructions.html">AWS Replication Agent</a> installed, <a href="https://docs.aws.amazon.com/mgn/latest/ug/lifecycle.html#initiation">Initial Sync</a> complete and <a href="https://docs.aws.amazon.com/mgn/latest/ug/configuring-target-gs.html">launch settings</a> configured.</li> 
 <li>An <a href="https://aws.amazon.com/iam/">AWS Identity and Access Management (IAM)</a> role or a user with permissions to perform Application Migration Service setup, Amazon EC2, and <a href="https://docs.aws.amazon.com/license-manager/latest/userguide/identity-access-management.html">License Manager</a> operations.</li> 
</ol> 
<h2>Part A – Setup License Manager</h2> 
<p>In this example, we’ve selected the license type to track Cores with the Number of Cores limit set to 36. The license type is the counting model for the license (<strong>vCPUs,&nbsp;Cores,&nbsp;Sockets</strong>, or&nbsp;<strong>Instances</strong>).</p> 
<ol> 
 <li>Open the <a href="https://console.aws.amazon.com/license-manager/">License Manager console</a> (Note that if you’re a first-time user, then you will have to set up onetime <a href="https://docs.aws.amazon.com/license-manager/latest/userguide/using-service-linked-roles.html">IAM service-linked role</a>, as License Manager requires a service-linked role to manage licenses on your behalf).</li> 
 <li>Select <strong>Self-managed licenses</strong> and select <strong>Create self-managed license</strong>. Here you will set up licensing rules based on the terms of your enterprise agreement to track any software that is licensed based on virtual cores (vCPUs), physical cores, sockets, or the number of machines. Then select <strong>Submit</strong>.</li> 
</ol> 
<div class="wp-caption aligncenter" id="attachment_32979" style="width: 710px;">
 <img alt="Create self-managed license configuration" class="wp-image-32979" height="332" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2022/09/21/cloudops_1069_2.png" width="700" />
 <p class="wp-caption-text" id="caption-attachment-32979">Figure 2. Create self-managed license configuration</p>
</div> 
<ol start="3"> 
 <li>In the AWS License Manager Console, choose <strong>Host resource groups</strong> and select <strong>Create host resource group</strong>.</li> 
</ol> 
<p>A host resource group is a collection of Amazon EC2 Dedicated Hosts that are associated with your server-bound licenses. After you create a host resource group, License Manager manages the hosts for you to track licenses shared to this group. Here you will configure the <strong>Host resource group name, EC2 Dedicated Host management settings and Associate the self-managed licenses</strong>.</p> 
<div class="wp-caption aligncenter" id="attachment_32980" style="width: 710px;">
 <img alt="Create host Resource group on License Manager" class="wp-image-32980" height="700" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2022/09/21/cloudops_1069_3.png" width="700" />
 <p class="wp-caption-text" id="caption-attachment-32980">Figur 3. Create host Resource group on License Manager</p>
</div> 
<h2>Part B – Configure Launch Template on Application Migration Service</h2> 
<p>This step will continue assuming that Application Migration Service is already initialized and setup, the <a href="https://docs.aws.amazon.com/mgn/latest/ug/agent-installation-instructions.html">AWS Replication Agent</a> is installed on the source server, and the <a href="https://docs.aws.amazon.com/mgn/latest/ug/lifecycle.html#initiation">Initial Sync</a> is complete and you are ready to launch test instances.</p> 
<ol> 
 <li>Open the Application Migration Service <a href="https://console.aws.amazon.com/mgn/home">console</a>.</li> 
 <li>Select <strong>Source servers</strong> and select the <strong>Source server name</strong>.</li> 
 <li>Select <strong>Launch settings</strong> then select <strong>Modify</strong> under <strong>EC2 Launch Template</strong>.</li> 
</ol> 
<div class="wp-caption aligncenter" id="attachment_32981" style="width: 710px;">
 <img alt="Application Migration Service Launch Settings" class="wp-image-32981" height="571" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2022/09/21/cloudops_1069_4.png" width="700" />
 <p class="wp-caption-text" id="caption-attachment-32981">Figure 4. Application Migration Service Launch Settings</p>
</div> 
<ol start="4"> 
 <li>Configure the Launch template name and version and choose <strong>Don’t include in launch template</strong> under <strong>Application and OS Images (Amazon machine Image)</strong>, since it will take this from the Application Migration Service.</li> 
</ol> 
<div class="wp-caption aligncenter" id="attachment_32982" style="width: 710px;">
 <img alt="Configure EC2 Launch Template" class="wp-image-32982" height="636" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2022/09/21/cloudops_1069_5.png" width="700" />
 <p class="wp-caption-text" id="caption-attachment-32982">Figure 5. Configure EC2 Launch Template</p>
</div> 
<ol start="5"> 
 <li>Continue filling in the rest of the contents where applicable until you’ve reached the <strong>Advanced details</strong> section to expand and specify the following settings:</li> 
</ol> 
<p style="padding-left: 40px;">Tenancy: <strong>Dedicated host</strong> –<strong> launch this instance on a dedicated Host</strong></p> 
<p style="padding-left: 40px;">Target host by: <strong>Host Resource Group</strong></p> 
<p style="padding-left: 40px;">Tenancy host ID: <strong>Select your dedicated host ID for your BYOL Instances</strong></p> 
<p style="padding-left: 40px;">License configuration: <strong>Select the license configuration ARN from the drop-down</strong>.</p> 
<div class="wp-caption aligncenter" id="attachment_32983" style="width: 710px;">
 <img alt="Configure EC2 Launch Template Advanced details" class="wp-image-32983" height="497" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2022/09/21/cloudops_1069_6.png" width="700" />
 <p class="wp-caption-text" id="caption-attachment-32983">Figure 6. Configure EC2 Launch Template Advanced details</p>
</div> 
<ol start="6"> 
 <li>Select <strong>Create template Version</strong>.</li> 
 <li>You can select this version that you’ve just created and set it as <strong>Default</strong>.</li> 
</ol> 
<h2>Part C – Launch test instances from Application Migration Service to Amazon EC2</h2> 
<ol> 
 <li>Open the Application Migration Service <a href="https://console.aws.amazon.com/mgn/home">console</a> and select the <strong>Source server name</strong>.</li> 
 <li>Select <strong>Test and cutover</strong> then select <strong>Launch test instances</strong>.</li> 
 <li>Once the test instance is launched successfully, you can check the <strong>AWS License Manager Dashboard</strong> for usage.</li> 
</ol> 
<div class="wp-caption aligncenter" id="attachment_32984" style="width: 710px;">
 <img alt="AWS License Manager Dashboard" class="wp-image-32984" height="514" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2022/09/21/cloudops_1069_7.png" width="700" />
 <p class="wp-caption-text" id="caption-attachment-32984">Figure 7. AWS License Manager Dashboard</p>
</div> 
<h2>Scenario 2 – Multi-Account Setup</h2> 
<p>In the Multi-Account Setup, we’re using <a href="https://aws.amazon.com/organizations/">AWS Organizations</a> to centrally manage multiple AWS accounts. In this setup, License Manager, <a href="https://aws.amazon.com/ram/">AWS Resource Access Manager (AWS RAM)</a>&nbsp; and the Amazon EC2 Dedicated Host(s) will run off of the Management account. Application Migration Service will be configured on a Member account.</p> 
<p>The following diagram shows the solution architecture for Multi-Account Setup where a Member Account is migrating on-premises servers over to AWS through Application Migration Service. This utilizes Amazon EC2 Launch templates that contain predefined launch parameters that BYOL images will use to launch on Amazon EC2 Dedicated Hosts. In the Management Account, License Manager will use the Host Resource Group of Dedicated Hosts that will be allocated to launch the BYOL test and cutover servers and will track the license usage based on the license configuration rules specified. The Member and Management Accounts will use a Resource Share to share a Host Resource Group of Dedicated Hosts and the license configurations.</p> 
<div class="wp-caption aligncenter" id="attachment_32985" style="width: 710px;">
 <img alt="Figure 8. Multi Account Setup Architecture" class="wp-image-32985" height="898" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2022/09/24/AWS-MGN-License-Manager-1-Multi-Account.drawio_fig_8.png" width="700" />
 <p class="wp-caption-text" id="caption-attachment-32985">Figure 8. Multi Account Setup Architecture</p>
</div> 
<h2>Prerequisites</h2> 
<p>The following prerequisites are required for following along with this post:</p> 
<ol> 
 <li>An AWS Account with Application Migration Service <a href="https://docs.aws.amazon.com/mgn/latest/ug/first-time-setup-gs.html">setup</a>.</li> 
 <li>A <a href="https://docs.aws.amazon.com/mgn/latest/ug/source-servers.html">Source Server</a> with the <a href="https://docs.aws.amazon.com/mgn/latest/ug/agent-installation-instructions.html">AWS Replication Agent</a> installed and the <a href="https://docs.aws.amazon.com/mgn/latest/ug/lifecycle.html#initiation">Initial Sync</a> complete <a href="https://docs.aws.amazon.com/mgn/latest/ug/configuring-target-gs.html">launch settings</a> configured.</li> 
 <li>An <a href="https://aws.amazon.com/iam/">AWS Identity and Access Management (IAM)</a> role or a user with permissions to perform Application Migration Service setup, Amazon EC2, and <a href="https://docs.aws.amazon.com/license-manager/latest/userguide/identity-access-management.html">License Manager</a> operations.</li> 
 <li>A <a href="https://docs.aws.amazon.com/license-manager/latest/userguide/identity-access-management.html">Member account and/or an AWS Organizations</a> structure.</li> 
 <li>An IAM role or user with permissions to perform Application Migration Service, Amazon EC2, <a href="https://docs.aws.amazon.com/license-manager/latest/userguide/identity-access-management.html">License Manager</a>, and AWS RAM operations in both the Management and Member accounts.</li> 
</ol> 
<h2>Part A – Setup AWS License Manager on the Management Account</h2> 
<ol> 
 <li>Open the <a href="https://console.aws.amazon.com/license-manager/">License Manager console</a>.</li> 
 <li>Choose <strong>Settings</strong> and select the <strong>Turn On</strong> option to enable <strong>Cross-account resource discovery</strong> to manage license usage across all of your Organization accounts.</li> 
</ol> 
<div class="wp-caption aligncenter" id="attachment_32986" style="width: 710px;">
 <img alt="Turn On Cross-account resource discovery" class="wp-image-32986" height="465" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2022/09/21/cloudops_1069_9.png" width="700" />
 <p class="wp-caption-text" id="caption-attachment-32986">Figure 9. Turn On Cross-account resource discovery</p>
</div> 
<ol start="3"> 
 <li>Next, select <strong>Self-managed licenses</strong> and select <strong>Create self-managed license</strong>. In this example, the license type specified is to track Cores with the Number of Cores limit set to 36. Then select <strong>Submit</strong>.</li> 
</ol> 
<div class="wp-caption aligncenter" id="attachment_32987" style="width: 3788px;">
 <img alt="Create self-managed managed license configuration" class="size-full wp-image-32987" height="1792" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2022/09/21/cloudops_1069_10.png" width="3778" />
 <p class="wp-caption-text" id="caption-attachment-32987">Figure 10. Create self-managed managed license configuration</p>
</div> 
<ol start="4"> 
 <li>Choose <strong>Host resource groups</strong> on the left, select <strong>Create host resource group</strong> and configure the <strong>Host resource group name</strong>, <strong>EC2 Dedicated Host management settings</strong> and <strong>Associate the self-managed licenses</strong> and select <strong>Create</strong>.</li> 
</ol> 
<div class="wp-caption aligncenter" id="attachment_32988" style="width: 710px;">
 <img alt="Create host resource group" class="wp-image-32988" height="700" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2022/09/21/cloudops_1069_11.png" width="700" />
 <p class="wp-caption-text" id="caption-attachment-32988">Figure 11. Create host resource group</p>
</div> 
<h2>Part B – Share License Configuration with Member Account(s) using AWS RAM.</h2> 
<p>Once the Host resource group is created, share this with the member accounts in your Organization by creating a Resource Share in AWS RAM.</p> 
<p>Open the AWS RAM console and choose <strong>Create a resource share</strong> to share this with the Member Accounts in your Organization.</p> 
<div class="wp-caption aligncenter" id="attachment_32989" style="width: 710px;">
 <img alt="Create a resource share in AWS RAM console" class="wp-image-32989" height="287" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2022/09/21/cloudops_1069_12.png" width="700" />
 <p class="wp-caption-text" id="caption-attachment-32989">Figure 12. Create a resource share in AWS RAM console</p>
</div> 
<ol> 
 <li>At <strong>Specify resource share details</strong>, specify the resources to share across accounts. At the <strong>Resources – optional</strong> section, select the drop-down at <strong>Select resource type</strong>, filter and select the resources <strong>License Configurations</strong> and <strong>Resource Groups</strong> (Self-managed license and Host resource group) created in Part A and B.</li> 
 <li>This will share the Host resource group and Self-managed licenses permissions with other accounts, then continue by selecting <strong>Next</strong>.</li> 
</ol> 
<div class="wp-caption aligncenter" id="attachment_32990" style="width: 710px;">
 <img alt="Specify resource share details" class="wp-image-32990" height="585" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2022/09/21/cloudops_1069_13.png" width="700" />
 <p class="wp-caption-text" id="caption-attachment-32990">Figure 13. Specify resource share details</p>
</div> 
<div class="wp-caption aligncenter" id="attachment_32991" style="width: 710px;">
 <img alt="Specify resource share details in AWS RAM" class="wp-image-32991" height="406" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2022/09/21/cloudops_1069_14.png" width="700" />
 <p class="wp-caption-text" id="caption-attachment-32991">Figure 14. Specify resource share details in AWS RAM</p>
</div> 
<ol start="3"> 
 <li>Select <strong>Next</strong> to the <strong>Grant access to principals</strong> step and add the AWS account number of the Member account with which you want to share these resources, select <strong>Add</strong> and then select <strong>Next</strong>.</li> 
</ol> 
<div class="wp-caption aligncenter" id="attachment_32992" style="width: 710px;">
 <img alt="Grant access to principals in AWS RAM" class="wp-image-32992" height="337" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2022/09/21/cloudops_1069_15.png" width="700" />
 <p class="wp-caption-text" id="caption-attachment-32992">Figure 15. Grant access to principals in AWS RAM</p>
</div> 
<ol start="4"> 
 <li>At <strong>Review and create</strong>, select <strong>Create resource share</strong>.</li> 
 <li>From the <strong>Member Account</strong>, open the AWS RAM console and choose <strong>Resource shares under Shared with me</strong> and select <strong>Accept resource share</strong>.</li> 
</ol> 
<div class="wp-caption aligncenter" id="attachment_32993" style="width: 710px;">
 <img alt="Accept resource share in AWS RAM" class="wp-image-32993" height="261" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2022/09/21/cloudops_1069_16.png" width="700" />
 <p class="wp-caption-text" id="caption-attachment-32993">Figure 16. Accept resource share in AWS RAM</p>
</div> 
<h2>Part C – Configure Launch Template on Application Migration Service on the Member Account</h2> 
<p>This step will continue assuming that Application Migration Service is initialized and setup on the Member account of the Organization, the AWS Replication Agent is installed on the source server, and the Initial Sync is complete and you are ready to launch test instances.</p> 
<ol> 
 <li>On the <strong>Member Account</strong>. Open the <strong>AWS MGN</strong> console, choose <strong>Source servers</strong> on the left, and select the <strong>Source server name</strong>.</li> 
 <li>Select <strong>Launch settings</strong> and then select <strong>Modify</strong> under <strong>EC2 Launch Template</strong>.</li> 
</ol> 
<div class="wp-caption aligncenter" id="attachment_32994" style="width: 710px;">
 <img alt="Application Migration Service Launch Settings" class="wp-image-32994" height="556" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2022/09/21/cloudops_1069_17.png" width="700" />
 <p class="wp-caption-text" id="caption-attachment-32994">Figure 17. Application Migration Service Launch Settings</p>
</div> 
<ol start="3"> 
 <li>Enter the Launch template name and choose Don’t include in launch template under Application and OS Images (Amazon machine Image) since it will take this from Application Migration Service.</li> 
 <li>Continue filling in the rest of the contents where applicable until you’ve reached the Advanced details section to expand and specify the following settings:</li> 
</ol> 
<p style="padding-left: 40px;">Tenancy: <strong>Dedicated host launch this instance on a dedicated Host</strong></p> 
<p style="padding-left: 40px;">Target host by: <strong>Host resource group</strong></p> 
<p style="padding-left: 40px;">Tenancy host resource group: <strong>Select the ARN of the host resource group</strong></p> 
<p style="padding-left: 40px;">License configuration: <strong>Select the license configuration ARN from the drop-down</strong></p> 
<ol start="5"> 
 <li>Select Create template Version. You can also select this version as Default.</li> 
</ol> 
<div class="wp-caption aligncenter" id="attachment_32995" style="width: 710px;">
 <img alt="Configure EC2 Launch Template Advanced details" class="wp-image-32995" height="497" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2022/09/21/cloudops_1069_18.png" width="700" />
 <p class="wp-caption-text" id="caption-attachment-32995">Figure 18. Configure EC2 Launch Template Advanced details</p>
</div> 
<h2>Part D – Launch test instance from Application Migration Service on an Amazon EC2 Dedicated Host for your BYOL server.</h2> 
<ol> 
 <li>Open the Application Migration Service console and select the Source server name.</li> 
 <li>Select Test and cutover then select Launch test instances.</li> 
 <li>Once the test instance has launched successfully, you can check the Management Account’s Amazon EC2 Console at Dedicated Hosts to see that a Dedicated Host is allocated by the Host resource group for the test instance from the Member Account.</li> 
 <li>You may now view AWS License Manager Dashboard in the Management Account to track licensing rules enforced.</li> 
</ol> 
<h2>Cleaning up</h2> 
<p>To avoid incurring future charges, delete resources that we created during the walkthrough:</p> 
<ol> 
 <li>Archive the server(s) on Application Migration Service.</li> 
 <li>Terminate the instance(s) launched by Application Migration Service.</li> 
 <li>Delete the Host Resource Group on License Manager.</li> 
 <li>Delete the License Configuration on License Manager.</li> 
</ol> 
<h2>Conclusion</h2> 
<p>This post showed how customers could use License Manager to track (BYOL) Microsoft Windows Server Licenses for Windows Servers migrated to AWS using the Application Migration Service. Customers can take this a step further to track license utilization for applications such as Microsoft SQL Server, etc., by integrating License Manager with the Inventory capability of AWS Systems Manager.</p> 
<p><strong>About the authors:</strong></p> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="aligncenter size-full wp-image-11636" height="160" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2022/09/23/anutawsh.jpg" width="120" />
  </div> 
  <h3 class="lb-h4">Anutosh</h3> 
  <p>Anutosh is a Solutions Architect working with Enterprise segment at AWS. He loves to dive deep into his customer’s technical issues to help them navigate through their journey on AWS. He enjoys building solutions on migration and modernization on cloud. He is also passionate about Data analytics and Machine learning.</p> 
 </div> 
</footer> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="aligncenter wp-image-11636" height="160" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2022/09/23/kcomali.jpg" width="120" />
  </div> 
  <h3 class="lb-h4">Kyler Comalie</h3> 
  <p>Kyler is a Support Engineer at AWS. She has years of experience working on Microsoft Windows Platforms. When she is not helping customers succeed on the AWS Cloud, she will more likely be enjoying the outdoors, hiking or spending time with friends and family.</p> 
 </div> 
</footer> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="aligncenter wp-image-11636" height="160" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2022/09/23/Charles-Meruwoma.jpg" width="120" />
  </div> 
  <h3 class="lb-h4">Charles Meruwoma</h3> 
  <p>Charles Meruwoma is a Support Engineer with AWS. He works with customers across the globe to provide guidance in deploying and managing production workloads at scale on AWS. In his spare time, Charles enjoys learning as-well as seeing sci-fi and action movies.</p> 
 </div> 
</footer>
