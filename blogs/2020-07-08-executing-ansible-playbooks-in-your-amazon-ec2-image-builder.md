---
title: "Executing Ansible playbooks in your Amazon EC2 Image Builder pipeline"
url: "https://aws.amazon.com/blogs/compute/executing-ansible-playbooks-in-your-amazon-ec2-image-builder-pipeline/"
date: "Wed, 08 Jul 2020 22:00:25 +0000"
author: "Chris Munns"
feed_url: "https://aws.amazon.com/blogs/compute/tag/ec2-image-builder/feed/"
---
<p><em>This post is contributed by Andrew Pearce – Sr. Systems Dev Engineer, AWS</em></p> 
<p>Since launching <a href="https://aws.amazon.com/image-builder/" rel="noopener noreferrer" target="_blank">Amazon EC2 Image Builder</a>, many customers say they want to re-use existing investments in configuration management technologies (such as <a href="https://www.ansible.com/" rel="noopener noreferrer" target="_blank">Ansible</a>, <a href="https://www.chef.io/home/" rel="noopener noreferrer" target="_blank">Chef</a>, or <a href="https://puppet.com/" rel="noopener noreferrer" target="_blank">Puppet</a>) with Image Builder pipelines.</p> 
<p>In this blog, I&nbsp;walk through creating a <a href="https://docs.aws.amazon.com/imagebuilder/latest/userguide/image-builder-application-documents.html" rel="noopener noreferrer" target="_blank">document</a> that can execute an Ansible playbook in an Image Builder pipeline. I then use the document to create an Image Builder component that can be added to an Image Builder image recipe.&nbsp;In addition to this blog, you can find further samples in the <a href="https://github.com/aws-samples/amazon-ec2-image-builder-samples" rel="noopener noreferrer" target="_blank">amazon-ec2-image-builder-samples</a> GitHub repository.</p> 
<h2>Ansible playbook requirements</h2> 
<p>There are likely some changes required so your existing Ansible playbooks can work in Image Builder.</p> 
<p>When executing Ansible within Image Builder, the playbook must be configured to execute using the localhost. In a traditional Ansible environment, the control server executes playbooks against remote hosts using a protocol like SSH or WinRM. In Image Builder, the host executing the playbook is also the host that must be configured.</p> 
<p>There are a number of ways to accomplish this in Ansible. In this example, I set the value of hosts&nbsp;to&nbsp;127.0.0.1, gather_facts to false, and connection to local. These values set execution at the localhost, prevent gathering Ansible facts about remote hosts, and force local execution.</p> 
<p>In this walk through, I use the following sample playbook. This installs Apache, configures a default webpage, and ensures that Apache is enabled and running.</p> 
<pre><code class="lang-yaml">---
- name: Apache Hello World
  hosts: 127.0.0.1
  gather_facts: false
  connection: local
  tasks:
    - name: Install Apache
      yum:
        name: httpd
        state: latest
    - name: Create a default page
      shell: echo "&lt;h1&gt;Hello world from EC2 Image Builder and Ansible&lt;/h1&gt;" &gt; /var/www/html/index.html
    - name: Enable Apache
      service: name=httpd enabled=yes state=started</code></pre> 
<h2>Creating the Ansible document</h2> 
<p>This example uses the <strong>build</strong> phase to install Ansible, download the playbook from an S3 bucket, execute the playbook and cleanup the system. It also ensures that Apache is returning the correct content in both the <strong>validate</strong> and <strong>test</strong> phases.</p> 
<p>I use Amazon Linux 2 as the target operating system, and&nbsp;assume that the playbook is already uploaded to my S3 bucket.</p> 
<p><strong>Document design diagram</strong></p> 
<p><img alt="Document Design Diagram" class="aligncenter size-full wp-image-10464" height="422" src="https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2020/07/08/doc-design-diagram.png" width="864" /></p> 
<p>To start, I must specify the top-level properties for the document. I specify a <strong>name</strong> and <strong>description</strong>&nbsp;to describe the document’s purpose, and a <strong>schemaVersion</strong>, which at the time of writing is <strong>1.0</strong>. I also include a <strong>build</strong> phase, as I want this document to be used when building the image.</p> 
<pre><code class="lang-yaml">name: 'Ansible Playbook Execution on Amazon Linux 2'
description: 'This is a sample component that demonstrates how to download and execute an Ansible playbook against Amazon Linux 2.'
schemaVersion: 1.0
phases:
  - name: build
    steps:</code></pre> 
<p>The first required step is to install Ansible. With Amazon Linux 2, I can install Ansible using <a href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/amazon-linux-ami-basics.html#extras-library" rel="noopener noreferrer" target="_blank">Amazon Linux Extras</a>, so I use the <strong>ExecuteBash</strong> action to perform the work.</p> 
<pre><code class="lang-yaml">     - name: InstallAnsible
        action: ExecuteBash
        inputs:
          commands:
           - sudo amazon-linux-extras install -y ansible2</code></pre> 
<p>Once Ansible is installed, I must download the playbook for execution. I use the <strong>S3Download</strong> action to download the playbook and store it in a temporary location. Note that the S3Download action accepts a list of inputs, so a single S3Download step could download multiple files.</p> 
<pre><code class="lang-yaml">     - name: DownloadPlaybook
        action: S3Download
        inputs:
          - source: 's3://mybucket/my-playbook.yml'
            destination: '/tmp/my-playbook.yml'</code></pre> 
<p>Now Ansible is installed and the playbook is downloaded. I use the <strong>ExecuteBinary</strong> action to invoke Ansible and perform the work described in the playbook. This step uses a feature of the component management application called <a href="https://docs.aws.amazon.com/imagebuilder/latest/userguide/image-builder-application-documents.html" rel="noopener noreferrer" target="_blank">chaining</a>.</p> 
<p>Chaining allows you to reference the input or output values of a step in subsequent steps. In the following example, <em>{{build.DownloadPlaybook.inputs[0].destination}}</em> passes the string of the first “destination” property value in the DownloadPlaybook step, which executes in the build phase. The <em>“[0]”</em> indicates the first value in the array (position zero).</p> 
<p>For more information about how chaining works, review the <a href="https://docs.aws.amazon.com/imagebuilder/latest/userguide/image-builder-component-manager.html" rel="noopener noreferrer" target="_blank">Component Manager documentation</a>. In this example, I refer to the S3Download destination path to avoid mistyping the value, which causes the build to fail.</p> 
<pre><code class="lang-yaml">      - name: InvokeAnsible
        action: ExecuteBinary
        inputs:
          path: ansible-playbook
          arguments:
            - '{{build.DownloadPlaybook.inputs[0].destination}}'</code></pre> 
<p>Finally, I clean up the system, so I use another <strong>ExecuteBash</strong> action combined with chaining to delete the downloaded playbook.</p> 
<pre><code class="lang-yaml">      - name: DeletePlaybook
        action: ExecuteBash
        inputs:
          commands:
            - rm '{{build.DownloadPlaybook.inputs[0].destination}}'</code></pre> 
<p>Now, I have installed Ansible, downloaded the playbook, executed it, and cleaned up the system. The next step is to add a&nbsp;<strong>validate</strong> phase where I use <strong>curl</strong> to ensure that Apache responds with the desired content. Including a validate phase allows you to identify build issues more quickly. This fails the build before Image Builder creates an AMI and moves to the test phase.</p> 
<p>In the <strong>validate</strong> phase, I use <strong>grep</strong> to confirm that Apache returned a value containing my required string. If the command fails, a non-zero exit code is returned, indicating a failure. This failure is returned to Image Builder and the image build is marked as “Failed”.</p> 
<pre><code class="lang-yaml">  - name: validate
    steps:
      - name: ValidateResponse
        action: ExecuteBash
        inputs:
          commands:
            - curl -s http://127.0.0.1 | grep "Hello world from EC2 Image Builder and Ansible"</code></pre> 
<p>After the validate phase executes, the EC2 instance is stopped and an Amazon Machine Image (AMI) is created. Image Builder moves to the test stage where a new EC2 instance is launched using the AMI that was created in the previous step. During this stage, the test phase of a document is executed.</p> 
<p>To ensure that Apache is still functioning as required, I add a <strong>test</strong>&nbsp;phase to the document using the same <strong>curl</strong> command as the <strong>validate</strong> phase. This ensures that Apache is started and still returning the correct content.</p> 
<pre><code class="lang-yaml">  - name: test
    steps:
      - name: ValidateResponse
        action: ExecuteBash
        inputs:
          commands:
            - curl -s http://127.0.0.1 | grep "Hello world from EC2 Image Builder and Ansible"</code></pre> 
<p>Here is the complete document:</p> 
<pre><code class="lang-yaml">name: 'Ansible Playbook Execution on Amazon Linux 2'
description: 'This is a sample component that demonstrates how to download and execute an Ansible playbook against Amazon Linux 2.'
schemaVersion: 1.0
phases:
  - name: build
    steps:
      - name: InstallAnsible
        action: ExecuteBash
        inputs:
          commands:
           - sudo amazon-linux-extras install -y ansible2
      - name: DownloadPlaybook
        action: S3Download
        inputs:
          - source: 's3://mybucket/playbooks/my-playbook.yml'
            destination: '/tmp/my-playbook.yml'
      - name: InvokeAnsible
        action: ExecuteBinary
        inputs:
          path: ansible-playbook
          arguments:
            - '{{build.DownloadPlaybook.inputs[0].destination}}'
      - name: DeletePlaybook
        action: ExecuteBash
        inputs:
          commands:
            - rm '{{build.DownloadPlaybook.inputs[0].destination}}'
  - name: validate
    steps:
      - name: ValidateResponse
        action: ExecuteBash
        inputs:
          commands:
            - curl -s http://127.0.0.1 | grep "Hello world from EC2 Image Builder and Ansible"
  - name: test
    steps:
      - name: ValidateResponse
        action: ExecuteBash
        inputs:
          commands:
            - curl -s http://127.0.0.1 | grep "Hello world from EC2 Image Builder and Ansible"</code></pre> 
<p>I now have a document that installs Ansible, downloads a playbook from an S3 bucket, and invokes Ansible to perform the work described in the playbook. I am also validating the installation was successful by ensuring Apache is returning the desired content.</p> 
<p>Next, I use this document to create a component in Image Builder and assign the component to an image recipe. The image recipe could then be used when creating an image pipeline. The blog post <a href="https://aws.amazon.com/blogs/aws/automate-os-image-build-pipelines-with-ec2-image-builder/" rel="noopener noreferrer" target="_blank">Automate OS Image Build Pipelines with EC2 Image Builder</a> provides a tutorial demonstrating how to create an image pipeline with the EC2 Image Builder console. For more information, review <a href="https://docs.aws.amazon.com/imagebuilder/latest/userguide/getting-started-image-builder.html" rel="noopener noreferrer" target="_blank">Getting started with EC2 Image Builder</a>.</p> 
<h2>Conclusion</h2> 
<p>In this blog, I walk through creating a document that can execute Ansible playbooks for use in Image Builder. I define the required changes for Ansible playbooks and when each phase of the document would execute. I also note the <a href="https://github.com/aws-samples/amazon-ec2-image-builder-samples" rel="noopener noreferrer" target="_blank">amazon-ec2-image-builder-samples</a> GitHub repository, which provides sample documents for executing Ansible playbooks and Chef recipes.</p> 
<p>We continue to use this repository for publishing sample content. We hope Image Builder is providing value and making it easier for you to build virtual machine (VM) images. As always, keep sending us your feedback!</p>
