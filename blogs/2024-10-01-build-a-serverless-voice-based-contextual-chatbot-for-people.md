---
title: "Build a serverless voice-based contextual chatbot for people with disabilities using Amazon Bedrock"
url: "https://aws.amazon.com/blogs/machine-learning/build-a-serverless-voice-based-contextual-chatbot-for-people-with-disabilities-using-amazon-bedrock/"
date: "Tue, 01 Oct 2024 17:45:46 +0000"
author: "Michael Shapira"
feed_url: "https://aws.amazon.com/blogs/machine-learning/category/artificial-intelligence/amazon-polly/feed/"
---
<p>At Amazon and AWS, we are always finding innovative ways to build inclusive technology. With voice assistants like <a href="https://alexa.amazon.com/" rel="noopener" target="_blank">Amazon Alexa</a>, we are enabling more people to ask questions and get answers on the spot without having to type. Whether you’re a person with a motor disability, juggling multiple tasks, or simply away from your computer, getting search results without typing is a valuable feature. With modern voice assistants, you can now ask your questions conversationally and get verbal answers instantly.</p> 
<p>In this post, we discuss voice-guided applications. Specifically, we focus on chatbots. Chatbots are no longer a niche technology. They are now ubiquitous on customer service websites, providing around-the-clock automated assistance. Although AI chatbots have been around for years, recent advances of large language models (LLMs) like <a href="https://aws.amazon.com/generative-ai/" rel="noopener" target="_blank">generative AI</a> have enabled more natural conversations. Chatbots are proving useful across industries, handling both general and industry-specific questions. Voice-based assistants like Alexa demonstrate how we are entering an era of conversational interfaces. Typing questions already feels cumbersome to many who prefer the simplicity and ease of speaking with their devices.</p> 
<p>We explore how to build a fully serverless, voice-based contextual chatbot tailored for individuals who need it. We also provide a sample chatbot application. The application is available in the accompanying <a href="https://github.com/aws-samples/serverless-conversational-chatbot" rel="noopener" target="_blank">GitHub repository</a>. We create an intelligent conversational assistant that can understand and respond to voice inputs in a contextually relevant manner. The AI assistant is powered by <a href="https://aws.amazon.com/bedrock" rel="noopener" target="_blank">Amazon Bedrock</a>. This chatbot is designed to assist users with various tasks, provide information, and offer personalized support based on their unique requirements. For our LLM, we use Anthropic Claude on Amazon Bedrock.</p> 
<p>We demonstrate the process of integrating Anthropic Claude’s advanced natural language processing capabilities with the serverless architecture of Amazon Bedrock, enabling the deployment of a highly scalable and cost-effective solution. Additionally, we discuss techniques for enhancing the chatbot’s accessibility and usability for people with motor disabilities. The aim of this post is to provide a comprehensive understanding of how to build a voice-based, contextual chatbot that uses the latest advancements in AI and serverless computing.</p> 
<p>We hope that this solution can help people with certain mobility disabilities. A limited level of interaction is still required, and specific identification of start and stop talking operations is required. In our sample application, we address this by having a dedicated <strong>Talk</strong> button that performs the transcription process while being pressed.</p> 
<p>For people with significant motor disabilities, the same operation can be implemented with a dedicated physical button that can be pressed by a single finger or another body part. Alternatively, a special keyword can be said to indicate the beginning of the command. This approach is used when you communicate with Alexa. The user always starts the conversation with “Alexa.”</p> 
<h2>Solution overview</h2> 
<p>The following diagram illustrates the architecture of the solution.</p> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/23/image001-7.png" rel="noopener" target="_blank"><img alt="Architecture of serverless components of the solution" class="alignnone size-full wp-image-87492" height="589" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/23/image001-7.png" width="481" /></a></p> 
<p>To deploy this architecture, we need managed compute that can host the web application, authentication mechanisms, and relevant permissions. We discuss this later in the post.</p> 
<p>All the services that we use are serverless and fully managed by AWS. You don’t need to provision the compute resources. You only consume the services through their API. All the calls to the services are made directly from the client application.</p> 
<p>The application is a simple <a href="https://react.dev/" rel="noopener" target="_blank">React</a> application that we create using the <a href="https://vitejs.dev/" rel="noopener" target="_blank">Vite</a> build tool. We use the <a href="https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/" rel="noopener" target="_blank">AWS SDK for JavaScript</a> to call the services. The solution uses the following major services:</p> 
<ul> 
 <li><a href="https://aws.amazon.com/polly/" rel="noopener" target="_blank">Amazon Polly</a> is a service that turns text into lifelike speech.</li> 
 <li><a href="https://aws.amazon.com/pm/transcribe/" rel="noopener" target="_blank">Amazon Transcribe</a> is an AWS AI service that makes it straightforward to convert speech to text.</li> 
 <li><a href="https://aws.amazon.com/bedrock/" rel="noopener" target="_blank">Amazon Bedrock</a> is a fully managed service that offers a choice of high-performing foundation models (FMs) along with a broad set of capabilities that you need to build generative AI applications.</li> 
 <li><a href="https://aws.amazon.com/pm/cognito/" rel="noopener" target="_blank">Amazon Cognito</a> is an identity service for web and mobile apps. It’s a user directory, an authentication server, and an authorization service for OAuth 2.0 access tokens and AWS credentials.</li> 
</ul> 
<p>To consume AWS services, the user needs to obtain temporary credentials from <a href="https://aws.amazon.com/iam/" rel="noopener" target="_blank">AWS Identity and Access Management</a> (IAM). This is possible due to the <a href="https://docs.aws.amazon.com/cognito/latest/developerguide/identity-pools.html" rel="noopener" target="_blank">Amazon Cognito identity pool</a>, which acts as a mediator between your application user and IAM services. The identity pool holds the information about the IAM roles with all permissions necessary to run the solution.</p> 
<p>Amazon Polly and Amazon Transcribe <a href="https://docs.aws.amazon.com/cognito/latest/developerguide/iam-roles.html" rel="noopener" target="_blank">don’t require</a> additional setup from the client aside from what we have described. However, Amazon Bedrock requires named user authentication. This means that having an Amazon Cognito identity pool is not enough—you also need to use the Amazon Cognito user pool, which allows you to define users and bind them to the Amazon Cognito identity pool. To understand better how Amazon Cognito allows external applications to invoke AWS services, refer to&nbsp;refer to&nbsp;<a class="c-link" href="https://aws.amazon.com/blogs/compute/secure-api-access-with-amazon-cognito-federated-identities-amazon-cognito-user-pools-and-amazon-api-gateway/" rel="noopener noreferrer" target="_blank">Secure API Access with Amazon Cognito Federated Identities, Amazon Cognito User Pools, and Amazon API Gateway</a>.</p> 
<p>The heavy lifting of provisioning the Amazon Cognito user pool and identity pool, including generating the sign-in UI for the React application, is done by <a href="https://aws.amazon.com/amplify" rel="noopener" target="_blank">AWS Amplify</a>. Amplify consists of a set of tools (open source framework, visual development environment, console) and services (web application and static website hosting) to accelerate the development of mobile and web applications on AWS. We cover the steps of setting Amplify in the next sections.</p> 
<h2>Prerequisites</h2> 
<p>Before you begin, complete the following prerequisites:</p> 
<ol> 
 <li>Make sure you have the following installed: 
  <ul> 
   <li><a href="https://nodejs.org/" rel="noopener" target="_blank">Node.js</a></li> 
   <li><a href="https://www.npmjs.com/" rel="noopener" target="_blank">npm</a></li> 
   <li><a href="https://git-scm.com/" rel="noopener" target="_blank">git</a></li> 
  </ul> </li> 
 <li>Create an IAM role to use in the Amazon Cognito identity pool. Use the least privilege principal to provide only the minimum set of permissions needed to run the application. 
  <ul> 
   <li>To invoke Amazon Bedrock, use the following code: 
    <div class="hide-language"> 
     <pre><code class="lang-json">{
					  "Version": "2012-10-17",
					  "Statement": [
						{
						  "Sid": "VisualEditor1",
						  "Effect": "Allow",
						  "Action": "bedrock:InvokeModel",
						  "Resource": "*"
						}
					  ]
					}</code></pre> 
    </div> </li> 
   <li>To invoke Amazon Polly, use the following code: 
    <div class="hide-language"> 
     <pre><code class="lang-json">{
					  "Version": "2012-10-17",
					  "Statement": [
						{
						  "Sid": "VisualEditor2",
						  "Effect": "Allow",
						  "Action": "polly:SynthesizeSpeech",
						  "Resource": "*"
						}
					  ]
					}</code></pre> 
    </div> </li> 
   <li>To invoke Amazon Transcribe, use the following code: 
    <div class="hide-language"> 
     <pre><code class="lang-json">{
				  "Version": "2012-10-17",
				  "Statement": [
					{
					  "Sid": "VisualEditor3",
					  "Effect": "Allow",
					  "Action": "transcribe:StartStreamTranscriptionWebSocket",
					  "Resource": "*"
					}
				  ]
				}</code></pre> 
    </div> </li> 
  </ul> </li> 
</ol> 
<p>The full policy JSON should look as follows:</p> 
<div class="hide-language"> 
 <pre><code class="lang-json">{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "VisualEditor1",
      "Effect": "Allow",
      "Action": "bedrock:InvokeModel",
      "Resource": "*"
    },
    {
      "Sid": "VisualEditor2",
      "Effect": "Allow",
      "Action": "polly:SynthesizeSpeech",
      "Resource": "*"
    },
    {
      "Sid": "VisualEditor3",
      "Effect": "Allow",
      "Action": "transcribe:StartStreamTranscriptionWebSocket",
      "Resource": "*"
    }
  ]
}</code></pre> 
</div> 
<ol start="3"> 
 <li>Run the following command to clone the <a href="https://github.com/aws-samples/serverless-conversational-chatbot" rel="noopener" target="_blank">GitHub repository</a>: 
  <div class="hide-language"> 
   <pre><code class="lang-bash">git clone https://github.com/aws-samples/serverless-conversational-chatbot.git</code></pre> 
  </div> </li> 
 <li>To use Amplify, refer to <a href="https://docs.amplify.aws/gen1/react/start/getting-started/installation/" rel="noopener" target="_blank">Set up Amplify CLI</a> to complete the initial setup.</li> 
 <li>To be consistent with the values that you use later in the instructions, call your AWS profile <code>amplify</code> when you see the following prompt.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/23/image002-5.png" rel="noopener" target="_blank"><img alt="Creation of the AWS profile &quot;amplify&quot;" class="alignnone wp-image-87493 size-full" height="240" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/23/image002-5.png" style="margin: 10px 0px 10px 0px;" width="1314" /></a></li> 
 <li>Create the role <code>amplifyconsole-backend-role</code> with the <code>AdministratorAccess-Amplify</code> managed policy, which allows Amplify to create the necessary resources.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/23/image003-2.jpg" rel="noopener" target="_blank"><img alt="IAM Role with &quot;AdministratorAccess-Amplify&quot; policy" class="alignnone size-full wp-image-87494" height="1170" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/23/image003-2.jpg" style="margin: 10px 0px 10px 0px;" width="1358" /></a></li> 
 <li>For this post, we use the Anthropic Claude 3 Haiku LLM. To enable the LLM in Amazon Bedrock, refer to <a href="https://docs.aws.amazon.com/bedrock/latest/userguide/model-access.html" rel="noopener" target="_blank">Access Amazon Bedrock foundation models</a>.</li> 
</ol> 
<h2>Deploy the solution</h2> 
<p>There are two options to deploy the solution:</p> 
<ul> 
 <li>Use Amplify to deploy the application automatically</li> 
 <li>Deploy the application manually</li> 
</ul> 
<p>We provide the steps for both options in this section.</p> 
<h3>Deploy the application automatically using Amplify</h3> 
<p>Amplify can deploy the application automatically if it’s stored in GitHub, Bitbucket, GitLab, or <a href="https://aws.amazon.com/codecommit/" rel="noopener" target="_blank">AWS CodeCommit</a>. Upload the application that you downloaded earlier to your preferred repository (from the aforementioned options). For instructions, see <a href="https://docs.aws.amazon.com/amplify/latest/userguide/getting-started.html" rel="noopener" target="_blank">Getting started with deploying an app to Amplify Hosting</a>.</p> 
<p>You can now continue to the next section of this post to set up IAM permissions.</p> 
<h3>Deploy the application manually</h3> 
<p>If you don’t have access to one of the storage options that we mentioned, you can deploy the application manually. This can also be useful if you want to modify the application to better fit your use case.</p> 
<p>We tested the deployment on <a href="https://aws.amazon.com/pm/cloud9/" rel="noopener" target="_blank">AWS Cloud9</a>, a cloud integrated development environment (IDE) for writing, running, and debugging code, with Ubuntu Server 22.04 and Amazon Linux 2023.</p> 
<p>We use the Visual Studio Code IDE and run all the following commands directly in the terminal window inside the IDE, but you can also run the commands in the terminal of your choice.</p> 
<ol> 
 <li>From the directory where you checked out the application on GitHub, run the following command: 
  <div class="hide-language"> 
   <pre><code class="lang-bash">cd serverless-conversational-chatbot</code></pre> 
  </div> </li> 
 <li>Run the following commands: 
  <div class="hide-language"> 
   <pre><code class="lang-bash">npm i

amplify init</code></pre> 
  </div> </li> 
 <li>Follow the prompts as shown in the following screenshot. 
  <ul> 
   <li>For authentication, choose the AWS profile <code>amplify</code> that you created as part of the prerequisite steps.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/23/image004-1.jpg" rel="noopener" target="_blank"><img alt="Initial AWS Amplify setup in React application: 1. Do you want to use an existing environment? No 2. Enter a name for the environment: sampleenv 3. Select the authentication method you want to use: AWS Profile 4. Please choose the profile you want to use: amplify" class="alignnone size-full wp-image-87495" height="802" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/23/image004-1.jpg" style="margin: 10px 0px 10px 0px;" width="1288" /></a></li> 
   <li>Two new files will appear in the project under the <code>src</code> folder: 
    <ul> 
     <li><code>amplifyconfiguration.json</code></li> 
     <li><code>aws-exports.js</code></li> 
    </ul> <p><a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/23/image005-1.jpg" rel="noopener" target="_blank"><img alt="New objects created by AWS Amplify: 1. aws-exports.js 2. amplifyconfiguration.json" class="alignnone size-full wp-image-87496" height="780" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/23/image005-1.jpg" style="margin: 10px 0px 10px 0px;" width="1036" /></a></p></li> 
  </ul> </li> 
</ol> 
<ol start="4"> 
 <li>Next run the following command: 
  <div class="hide-language"> 
   <pre><code class="lang-bash">amplify configure project</code></pre> 
  </div> </li> 
</ol> 
<p style="padding-left: 40px;">Then select “Project Information”</p> 
<p style="padding-left: 40px;"><a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/23/image006-1.jpg" rel="noopener" target="_blank"><img alt="Project Configuration of AWS Amplify in React Applications" class="alignnone size-full wp-image-87497" height="180" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/23/image006-1.jpg" style="margin: 10px 0px 10px 0px;" width="1576" /></a></p> 
<ol start="5"> 
 <li>&nbsp;Enter the following information: 
  <div class="hide-language"> 
   <pre><code class="lang-code">Which setting do you want to configure? Project information

Enter a name for the project: servrlsconvchat

Choose your default editor: Visual Studio Code

Choose the type of app that you're building: javascript

What javascript framework are you using: react

Source Directory Path: src

Distribution Directory Path: dist

Build Command: npm run-script build

Start Command: npm run-script start</code></pre> 
  </div> </li> 
</ol> 
<p>You can <a href="https://docs.amplify.aws/gen1/javascript/build-a-backend/auth/import-existing-resources/" rel="noopener" target="_blank">use an existing Amazon Cognito identity pool and user pool</a> or <a href="https://docs.amplify.aws/javascript/build-a-backend/auth/set-up-auth/" rel="noopener" target="_blank">create new objects</a>.</p> 
<ol start="6"> 
 <li>For our application, run the following command: 
  <div class="hide-language"> 
   <pre><code class="lang-bash">amplify add auth</code></pre> 
  </div> </li> 
</ol> 
<p style="padding-left: 40px;">If you get the following message, you can ignore it:</p> 
<div class="hide-language"> 
 <pre style="padding-left: 40px;"><code class="lang-bash">Auth has already been added to this project. To update run amplify update auth</code></pre> 
</div> 
<ol start="7"> 
 <li>Choose <strong>Default configuration</strong>.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/23/image007-2.png" rel="noopener" target="_blank"><img alt="Selecting &quot;default configuration&quot; when adding authentication objects" class="alignnone size-full wp-image-87498" height="190" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/23/image007-2.png" style="margin: 10px 0px 10px 0px;" width="1308" /></a></li> 
 <li>Accept all options proposed by the prompt.</li> 
 <li>Run the following command: 
  <div class="hide-language"> 
   <pre><code class="lang-bash">amplify add hosting</code></pre> 
  </div> </li> 
 <li>Choose your hosting option.</li> 
</ol> 
<p>You have two options to host the application. The application can be hosted to the Amplify console or to <a href="https://aws.amazon.com/pm/serv-s3/" rel="noopener" target="_blank">Amazon Simple Storage Service</a> (Amazon S3) and then exposed through <a href="https://aws.amazon.com/cloudfront/" rel="noopener" target="_blank">Amazon CloudFront</a>.</p> 
<p>Hosting with the Amplify console differs from CloudFront and Amazon S3. The Amplify console is a managed service providing continuous integration and delivery (CI/CD) and SSL certificates, prioritizing swift deployment of serverless web applications and backend APIs. In contrast, CloudFront and Amazon S3 offer greater flexibility and customization options, particularly for hosting static websites and assets with features like caching and distribution. CloudFront and Amazon S3 are preferable for intricate, high-traffic web applications with specific performance and security needs.</p> 
<p>For this post, we use the Amplify console. To learn more about the deployment with Amazon S3 and Amazon CloudFront, refer to <a href="https://docs.amplify.aws/gen1/react/tools/cli/hosting/" rel="noopener" target="_blank">documentation</a>.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/23/image008-4.jpg" rel="noopener" target="_blank"><img alt="Selecting the deployment option for the React application on the Amplify Console. Selected option: Hosting with Amplify Console" class="alignnone size-full wp-image-87499" height="244" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/23/image008-4.jpg" width="1658" /></a></p> 
<p>Now you’re ready to publish the application. There is an option to publish the application to GitHub to support CI/CD pipelines. Amplify has built-in integration with GitHub and can redeploy the application automatically when you push the changes. For simplicity, we use manual deployment.</p> 
<ol start="12"> 
 <li>Choose <strong>Manual deployment</strong>.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/23/image009.png" rel="noopener" target="_blank"><img alt="Selecting &quot;Manual Deployment&quot; when publishing the project" class="alignnone size-full wp-image-87500" height="204" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/23/image009.png" style="margin: 10px 0px 10px 0px;" width="984" /></a></li> 
 <li>Run the following command: 
  <div class="hide-language"> 
   <pre><code class="lang-bash">amplify publish</code></pre> 
  </div> </li> 
</ol> 
<p>After the application is published, you will see the following output. Note down this URL to use in a later step.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/23/image010-4.jpg" rel="noopener" target="_blank"><img alt="Result of the Deployment of the React Application on the Amplify Console. The URL that the user should use to enter the Amplify application" class="alignnone size-full wp-image-87501" height="760" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/23/image010-4.jpg" style="margin: 10px 0px 10px 0px;" width="1486" /></a></p> 
<ol start="13"> 
 <li>Log in to the Amplify console, navigate to the <code>servrlsconvchat</code> application, and choose <strong>General</strong> under <strong>App settings</strong> in the navigation pane.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/23/image011-1.jpg" rel="noopener" target="_blank"><img alt="Service Role attachment to the deployed application. First step. Select the deployed application. Seelct “General” option" class="alignnone size-full wp-image-87502" height="834" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/23/image011-1.jpg" style="margin: 10px 0px 10px 0px;" width="1482" /></a></li> 
 <li>Edit the app settings and enter <code>amplifyconsole-backend-role</code> for <strong>Service role</strong> (you created this role in the prerequisites section).<br /> <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/23/image012-1.jpg" rel="noopener" target="_blank"><img alt="Service Role attachment to the deployed application. Second step. Setting “amplifyconsole-backend-role” in the “Service role” field" class="alignnone size-full wp-image-87503" height="1096" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/23/image012-1.jpg" style="margin: 10px 0px 10px 0px;" width="1566" /></a></li> 
</ol> 
<p>Now you can proceed to the next section to set up IAM permissions.</p> 
<h2>Configure IAM permissions</h2> 
<p>As part of the publishing method you completed, you provisioned a new identity pool. You can view this on the Amazon Cognito console, along with a new user pool. The names will be different from those presented in this post.</p> 
<p>As we explained earlier, you need to attach policies to this role to allow interaction with Amazon Bedrock, Amazon Polly, and Amazon Transcribe. To set up IAM permissions, complete the following steps:</p> 
<ol> 
 <li>On the Amazon Cognito console, choose <strong>Identity pools</strong> in the navigation pane.</li> 
 <li>Navigate to your identity pool.</li> 
 <li>On the <strong>User access tab</strong>, choose the link for the authenticated role.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/23/image013.png" rel="noopener" target="_blank"><img alt="Identifying the IAM Authentication Role in the Cognitive Identity Pool. Select “Identity pools” option in the console. Select “User access” tab. Click on the link under “Authentication role”" class="alignnone size-full wp-image-87504" height="1172" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/23/image013.png" style="margin: 10px 0px 10px 0px;" width="2330" /></a></li> 
 <li>Attach the policies that you defined in the prerequisites section.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/23/image014-1.jpg" rel="noopener" target="_blank"><img alt="IAM Policies Attached to Cognito Identity Pool Authenticated Roles. Textual data presaented in “Prerequisites” section, item 2." class="alignnone size-full wp-image-87505" height="1482" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/23/image014-1.jpg" style="margin: 10px 0px 10px 0px;" width="1222" /></a></li> 
</ol> 
<p>Amazon Bedrock can only be used with a named user, so we create a sample user in the Amazon Cognito user pool that was provisioned as part of the application publishing process.</p> 
<ol start="5"> 
 <li>On the user pool details page, on the <strong>Users </strong>tab, choose <strong>Create user</strong>.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/23/image015.png" rel="noopener" target="_blank"><img alt="User Creation in the Cognito User Pool. Select relevant user pool in “User pools” section. Select “Users” tab. Click on “Create user” button" class="alignnone size-full wp-image-87506" height="1742" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/23/image015.png" style="margin: 10px 0px 10px 0px;" width="2318" /></a></li> 
 <li>Provide your user information.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/23/image016-1.jpg" rel="noopener" target="_blank"><img alt="Sample user definition in the Cognito User Pool. Enter email address and temporary password." class="alignnone size-full wp-image-87507" height="816" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/23/image016-1.jpg" style="margin: 10px 0px 10px 0px;" width="1289" /></a></li> 
</ol> 
<p>You’re now ready to run the application.</p> 
<h2>Use the sample serverless application</h2> 
<p>To access the application, navigate to the URL you saved from the output at the end of the application publishing process. Sign in to the application with the user you created in the previous step. You might be asked to change the password the first time you sign in.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/23/image017.jpg" rel="noopener" target="_blank"><img alt="Application Login Page. Enter user name and password" class="alignnone size-full wp-image-87508" height="736" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/23/image017.jpg" style="margin: 10px 0px 10px 0px;" width="1026" /></a></p> 
<p>Use the <strong>Talk</strong> button and hold it while you’re asking the question. (We use this approach for the simplicity of demonstrating the abilities of the tool. For people with motor disabilities, we propose using a dedicated button that can be operated with different body parts, or a special keyword to initiate the conversation.)</p> 
<p>When you release the button, the application sends your voice to Amazon Transcribe and returns the transcription text. This text is used as an input for an Amazon Bedrock LLM. For this example, we use Anthropic Claude 3 Haiku, but you can modify the code and use another model.</p> 
<p>The response from Amazon Bedrock is displayed as text and is also spoken by Amazon Polly.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/23/image018.jpg" rel="noopener" target="_blank"><img alt="Instructions on how to invoke the &quot;Talk&quot; operation, by using “Talk” operation" class="alignnone size-full wp-image-87509" height="586" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/23/image018.jpg" style="margin: 10px 0px 10px 0px;" width="1286" /></a></p> 
<p>The conversation history is also stored. This means that you can ask follow-up questions, and the context of the conversation is preserved. For example, we asked, “What is the most famous tower there?” without specifying the location, and our chatbot was able to understand that the context of the question is Paris based on our previous question.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/23/image019.jpg" rel="noopener" target="_blank"><img alt="Demonstration of context preservation during conversation. Continues question-answer conversation with chatbot." class="alignnone size-full wp-image-87510" height="677" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/23/image019.jpg" style="margin: 10px 0px 10px 0px;" width="1288" /></a></p> 
<p>We store the conversation history inside a JavaScript variable, which means that if you refresh the page, the context will be lost. We discuss how to preserve the conversation context in a persistent way later in this post.</p> 
<p>To identify that the transcription process is happening, choose and hold the <strong>Talk</strong> button. The color of the button changes and a microphone icon appears.</p> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/23/image020.png" rel="noopener" target="_blank"><img alt="&quot;Talk&quot; operation indicator. “Talk” button changes color to orche" class="alignnone wp-image-87511 size-medium" height="70" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/23/image020-300x70.png" style="margin: 10px 0px 10px 0px;" width="300" /></a></p> 
<h2>Clean up</h2> 
<p>To clean up your resources, run the following command from the same directory where you ran the Amplify commands:</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">amplify delete</code></pre> 
</div> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/23/image021.jpg" rel="noopener" target="_blank"><img alt="Result of the &quot;Cleanup&quot; operation after running “amplify delete” command" class="alignnone size-full wp-image-87512" height="264" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/23/image021.jpg" style="margin: 10px 0px 10px 0px;" width="1289" /></a></p> 
<p>This command removes the Amplify settings from the React application, Amplify resources, and all Amazon Cognito objects, including the IAM role and Amazon Cognito user pool’s user.</p> 
<h2>Conclusion</h2> 
<p>In this post, we presented how to create a fully serverless voice-based contextual chatbot using Amazon Bedrock with Anthropic Claude.</p> 
<p>This serves a starting point for a serverless and cost-effective solution. For example, you could extend the solution to have persistent conversational memory for your chats, such as <a href="https://aws.amazon.com/dynamodb/" rel="noopener" target="_blank">Amazon DynamoDB</a>. If you want to use a Retrieval Augmented Generation (RAG) approach, you can use <a href="https://aws.amazon.com/bedrock/knowledge-bases/" rel="noopener" target="_blank">Amazon Bedrock</a> Knowledge Bases to securely connect FMs in Amazon Bedrock to your company data.</p> 
<p>Another approach is to customize the model you use in Amazon Bedrock with your own data using fine-tuning or continued pre-training to build applications that are specific to your domain, organization, and use case. With custom models, you can create unique user experiences that reflect your company’s style, voice, and services.</p> 
<p>For additional resources, refer to the following:</p> 
<ul> 
 <li><a href="https://aws.amazon.com/blogs/compute/building-a-serverless-document-chat-with-aws-lambda-and-amazon-bedrock/" rel="noopener" target="_blank">Building a serverless document chat with AWS Lambda and Amazon Bedrock</a></li> 
 <li><a href="https://aws.amazon.com/blogs/aws/knowledge-bases-now-delivers-fully-managed-rag-experience-in-amazon-bedrock/" rel="noopener" target="_blank">Knowledge Bases now delivers fully managed RAG experience in Amazon Bedrock</a></li> 
 <li><a href="https://aws.amazon.com/blogs/aws/customize-models-in-amazon-bedrock-with-your-own-data-using-fine-tuning-and-continued-pre-training/" rel="noopener" target="_blank">Customize models in Amazon Bedrock with your own data using fine-tuning and continued pre-training</a></li> 
</ul> 
<p style="clear: both;"></p> 
<hr /> 
<h3>About the Author</h3> 
<p style="clear: both;"><strong><a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/25/Author1_Michael_small.png" rel="noopener" target="_blank"><img alt="" class="size-full wp-image-87728 alignleft" height="132" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/25/Author1_Michael_small.png" width="100" /></a>Michael Shapira</strong>&nbsp;is a Senior Solution Architect covering general topics in AWS and part of the AWS Machine Learning community. He has 16 years’ experience in Software Development. He finds it fascinating to work with cloud technologies and help others on their cloud journey.</p> 
<p style="clear: both;"><strong><a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/25/eitan-sela-100.png" rel="noopener" target="_blank"><img alt="" class="size-full wp-image-87727 alignleft" height="150" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/09/25/eitan-sela-100.png" width="100" /></a>Eitan Sela </strong>is a Machine Learning Specialist Solutions Architect with Amazon Web Services. He works with AWS customers to provide guidance and technical assistance, helping them build and operate machine learning solutions on AWS. In his spare time, Eitan enjoys jogging and reading the latest machine learning articles.</p>
