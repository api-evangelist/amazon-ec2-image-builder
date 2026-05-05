---
title: "Adding approval notifications to EC2 Image Builder before sharing AMIs"
url: "https://aws.amazon.com/blogs/compute/adding-approval-notifications-to-ec2-image-builder-before-sharing-amis-2/"
date: "Fri, 14 Oct 2022 17:41:35 +0000"
author: "Sheila Busser"
feed_url: "https://aws.amazon.com/blogs/compute/tag/ec2-image-builder/feed/"
---
<p><em>­­­­­This blog post is written by, Glenn Chia Jin Wee, Associate Cloud Architect, and Randall Han, Professional Services.</em></p> 
<p>You may be required to manually validate the <a href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html">Amazon Machine Image (AMI)</a> built from an <a href="https://aws.amazon.com/ec2/">Amazon Elastic Compute Cloud (Amazon EC2)</a> Image Builder pipeline before sharing this AMI to other AWS accounts or to an AWS organization. Currently, <a href="https://aws.amazon.com/image-builder/features/">Image Builder</a> provides an end-to-end pipeline that automatically shares AMIs after they’ve been built.</p> 
<p>In this post, we will walk through the steps to enable approval notifications before AMIs are shared with other AWS accounts. Image Builder supports automated <a href="https://docs.aws.amazon.com/imagebuilder/latest/userguide/how-image-builder-works.html">image testing</a> using test components. The recommended best practice is to automate test steps, however situations can arise where test steps become either challenging to automate or internal compliance policies mandate manual checks be conducted prior to distributing images. In such situations, having a manual approval step is useful if you would like to verify the AMI configuration before it is shared to other AWS accounts or an AWS Organization. A manual approval step reduces the potential for sharing an incorrectly configured AMI with other teams which can lead to downstream issues. This solution sends an email with a link to approve or reject the AMI. Users approve the AMI after they’ve verified that it is built according to specifications. Upon approving the AMI, the solution automatically shares it with the specified AWS accounts.</p> 
<h2>Overview<a href="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2022/09/27/figure1-architecture-diagram-1.png"><img alt="Architecture Diagram" class="aligncenter wp-image-18943 size-large" height="555" src="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2022/09/27/figure1-architecture-diagram-1-1024x555.png" width="1024" /></a></h2> 
<ol> 
 <li>In this solution, an Image Builder Pipeline is run that builds a Golden AMI in Account A. After the AMI is built, Image Builder publishes data about the AMI to an <a href="https://aws.amazon.com/sns/">Amazon Simple Notification Service (Amazon SNS)</a></li> 
 <li>The SNS Topic passes the data to an <a href="https://aws.amazon.com/lambda/">AWS Lambda</a> function that subscribes to it.</li> 
 <li>The Lambda function that subscribes to this topic retrieves the data, formats it, and then starts an SSM Automation, passing it the AMI Name and ID.</li> 
 <li>The first step of the SSM Automation is a manual approval step. The SSM Automation first publishes to an SNS Topic that has an email subscription with the Approver’s email. The approver will receive the email with a URL that they can click to approve the step.</li> 
 <li>The approval step defines a specific AWS Identity and Access Management (IAM) Role as an approver. This role has the minimum required permissions to approve the manual approval step. After performing manual tests on the Golden AMI, the Approver principal will assume this role.</li> 
 <li>After assuming this role, the approver will click on the approval link that was sent via email. After approving the step, an AWS Lambda Function is triggered.</li> 
 <li>This Lambda Function shares the Golden AMI with Account B and sends an email notifying the Target Account Recipients that the AMI has been shared.</li> 
</ol> 
<h2><strong>Prerequisites</strong></h2> 
<p>For this walkthrough, you will need the following:</p> 
<ul> 
 <li>Two <a href="https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Fportal.aws.amazon.com%2Fbilling%2Fsignup%2Fresume&amp;client_id=signup">AWS accounts</a> – one to host the solution resources, and the second which receives the shared Golden AMI. 
  <ul> 
   <li>In the account that hosts the solution, prepare an AWS Identity and Access Management (IAM) principal with the <strong>sts:AssumeRole </strong>permission. This principal must assume the IAM Role that is listed as an approver in the Systems Manager approval step. The ARN of this IAM principal is used in the AWS CloudFormation <strong>Approver</strong> parameter, This ARN is added to the trust policy of approval IAM Role.</li> 
   <li>In addition, in the account hosting the solution, ensure that the IAM principal deploying the CloudFormation template has the required permissions to create the resources in the stack.</li> 
  </ul> </li> 
 <li>A new <a href="https://aws.amazon.com/vpc/">Amazon Virtual Private Cloud (Amazon VPC)</a> will be created from the stack. Make sure that you have fewer than five VPCs in the selected Region.</li> 
</ul> 
<h2><strong>Walkthrough</strong></h2> 
<p>In this section, we will guide you through the steps required to deploy the Image Builder solution. The solution is deployed with CloudFormation.</p> 
<p>In this scenario, we deploy the solution within the approver’s account. The approval email will be sent to a predefined email address for manual approval, before the newly created AMI is shared to target accounts.</p> 
<p>The approver first assumes the approval IAM Role and then selects the approval link. This leads to the Systems Manager approval page. Upon approval, an email notification will be sent to the predefined target account email address, notifying the relevant stakeholders that the AMI has been successfully shared.</p> 
<p>The high-level steps we will follow are:</p> 
<ol> 
 <li>In Account A, deploy the provided AWS CloudFormation template. This includes an example Image Builder Pipeline, Amazon SNS topics, Lambda functions, and an SSM Automation Document.</li> 
 <li>Approve the SNS subscription from your supplied email address.</li> 
 <li>Run the pipeline from the Amazon EC2 Image Builder Console.</li> 
 <li>[Optional] To conduct manual tests, launch an Amazon EC2 instance from the built AMI after the pipeline runs.</li> 
 <li>An email will be sent to you with options to approve or reject the step. Ensure that you have assumed the IAM Role that is the approver before clicking the approval link that leads to the SSM console approval page.</li> 
 <li>Upon approving the step, an AWS Lambda function shares the AMI to the Account B and also sends an email to the target account email recipients notifying them that the AMI has been shared.</li> 
 <li>Log in to Account B and verify that the AMI has been shared.</li> 
</ol> 
<h3>Step 1: Deploy the AWS CloudFormation template</h3> 
<p>1. The CloudFormation template, template.yaml that deploys the solution can also found at this <a href="https://github.com/aws-samples/ec2-image-builder-send-approval-notifications-before-sharing-ami">GitHub</a> repository. Follow the instructions at the repository to deploy the stack.</p> 
<h3>Step 2: Verify your email address</h3> 
<ol> 
 <li>After running the deployment, you will receive an email prompting you to confirm the Subscription at the approver email address. Choose <strong>Confirm subscription</strong>.</li> 
</ol> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2022/10/13/figure2-email-to-confirm-subscription-2.png"><img alt="SNS Topic Subscription confirmation email" class="aligncenter wp-image-19032 size-large" height="279" src="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2022/10/13/figure2-email-to-confirm-subscription-2-1024x279.png" width="1024" /></a></p> 
<ol start="2"> 
 <li>This leads to the following screen, which shows that your subscription is confirmed.</li> 
</ol> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2022/10/13/figure3-subscription-confirmation-2.png"><img alt="subscription-confirmation" class="aligncenter wp-image-19033 size-full" height="248" src="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2022/10/13/figure3-subscription-confirmation-2.png" width="539" /></a></p> 
<ol start="3"> 
 <li>Repeat the previous 2 steps for the target email address.</li> 
</ol> 
<h3>Step 3: Run the pipeline from the Image Builder console</h3> 
<ol> 
 <li>In the Image Builder console, under <strong>Image pipelines</strong>, select the checkbox next to the Pipeline created, choose <strong>Actions</strong>, and select <strong>Run pipeline.</strong></li> 
</ol> 
<p><em> <a href="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2022/09/27/figure4-run-image-builder-pipeline-1.png"><img alt="run-image-builder-pipeline" class="aligncenter wp-image-18947 size-full" height="368" src="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2022/09/27/figure4-run-image-builder-pipeline-1.png" width="1001" /></a></em></p> 
<p>Note: The pipeline takes approximately 20 – 30 minutes to complete.</p> 
<h3>Step 4: [Optional] Launch an Amazon EC2 instance from the built AMI</h3> 
<p>If you have a requirement to manually validate the AMI before sharing it with other accounts or to the AWS organization an approver will launch an Amazon EC2 instance from the built AMI and conduct manual tests on the EC2 instance to make sure it is functional.</p> 
<ol> 
 <li>In the Amazon EC2 console, under <strong>Images</strong>, choose <strong>AMIs</strong>. Validate that the AMI is created.</li> 
</ol> 
<p><em><a href="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2022/09/27/figure5-ami-in-account-a-1.png"><img alt="ami-in-account-a" class="aligncenter wp-image-18948 size-full" height="145" src="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2022/09/27/figure5-ami-in-account-a-1.png" width="748" /></a></em></p> 
<ol start="2"> 
 <li>Follow <a href="https://aws.amazon.com/premiumsupport/knowledge-center/launch-instance-custom-ami/">AWS docs: Launching an EC2 instances from a custom AMI</a> for steps on how to launch an Amazon EC2 instance from the AMI.</li> 
</ol> 
<h3>Step 5: Select the approval URL in the email sent</h3> 
<ol> 
 <li>When the pipeline is run successfully, you will receive another email with a URL to approve the AMI.</li> 
</ol> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2022/09/27/figure6-approval-email-1.png"><img alt="approval-email" class="aligncenter wp-image-18949 size-large" height="465" src="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2022/09/27/figure6-approval-email-1-1024x465.png" width="1024" /></a></p> 
<ol start="2"> 
 <li>Before clicking on the <strong>Approve</strong> link, you must assume the IAM Role that is set as an approver for the Systems Manager step.</li> 
 <li>In the CloudFormation console, choose the stack that was deployed.</li> 
</ol> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2022/09/27/figure7-cloudformation-stack-1.png"><img alt="cloudformation-stack" class="aligncenter wp-image-18950 size-large" height="263" src="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2022/09/27/figure7-cloudformation-stack-1-1024x263.png" width="1024" /></a></p> 
<p style="padding-left: 40px;">4. Choose <strong>Outputs</strong> and copy the IAM Role name.</p> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2022/09/27/figure8-ssm-approval-role-output-1.png"><img alt="ssm-approval-role-output" class="aligncenter wp-image-18951 size-large" height="284" src="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2022/09/27/figure8-ssm-approval-role-output-1-1024x284.png" width="1024" /></a></p> 
<p style="padding-left: 40px;">5. While logged in as the IAM Principal that has permissions to assume the approval IAM Role, follow the instructions at <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-console.html">AWS IAM documentation for switching a role using the console</a> to assume the approval role.<br /> In the Switch Role page, in <strong>Role</strong> paste the name of the IAM Role that you copied in the previous step.</p> 
<p style="padding-left: 40px;">Note: This IAM Role was deployed with minimum permissions. Hence, seeing warning messages in the console is expected after assuming this role.</p> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2022/09/27/figure9-switch-role-1.png"><img alt="switch-role" class="aligncenter wp-image-18952 size-large" height="465" src="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2022/09/27/figure9-switch-role-1-1024x465.png" width="1024" /></a></p> 
<p style="padding-left: 40px;">6. Now in the approval email, select the Approve URL. This leads to the Systems Manager console. Choose <strong>Submit</strong>.</p> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2022/10/13/figure10-approve-console-2.png"><img alt="approve-console" class="aligncenter wp-image-19034 size-large" height="604" src="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2022/10/13/figure10-approve-console-2-1024x604.png" width="1024" /></a></p> 
<p style="padding-left: 40px;">7. After approving the manual step, the second step is executed, which shares the AMI to the target account.</p> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2022/10/13/figure11-automation-step-success-2.png"><img alt="automation-step-success" class="aligncenter wp-image-19035 size-large" height="595" src="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2022/10/13/figure11-automation-step-success-2-1024x595.png" width="1024" /></a></p> 
<h3>Step 6: Verify that the AMI is shared to Account B</h3> 
<ol> 
 <li>Log in to Account B.</li> 
 <li>In the Amazon EC2 console, under <strong>Images</strong>, choose <strong>AMIs</strong>. Then, in the dropdown, choose <strong>Private images</strong>. Validate that the AMI is shared.</li> 
</ol> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2022/09/27/figure12-verify-ami-in-account-b-1.png"><img alt="verify-ami-in-account-b" class="aligncenter wp-image-18955 size-full" height="146" src="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2022/09/27/figure12-verify-ami-in-account-b-1.png" width="753" /></a></p> 
<ol start="3"> 
 <li>Verify that a success email notification was sent to the target account email address provided.</li> 
</ol> 
<h2><a href="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2022/09/27/figure13-target-email-1.png"><img alt="target-email" class="aligncenter wp-image-18956 size-large" height="352" src="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2022/09/27/figure13-target-email-1-1024x352.png" width="1024" /></a></h2> 
<h2><strong>Clean up</strong></h2> 
<p>This section provides the necessary information for deleting various resources created as part of this post.</p> 
<ol> 
 <li>Deregister the AMIs that were created and shared. 
  <ol> 
   <li>Log in to Account A and follow the steps at <a href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/deregister-ami.html">AWS documentation: Deregister your Linux AMI</a>.</li> 
  </ol> </li> 
 <li>Delete the CloudFormation stack. For instructions, refer to <a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-delete-stack.html">Deleting a stack on the AWS CloudFormation console</a>.</li> 
</ol> 
<h2><strong>Conclusion</strong></h2> 
<p>In this post, we explained how to enable approval notifications for an Image Builder pipeline before AMIs are shared to other accounts. This solution can be extended to share to more than one AWS account or even to an AWS organization. With this solution, you will be notified when new golden images are created, allowing you to verify the accuracy of their configuration before sharing them to for wider use. This reduces the possibility of sharing AMIs with misconfigurations that the written tests may not have identified.</p> 
<p>We invite you to experiment with different AMIs created using Image Builder, and with different Image Builder components. Check out <a href="https://github.com/aws-samples/amazon-ec2-image-builder-samples">this GitHub repository for various examples that use Image Builder</a>. Also check out this blog on <a href="https://aws.amazon.com/blogs/compute/introducing-instance-refresh-for-ec2-auto-scaling/">Image builder integrations with EC2 Auto Scaling Instance Refresh</a>. Let us know your questions and findings in the comments, and have fun!</p>
