---
title: "Deploying highly-available SQL Server on Amazon EC2 Dedicated Hosts"
url: "https://aws.amazon.com/blogs/mt/deploying-highly-available-sql-server-on-amazon-ec2-dedicated-hosts/"
date: "Thu, 16 Jun 2022 20:07:42 +0000"
author: "Yogi Barot"
feed_url: "https://aws.amazon.com/blogs/mt/tag/aws-license-manager/feed/"
---
<p>Want to bring your eligible SQL Server licenses to use on AWS? If your organization is planning data center evacuation, and looking to extend the life of existing investments in Microsoft SQL Server and Windows Server licenses, <a href="https://aws.amazon.com/ec2/">Amazon Elastic Compute Cloud (Amazon EC2)</a> and <a href="https://aws.amazon.com/license-manager/">AWS License Manager</a> can help.</p> 
<p>Do you also want to setup high availability for your SQL Server on Amazon EC2 Dedicated Hosts?</p> 
<p>In this post, we’ll show you how <a href="https://aws.amazon.com/ec2/dedicated-hosts/">Amazon EC2 Dedicated Hosts</a> and AWS License Manager can be used to design a highly-available Microsoft SQL Server architecture while saving costs by bringing your own licenses. You will understand how License Manager integrates with Dedicated Hosts to reduce management overhead. Furthermore, you’ll understand how host resource groups simplify the management of Dedicated Hosts while allowing you to meet the high availability requirements for the SQL Server.</p> 
<h2>Solution overview</h2> 
<p>When designing high availability for Dedicated Hosts within a single AWS Region, make sure that you’ve allocated Dedicated Hosts in a minimum of two Availability Zones as shown in the following diagram.</p> 
<p>AWS License Manager is used for creating a <a href="https://docs.aws.amazon.com/license-manager/latest/userguide/license-configurations.html">license configuration</a> that will track Windows and SQL Server licensing usage.&nbsp;<a href="https://aws.amazon.com/image-builder/">EC2 Image Builder</a> is used to build the AMI, and as part of the deployment, it associates the Amazon Machine Image (AMI) to the license configuration. Then, within License Manager, a host resource group is configured and associated with the license configuration. This will automatically handle the provisioning, releasing, and recovering of the underlying Dedicated Hosts.&nbsp; You will need to bring your Windows Server license in Bring your own licensing Model or use Windows License included instance on Dedicated Hosts.</p> 
<div class="wp-caption alignnone" id="attachment_28195" style="width: 1534px;">
 <img alt="Figure 1 High-level architecture showing SQL Server cluster nodes replicating synchronously between Availability Zones." class="size-full wp-image-28195" height="932" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2022/05/24/Figure-1-High-level-architecture-showing-SQL-Server-cluster-nodes-replicating-synchronously-between-Availability-Zones..jpg" width="1524" />
 <p class="wp-caption-text" id="caption-attachment-28195">Figure 1 High-level architecture showing SQL Server cluster nodes replicating synchronously between Availability Zones.</p>
</div> 
<h2>Solution workflow</h2> 
<p>This solution workflow consists of the following steps:</p> 
<ol> 
 <li>Create customer managed license configuration</li> 
 <li>Associate a license configuration with an AMI through EC2 image builder</li> 
 <li>Create a host resource group and associate the license configuration</li> 
 <li>Launch your SQL Server using Host resources groups</li> 
 <li>Setup SQL Server high availability using Always On availability group or SQL Server FCI with <a href="https://aws.amazon.com/fsx/">Amazon FSx</a>.</li> 
</ol> 
<h2>Prerequisites</h2> 
<p>To deploy this solution, the following is required:</p> 
<ul> 
 <li>SQL Server Enterprise Edition/Standard Edition core-based or processor-based licenses</li> 
 <li>Access to <a href="https://aws.amazon.com/console/">AWS Management Console</a></li> 
 <li>·License Manager</li> 
 <li>Amazon Dedicated Hosts</li> 
 <li>Amazon FSx for Windows</li> 
</ul> 
<h3>1.Create License Manager configuration</h3> 
<p>License Manager emulates licensing terms as rules. With License configuration, you’ll create licensing rules based on the licensing terms in your enterprise agreements. The rules shown in the following diagram are an example scenario of license configuration to bring four Dedicated Hosts of r5 Instance family that you can license at the core level or vCPU level with the license type.&nbsp;While creating license configurations, work closely with your organization’s compliance team to review your enterprise agreements.</p> 
<div class="wp-caption alignnone" id="attachment_28196" style="width: 1372px;">
 <img alt="Figure 2 Creating AWS License Manager configuration rules with parameter and rules to track license usage." class="size-full wp-image-28196" height="662" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2022/05/24/Figure-2-Creating-AWS-License-Manager-configuration-rules-with-parameter-and-rules-to-track-license-usage..jpg" width="1362" />
 <p class="wp-caption-text" id="caption-attachment-28196">Figure 2 Creating AWS License Manager configuration rules with parameter and rules to track license usage.</p>
</div> 
<h3>License Configuration settings</h3> 
<ul> 
 <li>License Type : Core</li> 
 <li>License count : 192</li> 
 <li>License count hard limit : True</li> 
</ul> 
<h3>Rule type:</h3> 
<ul> 
 <li>License affinity to host (in days) : 90 days</li> 
 <li>Tenancy : Dedicated Host</li> 
</ul> 
<p>To learn more about parameters and rules, see this <a href="https://docs.aws.amazon.com/license-manager/latest/userguide/config-overview.html">document</a>.</p> 
<ol start="2"> 
 <li>Associate AMI to license configurations</li> 
</ol> 
<p>Now that the license configurations are set up, you must associate the AMI. This can either be done directly through the license configuration, or as part of EC2 Image Builder, which simplifies the process for building and maintaining secure images.</p> 
<h3>Prepare AMI for the BYOL use case</h3> 
<p>For BYOL, you must import and license your own media. You can use EC2 Image Builder or EC2 VM Import/Export to import your VM images.</p> 
<div class="wp-caption alignnone" id="attachment_28197" style="width: 1760px;">
 <img alt="Figure 3 Creating pipeline to build Windows AMI’s for running SQL Server Cluster nodes." class="size-full wp-image-28197" height="608" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2022/05/24/Figure-3-Creating-pipeline-to-build-Windows-AMI’s-for-running-SQL-Server-Cluster-nodes..jpg" width="1750" />
 <p class="wp-caption-text" id="caption-attachment-28197">Figure 3 Creating pipeline to build Windows AMI’s for running SQL Server Cluster nodes.</p>
</div> 
<p>If you already <a href="https://docs.aws.amazon.com/imagebuilder/latest/userguide/start-build-image-pipeline.html">build your AMIs with EC2 Image Builder</a>, and have an existing pipeline, then follow these steps.</p> 
<p>Associate the AMI using EC2 Image Builder</p> 
<ol> 
 <li>Open the <a href="https://console.aws.amazon.com/imagebuilder">EC2 Image Builder console</a>.</li> 
 <li>Under Distribution Settings, open the settings of the target AMI pipeline (the one that you want to associate with EC2 Image builder), and select edit.</li> 
 <li>Under “Associate License Configurations,” add the two license configurations that you created earlier.</li> 
 <li>OR</li> 
</ol> 
<p>Associate the AMI directly using License Manager</p> 
<ol> 
 <li>In the <a href="https://console.aws.amazon.com/license-manager">License Manager console</a>, open your license configuration for Windows Server.</li> 
 <li>Select “Associate AMI” under the “Associated AMI” tab.</li> 
</ol> 
<div class="wp-caption alignnone" id="attachment_28198" style="width: 1674px;">
 <img alt="Figure 4 Associating license configuration rules with Windows AMI to enforce licensing terms and usage." class="size-full wp-image-28198" height="650" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2022/05/24/Figure-4-Associating-license-configuration-rules-with-Windows-AMI-to-enforce-licensing-terms-and-usage..jpg" width="1664" />
 <p class="wp-caption-text" id="caption-attachment-28198">Figure 4 Associating license configuration rules with Windows AMI to enforce licensing terms and usage.</p>
</div> 
<ol start="3"> 
 <li>Create Dedicated Host resource group</li> 
</ol> 
<p>With the host resource group, host management tasks such as allocating, releasing, and recovery is taken care of for you by the service.</p> 
<p>Dedicated Host resource group is a logical collection of Dedicated Hosts that you manage as a single entity.&nbsp;Host resource group can be created with specific preferences, such as instance family and allowed license configuration.</p> 
<p>In a practical sense, this could represent a logical group of Dedicated Hosts of a similar instance family, i.e., a Dedicated Hosts group to run SQL Workload on r5 instances only. You could also create a host resource group with more relaxed preferences, such as including multiple instance family, and&nbsp;associate more than one license configuration.&nbsp;This simplifies instance placement on the Dedicated Hosts by automatically allocating the right type of Dedicated Hosts for your workload.</p> 
<p>For running the SQL cluster, you should configure the host resource group with precise settings that allow only the SQL Server license configuration rule to stay compliant, and pick r5 as your preferred Instance family.</p> 
<div class="wp-caption alignnone" id="attachment_28199" style="width: 1602px;">
 <img alt="Figure 5 Selecting Host management preferences within the Host resource group to manage Dedicated Hosts." class="size-full wp-image-28199" height="602" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2022/05/24/Figure-5-Selecting-Host-management-preferences-within-the-Host-resource-group-to-manage-Dedicated-Hosts..jpg" width="1592" />
 <p class="wp-caption-text" id="caption-attachment-28199">Figure 5 Selecting Host management preferences within the Host resource group to manage Dedicated Hosts.</p>
</div> 
<h3>Dedicated Host Management settings</h3> 
<p style="padding-left: 40px;">Auto-allocate : True</p> 
<p style="padding-left: 40px;">Auto-release : True</p> 
<p style="padding-left: 40px;">Auto-recover : True</p> 
<p style="padding-left: 40px;">Allowed Instance families : r5</p> 
<p>To learn more about host resource group, see this <a href="https://docs.aws.amazon.com/license-manager/latest/userguide/host-resource-groups.html">document</a>.</p> 
<p>Implement high availability for Dedicated Hosts environment</p> 
<p>Apart from leveraging Multi-AZ architecture, Amazon EC2 Dedicated Hosts offer a host recovery feature for supported instance families.&nbsp;Host recovery uses host-level health checks to assess Dedicated Host availability and detect underlying system failures.&nbsp;When a system failure is detected on your Dedicated Host, host recovery is initiated, and Amazon EC2 automatically moves the instances to the new host.</p> 
<p>Note that host recovery is not supported on instances with instance storage, such as r5d and z1d, where manual recovery is needed for replacement of the Dedicated Host.</p> 
<div class="wp-caption alignnone" id="attachment_28203" style="width: 1390px;">
 <img alt="Figure 6 High-level architecture showing Dedicated Hosts using two availability zones." class="size-full wp-image-28203" height="858" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2022/05/24/Figure-6-High-level-architecture-showing-Dedicated-Hosts-using-two-availability-zones.-1.jpg" width="1380" />
 <p class="wp-caption-text" id="caption-attachment-28203">Figure 6 High-level architecture showing Dedicated Hosts using two availability zones.</p>
</div> 
<h2>Implement high availability for Microsoft SQL Server environment</h2> 
<p>Microsoft SQL Server provides two deployment options of its Always On solution for business continuity use cases, such as high availability and disaster recovery: Always On Failover Cluster Instances (FCI) and Always On Availability Groups (AG). Both of these deployment options use Windows Server Failover Cluster (WSFC) technology to provide a cluster of independent nodes that work together to deliver high availability, but there are important differences:</p> 
<ul> 
 <li>AGs represent one or more user databases that fail over together across multiple replicas hosted in the cluster to provide database-level high availability. An AG consists of a primary replica and one or more secondary replicas that are maintained through SQL Server log-based data movement for data protection without the need for shared storage. Each replica is hosted by an instance of SQL Server on a different node of the WSFC, and each replica has its own local storage independent of the AG.</li> 
</ul> 
<div class="wp-caption alignnone" id="attachment_28202" style="width: 1748px;">
 <img alt="Figure 7 High-level architecture showing SQL Server Always on cluster nodes replicating synchronously between Availability Zones." class="size-full wp-image-28202" height="1066" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2022/05/24/Figure-7-High-level-architecture-showing-SQL-Server-Always-on-cluster-nodes-replicating-synchronously-between-Availability-Zones..jpg" width="1738" />
 <p class="wp-caption-text" id="caption-attachment-28202">Figure 7 High-level architecture showing SQL Server Always on cluster nodes replicating synchronously between Availability Zones.</p>
</div> 
<ul> 
 <li>You can create a Primary SQL Server Amazon EC2 instance on Dedicated Hosts in availability zone 1 on r5 Dedicated hosts, and a secondary SQL Server Amazon EC2 instance can be deployed on Dedicated Hosts in availability zone 2. You can setup an Always On availability group between two instances of SQL Server.</li> 
 <li>Check here for how to setup <a href="https://aws.amazon.com/premiumsupport/knowledge-center/ec2-windows-sql-server-always-on-cluster/">SQL Server Always On availability group</a>.</li> 
 <li>And the differences for the Always On FCIs with FSx are as follows:</li> 
 <li>FCIs present a single SQL Server instance that’s installed across nodes in a WSFC to provide high availability for the entire installation of SQL Server. This means that everything inside of the SQL Server instance moves to another node in the cluster should the underlying node encounter a problem. Among other things, system databases, SQL Server logins, SQL Server Agent jobs, and certificates get moved to another node. FCIs require some form of shared storage – either shared block storage (SAN) or shared file storage (accessed via SMB).</li> 
 <li>Amazon FSx provides you with fully managed shared file storage that automatically replicates the storage synchronously across two Availability Zones. Moreover, Amazon FSx provides high availability with automatic failure detection, failover, and failback. The service also fully supports the SMB Continuous Availability (CA) feature required to support SQL Server Always On FCI deployments.</li> 
</ul> 
<div class="wp-caption alignnone" id="attachment_28201" style="width: 1738px;">
 <img alt="Figure 8 High-level architecture showing SQL Server Failover Cluster Instance nodes between Availability Zones." class="size-full wp-image-28201" height="1058" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2022/05/24/Figure-8-High-level-architecture-showing-SQL-Server-Failover-Cluster-Instance-nodes-between-Availability-Zones..jpg" width="1728" />
 <p class="wp-caption-text" id="caption-attachment-28201">Figure 8 High-level architecture showing SQL Server Failover Cluster Instance nodes between Availability Zones.</p>
</div> 
<ul> 
 <li>You can create a Primary SQL Server Amazon EC2 instance on Dedicated Hosts in Availability Zone 1 on r5 Dedicated hosts, and secondary SQL Server Amazon EC2 instance can be deployed on Dedicated Hosts in Availability Zone 2. You can setup Windows failover cluster and SQL Server Failover cluster using FSx between two instances of SQL Server.</li> 
 <li>Check how to setup <a href="https://aws.amazon.com/blogs/storage/simplify-your-microsoft-sql-server-high-availability-deployments-using-amazon-fsx-for-windows-file-server/">SQL Server FCI with FSx implementation</a>.</li> 
</ul> 
<h2>Clean up</h2> 
<p>When you’re finished, you can <a href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/terminating-instances.html">terminate the Amazon EC2 instances</a> that you launched earlier. This will <a href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/how-dedicated-hosts-work.html#dedicated-hosts-releasing">automatically release the Dedicated Hosts</a>. There is no additional charge for using License Manager. You pay only for the resources that you create to run your applications, such as Amazon EC2 instances.</p> 
<h2>Conclusion</h2> 
<p>In this post, you have learned how Dedicated Hosts can be used with License Manager to save costs by utilizing your existing licensing and deploying a highly-available SQL Server environment. We showed how to setup SQL Server high availability using Always On availability group and SQL Server FCI deployments using Amazon FSx. We also showed how you could use license configurations, host resource groups, and EC2 image builder to reduce management overhead and easily track software license usage. This significantly improves your overall experience of BYOL Windows, and SQL Server licenses to AWS cloud that stay compliant.</p> 
<p>Check the&nbsp;<a href="https://docs.aws.amazon.com/prescriptive-guidance/latest/sql-server-ec2-best-practices/welcome.html">best practices document</a> to learn more about the most effective ways for working with SQL Server on Amazon EC2.</p> 
<p><strong>Authors:</strong></p> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="aligncenter size-full wp-image-11636" height="194" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2022/06/16/Yogi-Barot.jpg" width="169" />
  </div> 
  <h3 class="lb-h4">Yogi Barot</h3> 
  <p>Yogi is Principal Solutions Architect who has 22 years of experience working with different Microsoft technologies, her specialty is in SQL Server and different database technologies. Yogi has in depth AWS knowledge and expertise in running Microsoft workload on AWS.</p> 
 </div> 
</footer>
