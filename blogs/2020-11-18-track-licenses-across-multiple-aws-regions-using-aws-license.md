---
title: "Track licenses across multiple AWS Regions using AWS License Manager"
url: "https://aws.amazon.com/blogs/mt/track-licenses-across-multiple-aws-regions/"
date: "Wed, 18 Nov 2020 20:40:11 +0000"
author: "Harshitha Putta"
feed_url: "https://aws.amazon.com/blogs/mt/tag/aws-license-manager/feed/"
---
<p>Are you a license administrator who wants to manage software licenses from different vendors as you build your cloud infrastructure across multiple AWS Regions? If so, you can use <a href="https://aws.amazon.com/license-manager/">AWS License Manager</a> to gain control and visibility into license usage. In this blog post, we discuss a solution that integrates AWS License Manager with <a href="https://aws.amazon.com/lambda/">AWS Lambda</a> to manage licenses used across different AWS Regions.</p> 
<h2>Solution overview</h2> 
<p>In each of the AWS Regions, you create a <a href="https://docs.aws.amazon.com/license-manager/latest/userguide/license-configurations.html">license configuration</a> that contains licensing rules based on the terms of your enterprise agreements.</p> 
<p>A Lambda function is used to call License Manager API operations, <a href="https://docs.aws.amazon.com/license-manager/latest/APIReference/API_GetLicenseConfiguration.html">GetLicenseConfiguration</a> and <a href="https://docs.aws.amazon.com/license-manager/latest/APIReference/API_ListUsageForLicenseConfiguration.html">ListUsageForLicenseConfiguration</a>, which provide license consumption details. This Lambda function is triggered at regular intervals based on your administration policy by an <a href="https://aws.amazon.com/eventbridge/">Amazon EventBridge</a> rule. That interval can be weekly, daily, or at any given time intervals. The result of the Lambda function is then stored in an <a href="https://aws.amazon.com/s3/">Amazon Simple Storage Service</a> (Amazon S3) bucket. The following figure shows the solution design.</p> 
<p style="text-align: center;"><img alt="License configurations in multiple Regions. Lambda function is triggered by an EventBridge time-based event to store license consumption details in an S3 bucket." class="aligncenter size-full wp-image-14546" height="502" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/11/17/Architecture.png" width="845" />Figure 1: Architecture diagram of multi-Region license tracking solution</p> 
<h3>Prerequisites</h3> 
<p>To deploy the solution described in this post, you need the following:</p> 
<ul> 
 <li><a href="https://docs.aws.amazon.com/license-manager/latest/userguide/license-configurations.html">License configurations</a> created in the AWS Regions you want to monitor licenses. Follow the steps in <a href="https://docs.aws.amazon.com/license-manager/latest/userguide/create-license-configuration.html">create a license configuration</a> in the AWS License Manager User Guide, and then copy the <a href="https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html">Amazon Resource Names</a> (ARNs) of these license configurations. You need them in the deployment section of this post.</li> 
 <li>Permissions required to deploy the <a href="https://aws.amazon.com/cloudformation/">AWS CloudFormation</a> template that automates the multi-Region license tracking solution.</li> 
</ul> 
<h3>Deployment steps</h3> 
<p>Use the AWS CloudFormation template to deploy the solution.</p> 
<p>1. In the AWS CloudFormation console, choose <strong>Create stack</strong>, and then choose the <strong>With new resources (standard)</strong> option.</p> 
<p style="text-align: center;"><img alt="" class="size-full wp-image-14548 aligncenter" height="140" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/11/17/Picture2-lm.png" width="936" />Figure 2: Stacks page in the AWS CloudFormation console</p> 
<p>2. Save the code snippet from <a href="https://github.com/aws-samples/aws-management-and-governance-samples/blob/master/AWSLicenseManager/TrackLicenses_MultiRegion/LM-MULTIREGION_LicenseTracking.yml">aws-samples repo</a> as a YAML file on your local computer.<br /> 3. &nbsp;On the <strong>Create stack</strong> page, choose <strong>Template is ready</strong>, choose <strong>Upload a template file</strong>, and then choose <strong>Next</strong>.<br /> 4. On the <strong>Specify stack details</strong> page, provide the following details, and then choose <strong>Next</strong>:</p> 
<ul> 
 <li><strong>Stack name</strong>: Enter a stack name for your AWS CloudFormation stack. In this post, we used MultiRegionLicenseTracking.</li> 
 <li><strong>LambdaTriggerSchedule</strong>: Enter the <a href="https://docs.aws.amazon.com/eventbridge/latest/userguide/scheduled-events.html">schedule expression for the EventBridge rule</a> based on your administration policy. We chose rate(1 day), which triggers the Lambda function daily. For more information, check <a href="https://docs.aws.amazon.com/eventbridge/latest/userguide/scheduled-events.html#cron-expressions">Cron Expressions</a> &nbsp;in the Amazon EventBridge documentation.</li> 
 <li><strong>LicenseConfigArns:</strong> Enter the list of license configuration ARNs, separated by comma.</li> 
</ul> 
<p style="text-align: center;"><img alt="MultiRegionLicenseTracking appears in the Stack name field. In the Parameters section, rate(1 day) appears in the LambdaTriggerSchedule field. In LicenseConfigArns, a list of ARNs is displayed, separated by commas." class="aligncenter size-full wp-image-14550" height="440" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/11/17/figure3.png" width="936" />Figure 3: Specify stack details page</p> 
<p>5. (Optional) Configure the fields for tags, permissions, and other advanced options. If no changes are required, use the defaults, and then choose <strong>Next</strong>.</p> 
<p style="text-align: center;"><img alt="Enabled is selected for the Rollback on failure option. Disabled is selected for the Termination protection option. The Timeout field is left blank." class="aligncenter size-full wp-image-14551" height="400" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/11/17/Picture4-lm.png" width="936" />Figure 4: Stack creation options</p> 
<p>6. Review the stack details and in the <strong>Capabilities</strong> section, select <strong>I acknowledge that AWS CloudFormation might create IAM resources</strong>, and then choose <strong>Create stack</strong>.</p> 
<p style="text-align: center;"><img alt="Stack creation options displays a summary. The IAM check box is selected." class="aligncenter size-full wp-image-14552" height="552" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/11/17/Picture5-lm.png" width="936" />Figure 5: IAM check box</p> 
<p>7. On the stack details page, you can check the status of your stack. When all the resources defined in the CloudFormation template have been created, <strong>CREATE_COMPLETE</strong> is displayed under <strong>Status</strong>.</p> 
<p style="text-align: center;"><img alt="On the MultiRegionLicenseTracking stack details page, the stack ID, description, status, and created time is displayed. The status is CREATE_COMPLETE." class="aligncenter size-full wp-image-14554" height="506" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/11/17/Figur6.png" width="936" />Figure 6: CREATE_COMPLETE</p> 
<p>8. Under Resources, you should find the following four resources have been created.</p> 
<ul> 
 <li><strong>LicenseTrackingLambdaFunction</strong>: The Lambda function that calls License Manager to fetch the license consumption of license configurations whose ARNs were provided as input to the CloudFormation parameter.</li> 
 <li><strong>LicenseManagerLambdaEventRule</strong>: The Amazon EventBridge rule that triggers the Lambda function on a periodic schedule.</li> 
 <li><strong>LambdaRole</strong>: The <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html">IAM role</a> with the permissions required for the Lambda function to make License Manager API calls and store the function results in an S3 bucket.</li> 
 <li><strong>LicenseConsumptionBucket</strong>: The S3 bucket used for storing multi-Region license consumption details.</li> 
</ul> 
<p style="text-align: center;"><img alt="Under Resources, LambdaRole, LicenseConsumptionBucket, LicenseManagerLambdaEventRule, and LicenseTrackingLambdaFunction are displayed." class="aligncenter size-full wp-image-14557" height="504" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/11/17/figr7.png" width="936" />Figure 7: Resources created by the AWS CloudFormation stack</p> 
<p>Now that the solution is deployed, let us discuss the data that is sent to the Amazon S3 bucket on every Lambda function run. When the EventBridge Rule triggers the Lambda function, the Lambda function creates the following folder structure and files in the S3 bucket.</p> 
<ul> 
 <li>A folder structure in the format YYYY/MM/DD/Timestamp for each run, as shown here:</li> 
</ul> 
<p style="text-align: center;"><img alt="License consumption report folder structure in format yyyy/mm/dd/timestamp. The page displays a Folder overview section (Region, URI, ARN) and an Objects section." class="aligncenter size-full wp-image-14558" height="556" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/11/17/figr8.png" width="936" />Figure 8: License consumption report folder structure</p> 
<ul> 
 <li>A license consumption summary file with details of all the license configurations that are sent as parameters to the CloudFormation template. In addition to the summary, for each of the license configurations, individual usage files are created that provide information of licenses consumed.</li> 
</ul> 
<p><img alt="In the Objects section, .csv files (individual usage files) are displayed." class="aligncenter size-full wp-image-14559" height="332" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/11/17/figr9.png" width="936" /></p> 
<p style="text-align: center;">Figure 9: License consumption reports from multiple AWS Regions</p> 
<p>You can use these details to track consumption across AWS Regions and assess your license monitoring needs.</p> 
<h2>Cleanup</h2> 
<p>To avoid ongoing charges, <a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-delete-stack.html">delete the CloudFormation stack</a>. That deletes the resources deployed by the CloudFormation template. You must empty the S3 bucket before you can delete it.</p> 
<p>There is no additional charge for using License Manager. You pay only for the AWS resources that are managed by License Manager, based on the AWS pricing of the resources.</p> 
<h2>Conclusion</h2> 
<p>In this blog post, we showed you how to track licenses when your workloads are deployed in multiple AWS Regions. This solution makes it effortless to manage and monitor license administration with less operational overhead. For more information about license management solutions on AWS, check the<a href="https://aws.amazon.com/license-manager/"> AWS License Manager</a> documentation.</p> 
<p>&nbsp;</p> 
<h3>About the Authors</h3> 
<p><img alt="" class="alignleft wp-image-453 size-thumbnail" height="150" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/07/28/Webp.net-resizeimage.jpg" width="150" /></p> 
<p>Harshitha Putta is a Cloud Infrastructure Architect with AWS Professional Services in Seattle, WA. She is passionate about building innovative solutions using AWS services to help customers achieve their business objectives. She enjoys spending time with family and friends, playing board games and hiking.</p> 
<p>&nbsp;</p> 
<p>&nbsp;</p> 
<p><img alt="" class="alignleft wp-image-453 size-thumbnail" height="150" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/11/18/Untitled.png" width="150" /></p> 
<p>Kalhan Vundela is a Software Development Engineer with Amazon Braket service. He is passionate about developing solutions that solve customer challenges. Kalhan enjoys hiking, skiing, and cooking.</p>
