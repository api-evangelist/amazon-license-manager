---
title: "Automating the discovery of licensed software using AWS License Manager"
url: "https://aws.amazon.com/blogs/mt/automating-the-discovery-of-licensed-software-using-aws-license-manager/"
date: "Sun, 07 Jun 2020 21:45:37 +0000"
author: "Harshitha Putta"
feed_url: "https://aws.amazon.com/blogs/mt/tag/aws-license-manager/feed/"
---
<p>Software license management often comes with the challenges of staying compliant, controlling overages, and managing vendor audits. Significant time and manual effort go into making sure that software license inventories are updated and ready for auditing. Bringing cloud infrastructure into the picture, with the ability to spin up virtual servers in minutes, means that managing licenses becomes even harder.</p> 
<p><a href="http://aws.amazon.com/license-manager" rel="noopener noreferrer" target="_blank">AWS License Manager</a> is a one-stop solution for managing licenses across a variety of software vendors such as Microsoft, Oracle, IBM, SAP, and others. It helps customers to centrally track software license usage across their AWS and on-premises environments. At re:Invent 2019, we announced a new capability in License Manager that helps customers automatically discover new bring your own license (BYOL) software in their existing environments.</p> 
<p>Previously, license administrators had to use a manual search-based mechanism to discover new licenses installed on their servers and instances. While this search-based mechanism provided customers the ability to centrally discover new licenses, customers asked AWS to reduce the manual effort for this. Now customers can specify the resources they’d like to track through their license rules. Based upon those rules, License Manager automatically discovers and accounts for licenses used in all of the matching environments. Administrators can also opt to receive notifications for any licensing rule violations.</p> 
<p>In this blog post, we walk you through the process of setting up an automated discovery of licensed software using License Manager.</p> 
<h3><strong>Prerequisites</strong></h3> 
<ul> 
 <li>Enable cross-account inventory discovery by linking your AWS Organizations accounts with the License Manager. For more information, see <a href="https://docs.aws.amazon.com/license-manager/latest/userguide/settings.html">settings in AWS License Manager</a>.<img alt="" class="aligncenter size-full wp-image-9724" height="800" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/06/05/1-6.png" width="964" /> </li> 
 <li>Make sure that the <a href="https://docs.aws.amazon.com/systems-manager/latest/userguide/ssm-agent.html">AWS Systems Manager agent</a> is installed and updated on all of the instances you want to track. Also make sure that instances are managed in <a href="https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-inventory.html">Systems Manager Inventory</a>. By running the latest version of Systems Manager agent, you ensure that you can collect metadata for all supported inventory types. For information about how to update Systems Manager agent by using <a href="https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-state.html">State Manager</a>, see <a href="https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-state-cli.html">automatically update Systems Manager agent</a>.</li> 
 <li>When you’ve installed the Systems Manager agent on your <a href="https://aws.amazon.com/ec2/">Amazon EC2 instances</a>, or on any on-premises servers associated with your AWS account, these instances are called <a href="https://docs.aws.amazon.com/systems-manager/latest/userguide/managed_instances.html">managed instances</a>. A managed instance is any Amazon EC2 instance or server or virtual machine (VM) in your on-premises environment that has been configured for Systems Manager. If you want to track on-premises servers, make sure these are managed instances.</li> 
</ul> 
<h3>Option 1: Setting up automated discovery from within a license configuration</h3> 
<h4>a. Create a license configuration</h4> 
<p>Now that we’ve got our desired EC2 instances and on-premises servers set up with the Systems Manager agent and discoverable in our inventory, let’s set up our <a href="https://docs.aws.amazon.com/license-manager/latest/userguide/license-configurations.html">license configurations</a>.</p> 
<p>The License Manager console provides range of features, including a list of license configurations. License configurations details include name, status, license type, licenses consumed, and account ID. Other information, such as the <a href="https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html">Amazon Resource Number (ARN)</a> or whether the license count is enforced, can be added by clicking the gear icon.</p> 
<p>In order to configure automated discovery of our licenses, we start with creating a license configuration. Here, we click on <strong>Create license configuration.</strong></p> 
<p><img alt="" class="aligncenter size-full wp-image-9726" height="493" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/06/05/2-1.png" width="956" /></p> 
<p>Make sure to review your license agreement terms carefully before creating a license configuration.</p> 
<p><strong>License configuration details</strong></p> 
<p>To create a license configuration, enter details, such as a name and optional description. You also have to specify how you want to track your license usage. This is important for BYOL licenses. You can find guidance for what option to use in your license agreement with your software vendor. There are a number of options, such as vCPUs, cores, sockets, or instances.</p> 
<p>You can specify if your license configuration should track instances with AWS-provided licenses. AWS-provided licenses are acquired when you launch a new EC2 instance using any AWS provided <a href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html">Amazon Machine Images (AMI)</a>. For example, when launching an instance using the AWS Windows AMI, AWS provides the license for that instance. This can be optionally tracked in License Manager. We track them in this blog.</p> 
<p><strong>Product information</strong></p> 
<p>On to the fun part: setting up the automated discovery. Adding product information here, lets you set up the automatic tracking of Windows and SQL Server licenses on instances and servers from within the license configuration setup. Here we’ve selected SQL Server 2017. The other options are as follows:</p> 
<p>Windows Server data center: 2008, 2008 R2, 2012, 2012 R2, 2016, 2019<br /> Windows Server Standard: 2008, 2008 R2, 2012, 2012 R2, 2016, 2019<br /> SQL Server: 2008, 2012, 2014, 2016, 2017<br /> Oracle Database Options and packs on Amazon RDS: Active Data Guard, Label Security, Oracle Online Analytical Processing (OLAP), Diagnostic Pack for SQLT, Tuning Pack for SQLT<br /> Oracle Database Editions on Amazon RDS: Standard Edition, Standard Edition One, Standard Edition Two, Enterprise Edition</p> 
<p>Don’t worry if you are not able to find your product here. These are only shortcuts to help to get set up quickly. If you want License Manager to track a product that is not listed here, finish creating this license configuration without specifying a product, and then add the product information through <a href="https://docs.aws.amazon.com/license-manager/latest/userguide/inventory.html">Search inventory</a> experience. We’ll walk through how to do that in the next section.</p> 
<p><img alt="" class="aligncenter wp-image-9753" height="673" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/06/06/Screen-Shot-2020-06-05-at-8.48.04-PM-1024x861.png" width="800" /></p> 
<p>&nbsp;</p> 
<p><img alt="" class="aligncenter wp-image-9754" height="301" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/06/06/Screen-Shot-2020-06-05-at-8.50.48-PM.png" width="800" /></p> 
<p>&nbsp;</p> 
<p><img alt="" class="aligncenter size-full wp-image-9728" height="362" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/06/05/4-1.png" width="1068" /></p> 
<h4>b. Check the consumed licenses in the license configurations</h4> 
<p>You can view the total licenses consumed for individual configurations under your list of license configurations.</p> 
<p><img alt="" class="aligncenter size-full wp-image-9729" height="268" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/06/05/5.png" width="997" /></p> 
<h4>c. Detailed license usage information in the License Manager dashboard</h4> 
<p>You can also view more detailed information of license usage through the built-in dashboard. Here you can see an overview of all of the license configurations you’ve created, license consumption, and enforcements.</p> 
<p><img alt="" class="aligncenter wp-image-9730" height="496" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/06/05/6.png" width="800" /></p> 
<p>If you click on any of the license configurations, you can get more specific information about your configurations. This information includes product details, associated AMIs, usage, tracked resources, and association errors.</p> 
<p><strong>Tracked resources</strong></p> 
<p>Under tracked resources, you can find resources in your AWS account that match your product information. Tracked resources screen shows you the list of resources that are consuming licenses from your license configuration. The tracked resources list is useful to provide evidence of license usage at the time of licensing true-ups. You can see the details such as how many licenses are consumed, which AWS account the resource belongs to, and when you began tracking it.</p> 
<p><strong>Association errors</strong></p> 
<p>License Manager helps you understand the status of your resources by identifying association errors between resources and license configurations. A couple of association error examples are:</p> 
<p>1. Exceeding license usage means that a particular resource cannot be associated with a license configuration. For example, you’ve associated 10 out of 10 SQL Server licenses. But License Manager has discovered another SQL Server resource in your inventory, it creates an association error that says there is a resource that cannot be associated with that license configuration. This provides option for you to take appropriate action.</p> 
<p>2. The resource with the association error does not match the license configuration rules. The resource has an <a href="https://docs.aws.amazon.com/license-manager/latest/userguide/licenses-to-rules.html">allowed tenancy</a> of shared, but the license configuration rule only manages licenses for resources with an allowed tenancy of Dedicated Host.</p> 
<p>The list of errors helps you identify violations quickly and take appropriate steps to rectify the problem. These steps could include updating licensing count in the license configuration after getting more licenses, or stopping over usage of licenses. The association errors list also helps you easily identify any error caused due to wrong setup or permissions, so that you can correct them.</p> 
<p><img alt="" class="aligncenter wp-image-9731" height="588" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/06/05/7.png" width="800" /></p> 
<p>&nbsp;</p> 
<h3>Option 2: Setting up automated discovery using the Search inventory</h3> 
<p>Creating license configurations provides a quick and efficient way to specify certain products for automated discovery. These products currently include Windows Server Standard, SQL Server, and Oracle Database on Amazon RDS. However, you can add any installed product to an existing license configuration using <a href="https://docs.aws.amazon.com/license-manager/latest/userguide/discovery.html#discovery-setup">License Manager’s Search inventory</a>.</p> 
<h3>a. Associate license configuration</h3> 
<p><strong>Search inventory</strong></p> 
<p>For this section of the blog, we use a license configuration that is already created.</p> 
<p>After you’ve successfully created your license configuration, click <strong>Search inventory</strong> in the left-hand menu. Search an installed product that you want to configure for an automated discovery. From the search results, select your desired EC2 instance or server and click the <strong>Associate license configuration</strong> button.</p> 
<p><img alt="" class="aligncenter size-full wp-image-9742" height="560" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/06/05/8-1.png" width="960" /></p> 
<p>&nbsp;</p> 
<p><strong>Associate license configuration</strong></p> 
<p>Click the drop-down menu and find the license configuration you’d like to associate the product information with. You see the product information for the instance or server underneath <strong>‘Add product information.’</strong> Click the radio button for the product.</p> 
<p><img alt="" class="wp-image-9761 aligncenter" height="407" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/06/06/Screen-Shot-2020-06-05-at-9.30.04-PM.png" width="600" /></p> 
<p>&nbsp;</p> 
<p><strong>Check the license usage on License Manager dashboard</strong></p> 
<p>You’ve successfully associated your desired product information with the license configuration. With that completed, License Manager tracks any managed instances or servers that you launch with that particular product information, as long as you have unused licenses available.</p> 
<p>You can check your license usage through the License Manager dashboard.</p> 
<p><img alt="" class="aligncenter wp-image-9735" height="501" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/06/05/10.png" width="800" /></p> 
<h3><strong>Cleaning up</strong></h3> 
<p>There is no additional charge for AWS License Manager. You pay for AWS resources (for example, EC2 instances) whose licenses are managed in License Manager. Make sure to delete any resources that you do not plan to use in the future to avoid incurring costs.</p> 
<h3><strong>Conclusion</strong></h3> 
<p>Using the automated discovery capability of License Manager makes tracking and managing software licenses efficient. License administrators can now have peace of mind knowing that all new installs of their products are automatically tracked by AWS License Manager. They can also benefit from the alerts, if there are any violations detected. For more information on managing licenses using License Manager, refer to the <a href="https://aws.amazon.com/license-manager/">service documentation</a>.</p> 
<p>&nbsp;</p> 
<h3>About the Authors</h3> 
<p><img alt="" class="alignleft size-thumbnail wp-image-9719" height="150" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/06/05/author-150x150.png" width="150" />Aminah Burch is a Solutions Architect on the US Central-Midwest Greenfield team. She enjoys helping customers get their journeys started on Amazon Web Services. She also enjoys creating music, writing, and JavaScript coding in her spare time.</p> 
<p>&nbsp;</p> 
<p>&nbsp;</p> 
<p>&nbsp;</p> 
<p><img alt="" class="alignleft wp-image-453 size-thumbnail" height="150" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2019/10/23/Harshitha.jpeg" width="150" /></p> 
<p>Harshitha Putta is a Cloud Infrastructure Architect with AWS Professional Services in Seattle, WA. She is passionate about building innovative solutions using AWS services to help customers achieve their business objectives. She enjoys spending time with family and friends, playing board games and hiking.</p>
