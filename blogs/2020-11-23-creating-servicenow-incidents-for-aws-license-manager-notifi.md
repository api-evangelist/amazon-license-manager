---
title: "Creating ServiceNow incidents for AWS License Manager notifications"
url: "https://aws.amazon.com/blogs/mt/servicenow-incidents-for-license-manager/"
date: "Mon, 23 Nov 2020 23:18:40 +0000"
author: "Shashiraj Jeripotula"
feed_url: "https://aws.amazon.com/blogs/mt/tag/aws-license-manager/feed/"
---
<p id="ObU9CAP5UCC"><a href="https://aws.amazon.com/license-manager/">AWS License Manager</a><span style="color: #333333;">&nbsp;streamlines the process of managing software licenses from software vendors like Microsoft, </span><span style="color: #333333;">Oracle,</span><span style="color: #333333;"> IBM, SAP</span><span style="color: #333333;">,</span><span style="color: #333333;"> and others across AWS and </span><span style="color: #333333;">in </span><span style="color: #333333;">on-premise</span><span style="color: #333333;">s environments</span><span style="color: #333333;">. Administrators can create customized licensing rules that AWS License </span><span style="color: #333333;">Manager enforces</span><span style="color: #333333;"> when </span><a href="https://aws.amazon.com/ec2/">Amazon Elastic Compute Cloud</a> <span style="color: #333333;">(Amazon </span><span style="color: #333333;">EC2</span><span style="color: #333333;">)</span><span style="color: #333333;"> instances are launched. This help</span><span style="color: #333333;">s</span><span style="color: #333333;"> you prevent licensing violations by stopping the instances from launching or by notifying administrators. Administrators </span><span style="color: #333333;">have</span><span style="color: #333333;"> complete visibility of their licenses</span><span style="color: #333333;">.</span><span style="color: #333333;"> License Manager reduces the risk of non</span><span style="color: #333333;">compliance, misreporting</span><span style="color: #333333;">,</span><span style="color: #333333;"> and additional costs due to licensing overages.</span></p> 
<p id="ObU9CAzbtLQ"><a href="http://www.servicenow.com/" rel="noopener noreferrer" target="_blank">ServiceNow</a> <span style="color: #333333;">is an </span><a href="https://aws.amazon.com/partners/">AWS Partner Network</a> <span style="color: #000000;">(APN) Advanced Technology Partner with AWS </span><span style="color: #000000;">Public Sector </span><span style="color: #000000;">Competencies</span><span style="color: #333333;">. </span>ServiceNow gives enterprises complete visibility of their entire IT environment, including virtualized and cloud infrastructure. ServiceNow also simplifies service mapping, delivery, and assurance, consolidating IT service and infrastructure data into a single system of record.</p> 
<p id="ObU9CAmRkKP">The IT Service Management (ITSM) solution from ServiceNow can be used to log incidents, classify them by impact and urgency, assign them to appropriate groups, and escalate, resolve, and report them. Now customers can associate License Manager notifications and alerts with ITSM.</p> 
<p id="ObU9CAcWi33">In this blog post, I show you how to set up, configure, and send License Manager notifications and alerts to ITSM using <a href="https://aws.amazon.com/sns/">Amazon Simple Notification Service</a> (Amazon SNS). I also discuss how to test your configuration setup with a sample EC2 instance.</p> 
<h2 id="ObU9CAiF688">Services used in this post</h2> 
<p id="ObU9CAm5GOF"><a href="https://aws.amazon.com/license-manager/">AWS License Manager</a></p> 
<p id="ObU9CAetZ3M"><a href="https://aws.amazon.com/sns/">Amazon SNS</a></p> 
<p id="ObU9CAwrlDj"><a href="https://aws.amazon.com/ec2">Amazon EC2</a></p> 
<p id="ObU9CAe0WL2"><a href="http://servicenow.com/" rel="noopener noreferrer" target="_blank">ServiceNow</a></p> 
<h2 id="ObU9CA4pxqm">Prerequisites</h2> 
<p id="ObU9CAB2QwA">To follow the steps in this post, you need the following:</p> 
<div class=""> 
 <ul id="ObU9CA0NJo8"> 
  <li class="" id="ObU9CA5VZP0" value="1">An <a href="https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Fportal.aws.amazon.com%2Fbilling%2Fsignup%2Fresume&amp;client_id=signup">AWS account</a> </li> 
  <li class="" id="ObU9CANKiLG">Access to AWS License Manager with permissions to manage applications.</li> 
  <li class="" id="ObU9CAJ2lNc">A ServiceNow account with permissions to configure ITSM.</li> 
 </ul> 
</div> 
<h2 id="ObU9CALHi6h">Step 1: Configure ServiceNow</h2> 
<p id="ObU9CALq0Ug"><span style="color: #333333;">If you already have</span><span style="color: #333333;"> a free ServiceNow developer instance</span><span style="color: #333333;">, </span><span style="color: #333333;">you can</span><span style="color: #333333;"> use </span><span style="color: #333333;">it</span><span style="color: #333333;">.</span><span style="color: #333333;"> Otherwise, follow these steps to create one.</span></p> 
<div class=""> 
 <ul> 
  <li> <span style="color: #333333;">Sign</span><span style="color: #333333;"> in to</span> <span style="color: #333333;">the</span><span style="color: #0000ff;">&nbsp;</span><a href="https://developers.service-now.com/">ServiceNow developer site</a><span style="color: #333333;"> and request a developer instance</span><span style="color: #333333;">.</span> </li> 
  <li> <span style="color: #333333;">Sign</span><span style="color: #333333;"> in to the developer instance as </span><span style="color: #333333;">an </span><span style="color: #333333;">administrator</span><span style="color: #333333;">.</span> <span style="color: #333333;">Be</span><span style="color: #333333;"> sure to remember </span><span style="color: #333333;">your credentials</span> <span style="color: #333333;">because you’ll need them later</span><span style="color: #333333;"> when </span><span style="color: #333333;">you </span><span style="color: #333333;">configur</span><span style="color: #333333;">e</span><span style="color: #333333;"> SNS topic subscription URLs.</span> </li> 
  <li>Clone <a href="https://github.com/byukich/x_snc_aws_sns">this GitHub repository</a><span style="color: #333333;"> into your own GitHub account as a&nbsp;private&nbsp;repo (</span><span style="color: #333333;">for </span><span style="color: #333333;">exampl</span><span style="color: #333333;">e</span><span style="color: #333333;">, </span><span style="color: #333333;">https://github.com/shashivj/x_snc_aws_sns</span><span style="color: #333333;">). Go to </span><b><span style="color: #333333;">Profile</span></b><span style="color: #333333;">, choose </span><b><span style="color: #333333;">Your Own Repositories</span></b><span style="color: #333333;">, choose </span><b><span style="color: #333333;">New</span></b><span style="color: #333333;">, and then c</span><span style="color: #333333;">lick </span><span style="color: #333333;">the </span><span style="color: #333333;">link to import </span><span style="color: #333333;">the </span><span style="color: #333333;">repository. </span> </li> 
  <li> <span style="color: #333333;">In ServiceNow, </span><span style="color: #333333;">go</span><span style="color: #333333;"> to </span><b><span style="color: #333333;">System Applications</span></b><span style="color: #333333;">, choose</span> <b><span style="color: #333333;">Studio</span></b><span style="color: #333333;">, and </span><span style="color: #333333;">then</span> <span style="color: #333333;">choose</span> <b><span style="color: #333333;">Import From Source Control</span></b><span style="color: #333333;">.</span> </li> 
  <li> <span style="color: #333333;">On </span><b><span style="color: #333333;">Import </span></b><b><span style="color: #333333;">Application</span></b><span style="color: #333333;">, enter the URL of </span><span style="color: #333333;">the newly created repo</span><span style="color: #333333;">, your user name and password, and then choose</span> <b><span style="color: #333333;">Import</span></b><span style="color: #333333;">.</span> </li> 
 </ul> 
</div> 
<div class=""> 
 <div class="wp-caption alignnone" id="attachment_14257" style="width: 987px;">
  <img alt="Import Application in ServiceNow includes fields for network protocol (HTTPS or SSH), URL, branch, MID server, user name, and password." class="wp-image-14257 size-full" height="527" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/11/12/Picture1-6.png" width="977" />
  <p class="wp-caption-text" id="caption-attachment-14257">Figure 1: Import Application in ServiceNow</p>
 </div> 
</div> 
<p id="ObU9CA6A1u8"><span style="color: #333333;">If the import is successful, the </span><span style="color: #333333;">a <strong>Success</strong></span><span style="color: #333333;"> message is displayed</span><span style="color: #333333;">.</span><i></i></p> 
<div class=""></div> 
<div class=""> 
 <ul> 
  <li><span style="color: #333333;">Close the Studio browser tab. </span></li> 
  <li> <span style="color: #333333;">Refresh your ServiceNow browser tab, and </span><span style="color: #333333;">in the </span><span style="color: #333333;">search</span> <span style="color: #333333;">box</span><span style="color: #333333;">, search</span><span style="color: #333333;"> for “SNS”. </span><span style="color: #333333;">You</span><span style="color: #333333;"> can find</span><span style="color: #333333;"> three new navigation links </span><span style="color: #333333;">i</span><span style="color: #333333;">n the left pane</span><span style="color: #333333;">.</span> <span style="color: #333333;">I</span><span style="color: #333333;">n the </span><span style="color: #333333;">following</span><span style="color: #333333;"> image</span><span style="color: #333333;">, AWS SNS</span><span style="color: #333333;"> refers to the app name</span><span style="color: #333333;">,</span> <span style="color: #333333;">not to Amazon SNS</span><span style="color: #333333;">.</span> </li> 
 </ul> 
</div> 
<div class=""> 
 <div class="wp-caption alignnone" id="attachment_14259" style="width: 270px;">
  <img alt="SNS is entered in the search field. Under AWS SNS, there are entries for Subscriptions, Handlers, and Log." class="wp-image-14259 size-medium" height="300" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/11/12/Picture3-4-260x300.png" width="260" />
  <p class="wp-caption-text" id="caption-attachment-14259">Figure 2: ServiceNow search Navigator</p>
 </div> 
</div> 
<h2 id="ObU9CAEUYL4">Step 2: Create an SNS topic and subscription</h2> 
<div class=""> 
 <ul id="ObU9CAtyZAK"> 
  <li class="" id="ObU9CAuXrjN" value="1"> <span style="color: #333333;">Sign</span><span style="color: #333333;"> in to the </span><a href="https://console.aws.amazon.com/sns/v2/home">Amazon SNS console</a><span style="color: #333333;">.</span><span style="color: #333333;">&nbsp;</span><span style="color: #005b86;">&nbsp;</span> </li> 
  <li class="" id="ObU9CA1Qd9B"> <span style="color: #333333;">&nbsp;</span><span style="color: #333333;">In the left navigation pane, c</span><span style="color: #333333;">hoose</span><span style="color: #333333;">&nbsp;</span><span style="color: #333333;"><b>Topics</b></span><span style="color: #333333;">, and then choose </span><span style="color: #333333;"><b>Create new topic</b></span><span style="color: #333333;">.</span> </li> 
  <li class="" id="ObU9CA8QmfB"> <span style="color: #333333;">Enter</span><span style="color: #333333;"> a name and display name</span><span style="color: #333333;">, “</span><span style="color: #333333;">aws-license-manager-service-admin-notifications”</span><span style="color: #333333;">, for your topic</span><span style="color: #333333;">.</span> <span style="color: #333333;">License Manager req</span><span style="color: #333333;">u</span><span style="color: #333333;">ires the topic name in this format</span><span style="color: #333333;">:</span> <b><span style="color: #333333;">aws</span></b><b><span style="color: #333333;">-license-manager-</span></b><span style="color: #333333;">&lt;custom suffix&gt;</span><span style="color: #333333;">.</span> </li> 
  <li class="" id="ObU9CAg9XrS"> <span style="color: #333333;">Choose</span><span style="color: #333333;">&nbsp;</span><span style="color: #333333;"><b>Create Topic</b></span><span style="color: #333333;">.</span> </li> 
  <li class="" id="ObU9CADjhOJ"> <span style="color: #333333;">Choose </span><span style="color: #333333;">the <a href="https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html">Amazon Resource Name</a></span><span style="color: #333333;"> (ARN)</span><span style="color: #333333;">&nbsp;</span><span style="color: #333333;">link for the topic you just created. </span><span style="color: #333333;">Be sure to</span><span style="color: #333333;"> copy </span><span style="color: #333333;">the ARN</span><span style="color: #333333;"> because you will need it</span><span style="color: #333333;"> later</span><span style="color: #333333;">.</span> </li> 
  <li class="" id="ObU9CAstmrU"> <span style="color: #333333;">Choose</span><span style="color: #333333;">&nbsp;</span><span style="color: #333333;"><b>Create </b></span><span style="color: #333333;"><b>s</b></span><span style="color: #333333;"><b>ubscription</b></span><span style="color: #333333;">.</span> </li> 
  <li class="" id="ObU9CADvsT4"> <span style="color: #333333;">For </span><b><span style="color: #333333;">Protocol</span></b><span style="color: #333333;">, c</span><span style="color: #333333;">hoose</span><span style="color: #333333;">&nbsp;</span><span style="color: #333333;"><b>HTTPS</b></span><span style="color: #333333;">.</span> </li> 
  <li class="" id="ObU9CAANQEo"> <span style="color: #333333;">For</span><span style="color: #333333;">&nbsp;</span><span style="color: #333333;"><b>Endpoint</b></span><span style="color: #333333;"><b>,</b></span><span style="color: #333333;">&nbsp;</span><span style="color: #333333;">use the admin password </span><span style="color: #333333;">you </span><span style="color: #333333;">received </span><span style="color: #333333;">when you </span><span style="color: #333333;">request</span><span style="color: #333333;">ed the free ServiceNow developer instance</span><span style="color: #333333;">,</span><span style="color: #333333;"> and </span><span style="color: #333333;">then </span><span style="color: #333333;">choose the following link: https://admin:&lt;ServiceNow admin password&gt;@&lt;your developer instance&gt;.service-now.com/</span><span style="color: #333333;">api</span><span style="color: #333333;">/</span><span style="color: #333333;">x_snc_aws_sns</span><span style="color: #333333;">/</span><span style="color: #333333;">aws_sns</span> </li> 
  <li class="" id="ObU9CAtgi1n"> <span style="color: #333333;">Choose</span><span style="color: #333333;">&nbsp;</span><span style="color: #333333;"><b>Create subscription</b></span><span style="color: #333333;">.</span> </li> 
  <li class="" id="ObU9CAHLx2S"> <span style="color: #333333;">Under </span><b><span style="color: #333333;">Subscriptions</span></b><span style="color: #333333;">,</span> <span style="color: #333333;">you should see </span><b><span style="color: #333333;">Pending confirmation</span></b> <span style="color: #333333;">displayed next to your subscription</span><span style="color: #333333;">.</span> </li> 
 </ul> 
</div> 
<div class=""> 
 <div class="wp-caption alignnone" id="attachment_14244" style="width: 987px;">
  <img alt="Under Subscriptions, the status displayed for the subscription is Pending confirmation." class="wp-image-14244 size-full" height="289" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/11/12/Picture4-2.png" width="977" />
  <p class="wp-caption-text" id="caption-attachment-14244">Figure 3: Subscriptions tab of the Amazon SNS console</p>
 </div> 
</div> 
<h2 id="ObU9CAmMEEn">Step 3: Confirm Amazon SNS subscription on ServiceNow</h2> 
<p id="ObU9CAgkSYH"><span style="color: #333333;">Before</span><span style="color: #333333;"> Amazon</span><span style="color: #333333;"> SNS </span><span style="color: #333333;">can</span><span style="color: #333333;"> send messages to ServiceNow, you must confirm the subscription on ServiceNow. At this point, AWS has already sent a handshake request, and it</span> <span style="color: #333333;">is </span><span style="color: #333333;">awaiting confirmation </span><span style="color: #333333;">from</span><span style="color: #333333;"> your ServiceNow instance.</span></p> 
<div class=""> 
 <ul id="ObU9CA7OZyL"> 
  <li class="" id="ObU9CAj7yzZ" value="1"> <span style="color: #333333;">On your ServiceNow browser tab, </span><span style="color: #333333;">go</span><span style="color: #333333;"> to </span><b><span style="color: #333333;">SNS</span></b><span style="color: #333333;">, and then choose</span> <b><span style="color: #333333;">Subscriptions</span></b><span style="color: #333333;">.</span> <span style="color: #333333;">You should see th</span><span style="color: #333333;">at a new record has been created by AWS.</span> </li> 
 </ul> 
</div> 
<div class=""> 
 <div class="wp-caption alignnone" id="attachment_14251" style="width: 987px;">
  <img alt="The name, state, and subscribe URL are displayed." class="wp-image-14251 size-full" height="166" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/11/12/Picture5-4.png" width="977" />
  <p class="wp-caption-text" id="caption-attachment-14251">Figure 4: Subscription in ServiceNow</p>
 </div> 
</div> 
<div class=""> 
 <ul id="ObU9CAQ1m9g"> 
  <li class="" id="ObU9CAWpRYf" value="1">Choose the <b><span style="color: #16191f;"><span style="background-color: #ffffff;">aws</span></span></b><b><span style="color: #16191f;"><span style="background-color: #ffffff;">-license-manager-service-admin-notifications</span></span></b> <span style="color: #16191f;"><span style="background-color: #ffffff;">link </span></span><span style="color: #16191f;"><span style="background-color: #ffffff;">to open it, </span></span><span style="color: #16191f;"><span style="background-color: #ffffff;">and then choos</span></span><span style="color: #16191f;"><span style="background-color: #ffffff;">e</span></span> <b><span style="color: #16191f;"><span style="background-color: #ffffff;">Confirm Subscription</span></span></b><span style="color: #16191f;"><span style="background-color: #ffffff;">.</span></span> </li> 
 </ul> 
 <div class="wp-caption alignnone" id="attachment_14449" style="width: 987px;">
  <img alt="The subscription name is aws-license-manager-service-admin-notifications. Its topic ARN is displayed under Resource Names." class="wp-image-14449 size-full" height="337" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/11/13/Picture6-3.png" width="977" />
  <p class="wp-caption-text" id="caption-attachment-14449">Figure 5: Subscription confirmation in ServiceNow</p>
 </div> 
</div> 
<p id="ObU9CAXOYRM">Stay on this page because you will need to create a handler next.</p> 
<p id="ObU9CAwi9RB">When Amazon SNS sends an alarm, we want to open an incident when License Manager sends a notification. ServiceNow provides a handler script that is invoked when Amazon SNS sends an alarm message.</p> 
<div class=""> 
 <ul id="ObU9CAm1Iwy"> 
  <li class="" id="ObU9CAgvtB8" value="1">To configure a handler, on the <b>Subscription</b> page, in the <b>Handler</b><b>s</b> section, choose <b>New</b>.</li> 
  <li class="" id="ObU9CAymBox">Enter a name for the handler, such as “aws-license-manager-service-admin-notifi”.</li> 
  <li class="" id="ObU9CAltwEN">At line 3, inside the function, paste the following JavaScript code:<i><code></code></i> </li> 
 </ul> 
</div> 
<pre><code class="lang-js">var incident = new GlideRecord("incident");
	incident.initialize();
	incident.short_description = "License Manager Amazon SNS Alarm";
	incident.description = JSON.stringify(message);
	incident.insert(); 
</code></pre> 
<div class=""> 
 <div class="wp-caption alignnone" id="attachment_14260" style="width: 987px;">
  <img alt="name and JavaScript code." class="wp-image-14260 size-full" height="249" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/11/12/Picture7-3.png" width="977" />
  <p class="wp-caption-text" id="caption-attachment-14260">Figure 6: JavaScript code pasted at line 3</p>
 </div> 
</div> 
<div class=""> 
 <ul id="ObU9CAt3Ra4"> 
  <li class="" id="ObU9CA7tm0Z" value="1">Choose <b>Submit</b> to save the handler. Don’t close the browser. We will come back to it later.</li> 
 </ul> 
</div> 
<h2 id="ObU9CARlDMT">Step 4: Configure AWS License Manager</h2> 
<div class=""> 
 <ul id="ObU9CAOCwiv"> 
  <li class="" id="ObU9CAL1OL8" value="1">Sign in to the AWS License Manager console.</li> 
  <li class="" id="ObU9CAjDJTz">In the left navigation pane, choose <b>Settings</b>,<b> </b>and<b> </b>then choose <b>Edit</b>.</li> 
 </ul> 
</div> 
<div class=""> 
 <div class="wp-caption alignnone" id="attachment_14614" style="width: 987px;">
  <img alt="The Settings page displays account settings that include the account type, S3 bucket ARN, SNS topic ARN, resource share ARN, cross-account resource discovery, and link AWS Organizations accounts." class="wp-image-14614 size-full" height="254" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/11/18/Picture8-3.png" width="977" />
  <p class="wp-caption-text" id="caption-attachment-14614">Figure 7: Settings page of AWS License Manager console</p>
 </div> 
</div> 
<div class=""> 
 <ul id="ObU9CAVUdkf"> 
  <li class="" id="ObU9CA2avKt" value="1">Under <b>Simple Notification Service</b>, enter the SNS topic ARN created in Step 2.</li> 
 </ul> 
 <div class=""> 
  <div class="wp-caption alignnone" id="attachment_14253" style="width: 987px;">
   <img alt="The Settings page in the AWS License Manager console includes a Simple Notification Service section with am SNS topic ARN field." class="wp-image-14253 size-full" height="458" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/11/12/Picture9-2.png" width="977" />
   <p class="wp-caption-text" id="caption-attachment-14253">Figure 8: SNS topic ARN in AWS License Manager console</p>
  </div> 
 </div> 
</div> 
<div class=""> 
 <ul id="ObU9CAZLBwB"> 
  <li class="" id="ObU9CApVugC" value="1">Choose <b>Apply</b>.&nbsp; The success message will be displayed, “<strong>Your settings have been saved successfully.</strong>“</li> 
 </ul> 
</div> 
<div class=""> 
 <ul id="ObU9CAafHyt"> 
  <li class="" id="ObU9CA6XVbz" value="1">In the left navigation pane, choose <b>License configurations</b>.<b> </b> </li> 
  <li class="parent" id="ObU9CAV0Bsb">On the <b>Cre</b><b>ate </b><b>license configuration</b> page, do the following: 
   <ul> 
    <li class="" id="ObU9CANfbT3">Enter a name and optional description.</li> 
    <li class="" id="ObU9CACaKM8">From <b>License type</b>, choose <b>vCPUs</b>.</li> 
    <li class="" id="ObU9CASvSDB">For <b>Number of vCPUs</b>, enter <strong>4</strong>.</li> 
   </ul> </li> 
 </ul> 
</div> 
<div class=""> 
 <div class="wp-caption alignnone" id="attachment_14261" style="width: 978px;">
  <img alt="In Configuration details, there are fields for license configuration name, description, license type (in this example, vCPUs) and number of vCPUs (in this example, 4)." class="wp-image-14261 size-full" height="770" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/11/12/Picture11.png" width="968" />
  <p class="wp-caption-text" id="caption-attachment-14261">Figure 9: Details for the Linux Servers license configuration</p>
 </div> 
</div> 
<div class=""> 
 <ul id="ObU9CA0QX0s"> 
  <li class="" id="ObU9CA3IGSt" value="1">Keep the other fields blank, and then choose <b>Submit</b>. The success message will be displayed, “<strong>Linux Servers was successfully created</strong>.”</li> 
 </ul> 
</div> 
<h2 id="ObU9CAZavQ5">Step 5: Test License Manager notifications</h2> 
<p id="ObU9CACeVbJ">To test this integration, we create an EC2 instance and then use it to create an <a href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html">AMI</a>. We then associate this AMI with the license configuration (Linux servers) we created. Finally, we launch a new EC2 instance with this AMI with a vCPU of more than 4 so it triggers an Amazon SNS alarm. The alarm, in turn, creates an incident in the ITSM table.</p> 
<div class=""> 
 <ul id="ObU9CAihy3B"> 
  <li class="" id="ObU9CAauTFV" value="1"> <span style="color: #000000;">Sign</span><span style="color: #000000;"> in to the A</span><span style="color: #000000;">mazon EC2 </span><span style="color: #000000;">console</span><span style="color: #000000;">, and choose </span><b><span style="color: #000000;">Launch instance</span></b><span style="color: #000000;">.</span> </li> 
  <li class="" id="ObU9CA3pQnD">On <b>Choose</b><b> A</b><b>MI</b>, choose <b>Amazon Linux AMI</b>.</li> 
  <li class="" id="ObU9CArOMLg">On <b>Choose an Instance Type</b>, select t2.micro. A t2.micro instance type has 1 vCPUs and 1 GiB of memory.</li> 
 </ul> 
</div> 
<div class=""> 
 <div class="wp-caption alignnone" id="attachment_14450" style="width: 987px;">
  <img alt="The Choose an instance type page displays columns for Family, Type, vCPUs, Memory (GiB), Instance Storage (GB), EBS-Optimized Available, Network Performance, and IPv6 Support. Choose Review and Launch" class="wp-image-14450 size-full" height="435" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/11/13/Picture13-1.png" width="977" />
  <p class="wp-caption-text" id="caption-attachment-14450">Figure 10: Choose an Instance Type page of the Amazon EC2 console</p>
 </div> 
</div> 
<div class=""> 
 <ul id="ObU9CAd8pmN"> 
  <li class="" id="ObU9CAOI95c" value="1">Choose <b>Review and Launch</b>.</li> 
  <li class="" id="ObU9CAgasQp">On the <b>Review Instance Launch</b> page, choose <b>Launch</b>.</li> 
 </ul> 
</div> 
<div class=""></div> 
<div class=""> 
 <ul id="ObU9CAZKAmE"> 
  <li class="" id="ObU9CATK31U" value="1">On<b> Select an existing key pair or create a new key pair</b>, choose <b>Proceed without a key pair</b><b>.</b> Select the <b>I acknowledge that I will not be able to connect to this instance unless I already know the password build into this AMI</b> check box. Choose <b>Launch Instances</b>.</li> 
 </ul> 
</div> 
<div class=""> 
 <ul id="ObU9CA1S1Lh"> 
  <li class="" id="ObU9CAajNXX" value="1"> <b>Launch Status</b> page is displayed, choose <b>View Instances</b>.</li> 
 </ul> 
</div> 
<div class=""> 
 <ul id="ObU9CAVwrBF"> 
  <li class="" id="ObU9CAtxEjP" value="1">EC2 Instances page is displayed. After the instance is in a running state, choose the instance.</li> 
  <li class="" id="ObU9CAtDUJN">From the <b>Actions</b> menu, choose <b>Image</b>, and then choose <b>Create</b> <b>i</b><b>mage</b>.</li> 
 </ul> 
</div> 
<div class=""> 
 <div class="wp-caption alignnone" id="attachment_14270" style="width: 987px;">
  <img alt="On the Instances page, the running instance is selected." class="wp-image-14270 size-full" height="258" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/11/12/Picture17.png" width="977" />
  <p class="wp-caption-text" id="caption-attachment-14270">Figure 11: Instances page of the Amazon EC2 console</p>
 </div> 
</div> 
<div class=""> 
 <ul id="ObU9CAP01cX"> 
  <li class="" id="ObU9CAluKTq" value="1">In <b>Create image</b>, for <b>Image name</b>, enter “Linux Image”. Enter an optional description, and then choose <b>Create </b><b>i</b><b>mage</b>.</li> 
 </ul> 
</div> 
<div class=""> 
 <div class="wp-caption alignnone" id="attachment_14271" style="width: 987px;">
  <img alt="On Create image, there are fields for the image name (in this example, Linux Image) and an optional description (in this example, Linux Image for License Manager notifications)." class="wp-image-14271 size-full" height="543" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/11/12/Picture18.png" width="977" />
  <p class="wp-caption-text" id="caption-attachment-14271">Figure 12: Create image section of the Amazon EC2 console</p>
 </div> 
</div> 
<div class=""> 
 <ul id="ObU9CAXnrRN"> 
  <li class="" id="ObU9CADRk9I" value="1">If there are no errors, a success message is displayed, “<strong>Successfully created ami-xxxxx from instance i-xxxxxxx.</strong>“</li> 
 </ul> 
</div> 
<div class=""> 
 <ul id="ObU9CAKBCvi"> 
  <li class="" id="ObU9CAJinHe" value="1">In the left navigation pane, under <b>Images</b>, choose <b>AMIs</b>. You can find our new image, Linux Image displayed.</li> 
 </ul> 
</div> 
<div class=""> 
 <div class="wp-caption alignnone" id="attachment_14331" style="width: 987px;">
  <img alt="The Linux Image AMI is selected on the AMIs page." class="wp-image-14331 size-full" height="341" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/11/12/Picture20-1.png" width="977" />
  <p class="wp-caption-text" id="caption-attachment-14331">Figure 13: New image displayed on the AMIs page of the Amazon EC2 console</p>
 </div> 
</div> 
<p id="ObU9CAUXfau">Keep this browser open. We will come back to it later to launch the instance.</p> 
<div class=""> 
 <ul id="ObU9CApaW3N"> 
  <li class="" id="ObU9CA7KVQk" value="1">Open another browser window, navigate to the AWS License Manager console, and from the left navigation pane, choose <b>License configurations</b>.</li> 
 </ul> 
</div> 
<div class=""> 
 <ul id="ObU9CA577uI"> 
  <li class="" id="ObU9CAsKaqf" value="1">Choose <b>Linux Servers</b>, and from the <b>Actions</b> menu, choose <b>Associate AMI</b>.</li> 
 </ul> 
</div> 
<div class=""> 
 <div class="wp-caption alignnone" id="attachment_14275" style="width: 987px;">
  <img alt="Linux Servers is selected. The Actions menu displays options to Edit, Delete, Deactivate, and Associate AMI." class="wp-image-14275 size-full" height="200" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/11/12/Picture22.png" width="977" />
  <p class="wp-caption-text" id="caption-attachment-14275">Figure 14: License configurations page showing Actions menu</p>
 </div> 
</div> 
<div class=""> 
 <ul id="ObU9CAflh5Q"> 
  <li class="" id="ObU9CAMdrTL" value="1">On the <b>Associate Linux Servers with AMI</b> page, choose the Linux Image AMI, and then choose <b>Associate</b>.</li> 
 </ul> 
</div> 
<div class=""> 
 <div class="wp-caption alignnone" id="attachment_14276" style="width: 972px;">
  <img alt="On Associate Linux Servers with AMI page, the AMI is selected." class="wp-image-14276 size-full" height="252" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/11/12/Picture23.png" width="962" />
  <p class="wp-caption-text" id="caption-attachment-14276">Figure 15: Linux AMI image for License Manager notifications</p>
 </div> 
</div> 
<div class=""> 
 <ul id="ObU9CAIZJUK"> 
  <li class="" id="ObU9CAdu5UF" value="1">A success message is displayed, “<strong>Successfully Associated</strong>“.</li> 
 </ul> 
</div> 
<div class=""> 
 <ul id="ObU9CAwRjGr"> 
  <li class="" id="ObU9CAiJ2Gi" value="1">Go back to the Amazon EC2 console.</li> 
  <li class="" id="ObU9CAI4R5V">On the <b>AMIs</b> page, choose <b>Linux Image</b>, and then choose <b>Launch</b>.</li> 
 </ul> 
</div> 
<div class=""></div> 
<div> 
 <div class="wp-caption alignnone" id="attachment_14279" style="width: 987px;">
  <img alt="Linux Image is selected on the AMIs page." class="wp-image-14279 size-full" height="314" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/11/12/Picture25-1.png" width="977" />
  <p class="wp-caption-text" id="caption-attachment-14279">Figure 16: AMIs page with Linux Image selected</p>
 </div> 
</div> 
<div class=""> 
 <ul id="ObU9CAYGZOT"> 
  <li class="" id="ObU9CAy373y" value="1">On <b>Choose an Instance Type</b>, make sure <b>t</b><b>2.micro</b> is selected, and from the top of the page, choose <b>Configure Instance</b>.</li> 
 </ul> 
</div> 
<div class=""> 
 <div class="wp-caption alignnone" id="attachment_14281" style="width: 987px;">
  <img alt="On Choose an Instance Type page, A general purpose instance of the t2.micro type is selected." class="wp-image-14281 size-full" height="210" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/11/12/Picture26.png" width="977" />
  <p class="wp-caption-text" id="caption-attachment-14281">Figure 17: Choose an Instance Type page</p>
 </div> 
</div> 
<div class=""> 
 <ul id="ObU9CAmSTSd"> 
  <li class="" id="ObU9CACfAIg" value="1">On <b>Configure Instance Details</b>, for <b>Number of instances</b>, enter <strong>5</strong>,<b> </b>and then choose <b>Review and Launch</b>.</li> 
 </ul> 
</div> 
<div class=""> 
 <div class="wp-caption alignnone" id="attachment_14282" style="width: 987px;">
  <img alt="On the Configure Instance Details page, 5 is entered into the Number of Instances field." class="wp-image-14282 size-full" height="449" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/11/12/Picture27.png" width="977" />
  <p class="wp-caption-text" id="caption-attachment-14282">Figure 18: Configure Instance Details page</p>
 </div> 
</div> 
<div class=""> 
 <ul id="ObU9CA8eCII"> 
  <li class="" id="ObU9CAqxCKH" value="1">On<b> </b><b>Review Instance Launch</b>, choose <b>Launch</b>.</li> 
 </ul> 
</div> 
<div class=""> 
 <ul id="ObU9CAkRnUB"> 
  <li class="" id="ObU9CACymxY" value="1">On<b> Select an existing key pair or create a new key pair</b>, choose <b>Proceed without a key pair</b>. Then select the <b>I acknowledge that I will not be able to connect to this instance unless I already know the password build into this AMI</b> check box. Choose <b>Launch Instances</b>.</li> 
 </ul> 
</div> 
<div class=""> 
 <ul id="ObU9CAbhV6F"> 
  <li class="" id="ObU9CAXOrv3" value="1">On <b>Launch Status</b>, choose <b>View Instances</b>.</li> 
 </ul> 
</div> 
<div class=""> 
 <ul id="ObU9CAHvhCJ"> 
  <li class="" id="ObU9CAyCO6D" value="1">Check the <b>Instances</b> page to confirm that the instance is running.</li> 
 </ul> 
</div> 
<div class=""> 
 <div class="wp-caption alignnone" id="attachment_14286" style="width: 987px;">
  <img alt="Instances page displays the running instances." class="wp-image-14286 size-full" height="243" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/11/12/Picture31.png" width="977" />
  <p class="wp-caption-text" id="caption-attachment-14286">Figure 19: Instances page of the Amazon EC2 console</p>
 </div> 
</div> 
<div class=""> 
 <ul id="ObU9CAI6uXa"> 
  <li class="" id="ObU9CAfyraq" value="1"> <span style="color: #333333;">On your ServiceNow browser tab, </span><span style="color: #333333;">in the </span><span style="color: #333333;">search </span><span style="color: #333333;">field, enter “</span><span style="color: #333333;">incidents”</span><span style="color: #333333;">. </span><span style="color: #333333;">Under</span> <b><span style="color: #333333;">Service Desk</span></b><span style="color: #333333;">,</span> <span style="color: #333333;">choose </span><b><span style="color: #333333;">Incidents</span></b><span style="color: #333333;">.</span> </li> 
 </ul> 
</div> 
<div class="tall"> 
 <div class="wp-caption alignnone" id="attachment_14287" style="width: 208px;">
  <img alt="Incidents appears under Service Desk." class="wp-image-14287 size-medium" height="300" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/11/12/Picture32-198x300.png" width="198" />
  <p class="wp-caption-text" id="caption-attachment-14287">Figure 20: ServiceNow search Navigator</p>
 </div> 
</div> 
<div class=""> 
 <ul id="ObU9CALZr1C"> 
  <li class="" id="ObU9CAaUqc7" value="1">If a new incident appears, then you have successfully created a ServiceNow incident for the License Manager SNS alarm. You can define a workflow to route the incident to any person or group responsible for addressing it.</li> 
 </ul> 
</div> 
<div class=""> 
 <div class="wp-caption alignnone" id="attachment_14288" style="width: 987px;">
  <img alt="The License Manager SNS Alarm appears in the Incident table." class="wp-image-14288 size-full" height="102" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/11/12/Picture33.png" width="977" />
  <p class="wp-caption-text" id="caption-attachment-14288">Figure 21: License Manager SNS Alarm displayed as an incident in ServiceNow</p>
 </div> 
</div> 
<div class=""> 
 <ul id="ObU9CAMSMZ2"> 
  <li class="" id="ObU9CAwmfH7" value="1">Go back to the License Manager console and choose <b>Dashboard</b>.<b> </b>In the<b> Overview </b>section, you can find that there is one usage limit alert. Choose the <b>View exceeded license configurations</b> link.</li> 
 </ul> 
</div> 
<div class=""> 
 <div class="wp-caption alignnone" id="attachment_14289" style="width: 987px;">
  <img alt="The dashboard in the License Manager console displays a usage limit alert." class="wp-image-14289 size-full" height="370" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/11/12/Picture34.png" width="977" />
  <p class="wp-caption-text" id="caption-attachment-14289">Figure 22: Usage limit alert displayed in the AWS License Manager console</p>
 </div> 
</div> 
<div class=""> 
 <ul id="ObU9CA1id8m"> 
  <li class="" id="ObU9CAF0cqG" value="1">On<b> </b>the<b> License configurations </b>page,<b> </b>under <b>Licenses Consumed</b>, you can find 5 of 4 have been consumed.</li> 
 </ul> 
</div> 
<div class=""> 
 <div class="wp-caption alignnone" id="attachment_14290" style="width: 987px;">
  <img alt="Under License configurations, the Licenses Consumed column shows 5 of 4 licenses." class="wp-image-14290 size-full" height="183" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/11/12/Picture35.png" width="977" />
  <p class="wp-caption-text" id="caption-attachment-14290">Figure 23: Licenses Consumed displayed on the License configurations page</p>
 </div> 
</div> 
<h2 id="ObU9CA353kw">Conclusion</h2> 
<p id="ObU9CADxX5C">In this post, I showed you how to configure AWS License Manager to send notifications and alerts to ServiceNow using Amazon SNS.</p> 
<p id="ObU9CAKl3Im"><span style="color: #000000;">You can extend t</span><span style="color: #000000;">his setup to any </span><span style="color: #000000;">Amazon SNS</span><span style="color: #000000;"> topic that notifies ServiceNow whenever anything meaningful happens in your AWS Cloud environment.&nbsp;</span><span style="color: #000000;">I</span><span style="color: #000000;">n </span><span style="color: #000000;">the </span><span style="color: #000000;">ServiceNow </span><span style="color: #000000;">Amazon </span><span style="color: #000000;">SNS </span><span style="color: #000000;">h</span><span style="color: #000000;">andlers, you can create any type of ServiceNow record you like. It can </span><span style="color: #000000;">trigger</span><span style="color: #000000;"> an automated workflow or create </span><span style="color: #000000;">e</span><span style="color: #000000;">vents</span><span style="color: #000000;">, alerts</span><span style="color: #000000;">, or </span><span style="color: #000000;">n</span><span style="color: #000000;">otifications</span><span style="color: #000000;">. It can also</span><span style="color: #000000;">&nbsp;update </span><span style="color: #000000;">a configuration management database (</span><span style="color: #000000;">CMDB</span><span style="color: #000000;">),</span><span style="color: #000000;"> or even automatically orchestrate a remediation.</span></p> 
<p id="ObU9CAE7b9B">To learn more about AWS License Manager, check the&nbsp;<a href="https://aws.amazon.com/license-manager/">AWS License Manager</a><span style="color: #333333;">&nbsp;</span><span style="color: #333333;">documentation</span>. If you have any questions, post them on&nbsp;<a href="https://forums.aws.amazon.com/forum.jspa?forumID=30&amp;start=0">Amazon Elastic Compute (EC2) service forum</a>.</p> 
<h2 id="ObU9CA0xHPM">About the author</h2> 
<div class="">
 <img alt="" class="alignleft wp-image-14213 size-thumbnail" height="150" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2020/11/11/Shashi-Raj-Jeripotula-150x150.jpg" width="150" />
</div> 
<p id="ObU9CA5CMSx">Shashiraj Jeripotula(Raj) is a San Francisco-based Sr. Partner Solutions Architect at AWS. He works with various Independent Software Vendors(ISVs), and partners who specialize in Cloud Management Tools and DevOps to develop joint solutions and accelerate cloud adoption on AWS. While he’s not at work, Raj works with charities that provide education, food, and health support for kids.</p> 
<div class=""></div>
