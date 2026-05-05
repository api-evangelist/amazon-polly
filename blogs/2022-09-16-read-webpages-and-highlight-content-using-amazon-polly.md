---
title: "Read webpages and highlight content using Amazon Polly"
url: "https://aws.amazon.com/blogs/machine-learning/read-webpages-and-highlight-content-using-amazon-polly/"
date: "Fri, 16 Sep 2022 15:23:27 +0000"
author: "Mike Havey"
feed_url: "https://aws.amazon.com/blogs/machine-learning/category/artificial-intelligence/amazon-polly/feed/"
---
<p>In this post, we demonstrate how to use <a href="https://aws.amazon.com/polly/" rel="noopener noreferrer" target="_blank">Amazon Polly</a>—a leading cloud service that converts text into lifelike speech—to read the content of a webpage and highlight the content as it’s being read. Adding audio playback to a webpage improves the accessibility and visitor experience of the page. Audio-enhanced content is more impactful and memorable, draws more traffic to the page, and taps into the spending power of visitors. It also improves the brand of the company or organization that publishes the page. Text-to-speech technology makes these business benefits attainable. We accelerate that journey by demonstrating how to achieve this goal using Amazon Polly.</p> 
<p>This capability improves accessibility for visitors with disabilities, and could be adopted as part of your organization’s accessibility strategy. Just as importantly, it enhances the page experience for visitors without disabilities. Both groups have significant spending power and spend more freely from pages that use audio enhancement to grab their attention.</p> 
<h2>Overview of solution</h2> 
<p><code>PollyReadsThePage</code> (PRTP)—as we refer to the solution—allows a webpage publisher to drop an audio control onto their webpage. When the visitor chooses <strong>Play</strong> on the control, the control reads the page and highlights the content. PRTP uses the general capability of Amazon Polly to synthesize speech from text. It invokes Amazon Polly to generate two artifacts for each page:</p> 
<ul> 
 <li>The audio content in a format playable by the browser: MP3</li> 
 <li>A speech marks file that indicates for each sentence of text: 
  <ul> 
   <li>The time during playback that the sentence is read</li> 
   <li>The location on the page the sentence appears</li> 
  </ul> </li> 
</ul> 
<p>When the visitor chooses <strong>Play</strong>, the browser plays the MP3 file. As the audio is read, the browser checks the time, finds in the marks file which sentence to read at that time, locates it on the page, and highlights it.</p> 
<p>PRTP allows the visitor to read in different voices and languages. Each voice requires its own pair of files. PRTP uses neural voices. For a list of supported neural voices and languages, see <a href="https://docs.aws.amazon.com/polly/latest/dg/ntts-voices-main.html" rel="noopener noreferrer" target="_blank">Neural Voices</a>. For a full list of standard and neural voices in Amazon Polly, see <a href="https://docs.aws.amazon.com/polly/latest/dg/voicelist.html" rel="noopener noreferrer" target="_blank">Voices in Amazon Polly</a>.</p> 
<p>We consider two types of webpages: static and dynamic pages. In a <em>static</em> page, the content is contained within the page and changes only when a new version of the page is published. The company might update the page daily or weekly as part of its web build process. For this type of page, it’s possible to pre-generate the audio files at build time and place them on the web server for playback. As the following figure shows, the script <code>PRTP Pre-Gen</code> invokes Amazon Polly to generate the audio. It takes as input the HTML page itself and, optionally, a configuration file that specifies which text from the page to extract (<code>Text Extract Config</code>). If the extract config is omitted, the pre-gen script makes a sensible choice of text to extract from the body of the page. Amazon Polly outputs the files in an <a href="https://aws.amazon.com/s3/" rel="noopener noreferrer" target="_blank">Amazon Simple Storage Service</a> (Amazon S3) bucket; the script copies them to your web server. When the visitor plays the audio, the browser downloads the MP3 directly from the web server. For highlights, a drop-in library, <code>PRTP.js</code>, uses the marks file to highlight the text being read.<a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2022/08/30/image001-8.png"><img alt="Solution Architecture Diagram" class="alignnone size-full wp-image-41844" height="1152" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2022/08/30/image001-8.png" style="margin: 10px 0px 10px 0px;" width="1676" /></a></p> 
<p>The content of a <em>dynamic</em> page changes in response to the visitor interaction, so audio can’t be pre-generated but must be synthesized dynamically. As the following figure shows, when the visitor plays the audio, the page uses <code>PRTP.js</code> to generate the audio in Amazon Polly, and it highlights the synthesized audio using the same approach as with static pages. To access AWS services from the browser, the visitor requires an AWS identity. We show how to use an <a href="https://aws.amazon.com/cognito/" rel="noopener noreferrer" target="_blank">Amazon Cognito</a> identity pool to allow the visitor just enough access to Amazon Polly and the S3 bucket to render the audio.</p> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2022/08/30/image003-4.png"><img alt="Dynamic Content" class="alignnone size-full wp-image-41845" height="916" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2022/08/30/image003-4.png" style="margin: 10px 0px 10px 0px;" width="1406" /></a></p> 
<p>Generating both Mp3 audio and speech marks requires the Polly service to synthesize the same input twice. Refer to the <a href="https://aws.amazon.com/polly/pricing/">Amazon Polly Pricing Page</a> to understand cost implications. Pre-generation saves costs because synthesis is performed at build time rather than on-demand for each visitor interaction.</p> 
<p>The code accompanying this post is available as an open-source repository on <a href="https://github.com/aws-samples/amazon-polly-reads-the-page" rel="noopener noreferrer" target="_blank">GitHub</a>.</p> 
<p>To explore the solution, we follow these steps:</p> 
<ol> 
 <li>Set up the resources, including the pre-gen build server, S3 bucket, web server, and Amazon Cognito identity.</li> 
 <li>Run the static pre-gen build and test static pages.</li> 
 <li>Test dynamic pages.</li> 
</ol> 
<h2>Prerequisites</h2> 
<p>To run this example, you need an <a href="https://portal.aws.amazon.com/billing/signup" rel="noopener noreferrer" target="_blank">AWS account</a> with permission to use Amazon Polly, Amazon S3, Amazon Cognito, and (for demo purposes) <a href="https://aws.amazon.com/cloud9/" rel="noopener noreferrer" target="_blank">AWS Cloud9</a>.</p> 
<h2>Provision resources</h2> 
<p>We share an <a href="http://aws.amazon.com/cloudformation" rel="noopener noreferrer" target="_blank">AWS CloudFormation</a> template to create in your account a self-contained demo environment to help you follow along with the post. If you prefer to set up PRTP in your own environment, refer to instructions in <a href="https://github.com/aws-samples/amazon-polly-reads-the-page/blob/main/README.md" rel="noopener noreferrer" target="_blank">README.md</a>.</p> 
<p>To provision the demo environment using CloudFormation, first download a copy of the <a href="https://github.com/aws-samples/amazon-polly-reads-the-page/blob/main/cfn/prtp.yaml" rel="noopener noreferrer" target="_blank">CloudFormation template</a>. Then complete the following steps:</p> 
<ol> 
 <li>On the AWS CloudFormation console, choose <strong>Create stack.</strong></li> 
 <li>Choose <strong>With new resources (standard)</strong>.</li> 
 <li>Select <strong>Upload a template file. </strong></li> 
 <li>Choose <strong>Choose file</strong> to upload the local copy of the template that you downloaded. The name of the file is <code>prtp.yml</code>.</li> 
 <li>Choose <strong>Next</strong>.</li> 
 <li>Enter a stack name of your choosing. Later you enter this again as a replacement for <em>&lt;StackName&gt;</em>.</li> 
 <li>You may keep default values in the <strong>Parameters </strong>section.</li> 
 <li>Choose <strong>Next</strong>.</li> 
 <li>Continue through the remaining sections.</li> 
 <li>Read and select the check boxes in the <strong>Capabilities</strong> section.</li> 
 <li>Choose <strong>Create stack</strong>.</li> 
 <li>When the stack is complete, find the value for <code>BucketName</code> in the stack outputs.</li> 
</ol> 
<p>We encourage you to review the stack with your security team prior to using it a production environment.</p> 
<h2>Set up the web server and pre-gen server in an AWS Cloud9 IDE</h2> 
<p>Next, on the AWS Cloud9 console, locate the environment <code>PRTPDemoCloud9</code> created by the CloudFormation stack. Choose <strong>Open IDE</strong> to open the AWS Cloud9 environment. Open a terminal window and run the following commands, which clones the PRTP code, sets up pre-gen dependencies, and starts a web server to test with:</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">#Obtain PRTP code
cd /home/ec2-user/environment
git clone https://github.com/aws-samples/amazon-polly-reads-the-page.git

# Navigate to that code
cd amazon-polly-reads-the-page/setup

# Install Saxon and html5 Python lib. For pre-gen.
sh ./setup.sh &lt;StackName&gt;

# Run Python simple HTTP server
cd ..
./runwebserver.sh &lt;IngressCIDR&gt; </code></pre> 
</div> 
<p>For <em>&lt;StackName&gt;</em>, use the name you gave the CloudFormation stack. For <em>&lt;IngressCIDR&gt;</em>, specify a range of IP addresses allowed to access the web server. To restrict access to the browser on your local machine, find your IP address using <a href="https://whatismyipaddress.com/" rel="noopener noreferrer" target="_blank">https://whatismyipaddress.com/</a> and append <code>/32</code> to specify the range. For example, if your IP is <code>10.2.3.4, use 10.2.3.4/32</code>. The server listens on port 8080. The public IP address on which the server listens is given in the output. For example:</p> 
<div class="hide-language"> 
 <pre><code class="lang-code">Public IP is

3.92.33.223</code></pre> 
</div> 
<h2>Test static pages</h2> 
<p>In your browser, navigate to <code>PRTPStaticDefault.html</code>. (If you’re using the demo, the URL is <code>http://&lt;cloud9host&gt;:8080/web/PRTPStaticDefault.html</code>, where <em>&lt;cloud9host&gt;</em> is the public IP address that you discovered in setting up the IDE.) Choose <strong>Play</strong> on the audio control at the top. Listen to the audio and watch the highlights. Explore the control by changing speeds, changing voices, pausing, fast-forwarding, and rewinding. The following screenshot shows the page; the text “Skips hidden paragraph” is highlighted because it is currently being read.</p> 
<p><img alt="" class="alignnone wp-image-42705 size-full" height="840" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2022/09/15/Screen-Shot-2022-09-15-at-3.56.56-PM-1.png" width="722" /></p> 
<p>Try the same for <code>PRTPStaticConfig.html</code> and <code>PRTPStaticCustom.html</code>. The results are similar. For example, all three read the alt text for the photo of the cat (“Random picture of a cat”). All three read NE, NW, SE, and SW as full words (“northeast,” “northwest,” “southeast,” “southwest”), taking advantage of Amazon Polly lexicons.</p> 
<p>Notice the main differences in audio:</p> 
<ul> 
 <li><code>PRTPStaticDefault.html</code> reads all the text in the body of the page, including the wrapup portion at the bottom with “Your thoughts in one word,” “Submit Query,” “Last updated April 1, 2020,” and “Questions for the dev team.” <code>PRTPStaticConfig.html</code> and <code>PRTPStaticCustom.html</code> don’t read these because they explicitly exclude the wrapup from speech synthesis.</li> 
 <li><code>PRTPStaticCustom.html</code> reads the <strong>QB Best Sellers</strong> table differently from the others. It reads the first three rows only, and reads the row number for each row. It repeats the columns for each row. <code>PRTPStaticCustom.html</code> uses a custom transformation to tailor the readout of the table. The other pages use default table rendering.</li> 
 <li><code>PRTPStaticCustom.html</code> reads “Tom Brady” at a louder volume than the rest of the text. It uses the speech synthesis markup language (SSML) <code>prosody</code> tag to tailor the reading of Tom Brady. The other pages don’t tailor in this way.</li> 
 <li><code>PRTPStaticCustom.html</code>, thanks to a custom transformation, reads the main tiles in NW, SW, NE, SE order; that is, it reads “Today’s Articles,” “Quote of the Day,” “Photo of the Day,” “Jokes of the Day.” The other pages read in the order the tiles appear in the natural NW, NE, SW, SE order they appear in the HTML: “Today’s Articles,” “Photo of the Day,” “Quote of the Day,” “Jokes of the Day.”</li> 
</ul> 
<p>Let’s dig deeper into how the audio is generated, and how the page highlights the text.</p> 
<h3>Static pre-generator</h3> 
<p>Our GitHub repo includes pre-generated audio files for the <code>PRPTStatic</code> pages, but if you want to generate them yourself, from the bash shell in the AWS Cloud9 IDE, run the following commands:</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash"># navigate to examples
cd /home/ec2-user/environment/amazon-polly-reads-the-page-blog/pregen/examples

# Set env var for my S3 bucket. Example, I called mine prtp-output
S3_BUCKET=prtp-output <strong># Use output BucketName from CloudFormation</strong>

#Add lexicon for pronuniciation of NE NW SE NW
#Script invokes aws polly put-lexicon
./addlexicon.sh.

#Gen each variant
./gen_default.sh
./gen_config.sh
./gen_custom.sh</code></pre> 
</div> 
<p>Now let’s look at how those scripts work.</p> 
<h3>Default case</h3> 
<p>We begin with <code>gen_default.sh</code>:</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">cd ..
python FixHTML.py ../web/PRTPStaticDefault.html \ 
   example/tmp_wff.html
./gen_ssml.sh example/tmp_wff.html generic.xslt example/tmp.ssml
./run_polly.sh example/tmp.ssml en-US Joanna \
   ../web/polly/PRTPStaticDefault compass
./run_polly.sh example/tmp.ssml en-US Matthew \
   ../web/polly/PRTPStaticDefault compass</code></pre> 
</div> 
<p>The script begins by running the Python program <code>FixHTML.py</code> to make the source HTML file <code>PRTPStaticDefault.html</code> well-formed. It writes the well-formed version of the file to <code>example/tmp_wff.html</code>. This step is crucial for two reasons:</p> 
<ul> 
 <li>Most source HTML is not well formed. This step repairs the source HTML to be well formed. For example, many HTML pages don’t close <code>P</code> elements. This step closes them.</li> 
 <li>We keep track of where in the HTML page we find text. We need to track locations using the same document object model (DOM) structure that the browser uses. For example, the browser automatically adds a <code>TBODY</code> to a <code>TABLE</code>. The Python program follows the same well-formed repairs as the browser.</li> 
</ul> 
<p><code>gen_ssml.sh</code> takes the well-formed HTML as input, applies an XML stylesheet transformation (XSLT) transformation to it, and outputs an SSML file. (SSML is the language in Amazon Polly to control how audio is rendered from text.) In the current example, the input is <code>example/tmp_wff.html</code>. The output is <code>example/tmp.ssml</code>. The transform’s job is to decide what text to extract from the HTML and feed to Amazon Polly. <code>generic.xslt</code> is a sensible default XSLT transform for most webpages. In the following example code snippet, it excludes the audio control, the HTML header, as well as HTML elements like <code>script</code> and <code>form</code>. It also excludes elements with the hidden attribute. It includes elements that typically contain text, such as <code>P</code>, <code>H1</code>, and <code>SPAN</code>. For these, it renders both a mark, including the full XPath expression of the element, and the value of the element.</p> 
<div class="hide-language"> 
 <pre><code class="lang-xml">&lt;!-- skip the header --&gt;
&lt;xsl:template match="html/head"&gt;
&lt;/xsl:template&gt;

&lt;!-- skip the audio itself --&gt;
&lt;xsl:template match="html/body/table[@id='prtp-audio']"&gt;
&lt;/xsl:template&gt;

&lt;!-- For the body, work through it by applying its templates. This is the default. --&gt;
&lt;xsl:template match="html/body"&gt;
&lt;speak&gt;
      &lt;xsl:apply-templates /&gt;
&lt;/speak&gt;
&lt;/xsl:template&gt;

&lt;!-- skip these --&gt;
&lt;xsl:template match="audio|option|script|form|input|*[@hidden='']"&gt;
&lt;/xsl:template&gt;

&lt;!-- include these --&gt;
&lt;xsl:template match="p|h1|h2|h3|h4|li|pre|span|a|th/text()|td/text()"&gt;
&lt;xsl:for-each select="."&gt;
&lt;p&gt;
      &lt;mark&gt;
          &lt;xsl:attribute name="name"&gt;
          &lt;xsl:value-of select="prtp:getMark(.)"/&gt;
          &lt;/xsl:attribute&gt;
      &lt;/mark&gt;
      &lt;xsl:value-of select="normalize-space(.)"/&gt;
&lt;/p&gt;
&lt;/xsl:for-each&gt;
&lt;/xsl:template&gt;</code></pre> 
</div> 
<p>The following is a snippet of the SSML that is rendered. This is fed as input to Amazon Polly. Notice, for example, that the text “Skips hidden paragraph” is to be read in the audio, and we associate it with a mark, which tells us that this text occurs in the location on the page given by the XPath expression <code>/html/body[1]/div[2]/ul[1]/li[1]</code>.</p> 
<div class="hide-language"> 
 <pre><code class="lang-html">&lt;speak&gt;
&lt;p&gt;&lt;mark name="/html/body[1]/div[1]/h1[1]"/&gt;PollyReadsThePage Normal Test Page&lt;/p&gt;
&lt;p&gt;&lt;mark name="/html/body[1]/div[2]/p[1]"/&gt;PollyReadsThePage is a test page for audio readout with highlights.&lt;/p&gt;
&lt;p&gt;&lt;mark name="/html/body[1]/div[2]/p[2]"/&gt;Here are some features:&lt;/p&gt;
<strong>&lt;p&gt;&lt;mark name="/html/body[1]/div[2]/ul[1]/li[1]"/&gt;Skips hidden paragraph&lt;/p&gt;</strong>
&lt;p&gt;&lt;mark name="/html/body[1]/div[2]/ul[1]/li[2]"/&gt;Speaks but does not highlight collapsed content&lt;/p&gt;
…
&lt;/speak&gt;</code></pre> 
</div> 
<p>To generate audio in Amazon Polly, we call the script <code>run_polly.sh</code>. It runs the <a href="http://aws.amazon.com/cli" rel="noopener noreferrer" target="_blank">AWS Command Line Interface</a> (AWS CLI) command <code>aws polly start-speech-synthesis-task</code> twice: once to generate MP3 audio, and again to generate the marks file. Because the generation is asynchronous, the script polls until it finds the output in the specified S3 bucket. When it finds the output, it downloads to the build server and copies the files to the <code>web/polly</code> folder. The following is a listing of the web folders:</p> 
<ul> 
 <li>PRTPStaticDefault.html</li> 
 <li>PRTPStaticConfig.html</li> 
 <li>PRTPStaticCustom.html</li> 
 <li>PRTP.js</li> 
 <li>polly/PRTPStaticDefault/Joanna.mp3, Joanna.marks, Matthew.mp3, Matthew.marks</li> 
 <li>polly/PRTPStaticConfig/Joanna.mp3, Joanna.marks, Matthew.mp3, Matthew.marks</li> 
 <li>polly/PRTPStaticCustom/Joanna.mp3, Joanna.marks, Matthew.mp3, Matthew.marks</li> 
</ul> 
<p>Each page has its own set of voice-specific MP3 and marks files. These files are the pre-generated files. The page doesn’t need to invoke Amazon Polly at runtime; the files are part of the web build.</p> 
<h3>Config-driven case</h3> 
<p>Next, consider <code>gen_config.sh</code>:</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">cd ..
python FixHTML.py ../web/PRTPStaticConfig.html \
  example/tmp_wff.html
python <strong>ModGenericXSLT.py example/transform_config.json</strong> \
  <strong>example/tmp.xslt</strong>
.<strong>/gen_ssml.sh example/tmp_wff.html example/tmp.xslt</strong> \
  <strong>example/tmp.ssml</strong>
./run_polly.sh example/tmp.ssml en-US Joanna \
  ../web/polly/PRTPStaticConfig compass
./run_polly.sh example/tmp.ssml en-US Matthew \
  ../web/polly/PRTPStaticConfig compass</code></pre> 
</div> 
<p>The script is similar to the script in the default case, but the bolded lines indicate the main difference. Our approach is config-driven. We tailor the content to be extracted from the page by specifying what to extract through configuration, not code. In particular, we use the JSON file <code>transform_config.json</code>, which specifies that the content to be included are the elements with IDs <code>title</code>, <code>main</code>, <code>maintable</code>, and <code>qbtable</code>. The element with ID <code>wrapup</code> should be excluded. See the following code:</p> 
<div class="hide-language"> 
 <pre><code class="lang-json">{
 "inclusions": [ 
 	{"id" : "title"} , 
 	{"id": "main"}, 
 	{"id": "maintable"}, 
 	{"id": "qbtable" }
 ],
 "exclusions": [
 	{"id": "wrapup"}
 ]
}</code></pre> 
</div> 
<p>We run the Python program <code>ModGenericXSLT.py</code> to modify <code>generic.xslt</code>, used in the default case, to use the inclusions and exclusions that we specify in <code>transform_config.json</code>. The program writes the results to a temp file (<code>example/tmp.xslt</code>), which it passes to <code>gen_ssml.sh</code> as its XSLT transform.</p> 
<p>This is a low-code option. The web publisher doesn’t need to know how to write XSLT. But they do need to understand the structure of the HTML page and the IDs used in its main organizing elements.</p> 
<h3>Customization case</h3> 
<p>Finally, consider <code>gen_custom.sh</code>:</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">cd ..
python FixHTML.py ../web/PRTPStaticCustom.html \
   example/tmp_wff.html
./gen_ssml.sh example/tmp_wff.html <strong>example/custom.xslt</strong> \ 
   example/tmp.ssml
./run_polly.sh example/tmp.ssml en-US Joanna \
   ../web/polly/PRTPStaticCustom compass
./run_polly.sh example/tmp.ssml en-US Matthew \
   ../web/polly/PRTPStaticCustom compass</code></pre> 
</div> 
<p>This script is nearly identical to the default script, except it uses its own XSLT—<code>example/custom.xslt</code>—rather than the generic XSLT. The following is a snippet of the XSLT:</p> 
<div class="hide-language"> 
 <pre><code class="lang-xml">&lt;!-- Use NW, SW, NE, SE order for main tiles! --&gt;
&lt;xsl:template match="*[@id='maintable']"&gt;
    &lt;mark&gt;
        &lt;xsl:attribute name="name"&gt;
        &lt;xsl:value-of select="stats:getMark(.)"/&gt;
        &lt;/xsl:attribute&gt;
    &lt;/mark&gt;
    &lt;xsl:variable name="tiles" select="./tbody"/&gt;
    &lt;xsl:variable name="tiles-nw" select="$tiles/tr[1]/td[1]"/&gt;
    &lt;xsl:variable name="tiles-ne" select="$tiles/tr[1]/td[2]"/&gt;
    &lt;xsl:variable name="tiles-sw" select="$tiles/tr[2]/td[1]"/&gt;
    &lt;xsl:variable name="tiles-se" select="$tiles/tr[2]/td[2]"/&gt;
    &lt;xsl:variable name="tiles-seq" select="($tiles-nw,  $tiles-sw, $tiles-ne, $tiles-se)"/&gt;
    &lt;xsl:for-each select="$tiles-seq"&gt;
         &lt;xsl:apply-templates /&gt;  
    &lt;/xsl:for-each&gt;
&lt;/xsl:template&gt;   

&lt;!-- Say Tom Brady load! --&gt;
&lt;xsl:template match="span[@style = 'color:blue']" &gt;
&lt;p&gt;
      &lt;mark&gt;
          &lt;xsl:attribute name="name"&gt;
          &lt;xsl:value-of select="prtp:getMark(.)"/&gt;
          &lt;/xsl:attribute&gt;
      &lt;/mark&gt;
      &lt;prosody volume="x-loud"&gt;Tom Brady&lt;/prosody&gt;
&lt;/p&gt;
&lt;/xsl:template&gt;
</code></pre> 
</div> 
<p>If you want to study the code in detail, refer to the scripts and programs in the GitHub repo.</p> 
<h3>Browser setup and highlights</h3> 
<p>The static pages include an HTML5 audio control, which takes as its audio source the MP3 file generated by Amazon Polly and residing on the web server:</p> 
<div class="hide-language"> 
 <pre><code class="lang-html">&lt;audio id="audio" controls&gt;
  &lt;source src="polly/PRTPStaticDefault/en/Joanna.mp3" type="audio/mpeg"&gt;
&lt;/audio&gt;</code></pre> 
</div> 
<p>At load time, the page also loads the Amazon Polly-generated marks file. This occurs in the <code>PRTP.js</code> file, which the HTML page includes. The following is a snippet of the marks file for <code>PRTPStaticDefault</code>:</p> 
<div class="hide-language"> 
 <pre><code class="lang-html">{“time”:11747,“type”:“sentence”,“start”:289,“end”:356,“value”:“PollyReadsThePage is a test page for audio readout with highlights.“}
{“time”:15784,“type”:“ssml”,“start”:363,“end”:403,“value”:“\/html\/body[1]\/div[2]\/p[2]“}
{“time”:16427,“type”:“sentence”,“start”:403,“end”:426,“value”:“Here are some features:“}
{<strong>“time”:17677,“type”:“ssml”,“start”:433,“end”:480,“value”:“\/html\/body[1]\/div[2]\/ul[1]\/li[1]“</strong>}
{<strong>“time”:18344,“type”:“sentence”,“start”:480,“end”:502,“value”:“Skips hidden paragraph”</strong>}
{“time”:19894,“type”:“ssml”,“start”:509,“end”:556,“value”:“\/html\/body[1]\/div[2]\/ul[1]\/li[2]“}
{“time”:20537,“type”:“sentence”,“start”:556,“end”:603,“value”:“Speaks but does not highlight collapsed content”}</code></pre> 
</div> 
<p>During audio playback, there is an audio timer event handler in <code>PRTP.js</code> that checks the audio’s current time, finds the text to highlight, finds its location on the page, and highlights it. The text to be highlighted is an entry of type <code>sentence</code> in the marks file. The location is the XPath expression in the name attribute of the entry of type SSML that precedes the sentence. For example, if the time is 18400, according to the marks file, the sentence to be highlighted is “Skips hidden paragraph,” which starts at 18334. The location is the SSML entry at time 17667: <code>/html/body[1]/div[2]/ul[1]/li[1]</code>.</p> 
<h2>Test dynamic pages</h2> 
<p>The page <code>PRTPDynamic.html</code> demonstrates dynamic audio readback using default, configuration-driven, and custom audio extraction approaches.</p> 
<h3>Default case</h3> 
<p>In your browser, navigate to <code>PRTPDynamic.html</code>. The page has one query parameter, <code>dynOption</code>, which accepts values <code>default</code>, <code>config</code>, and <code>custom</code>. It defaults to <code>default</code>, so you may omit it in this case. The page has two sections with dynamic content:</p> 
<ul> 
 <li><strong>Latest Articles</strong> – Changes frequently throughout the day</li> 
 <li><strong>Greek Philosophers Search By Date </strong>– Allows the visitor to search for Greek philosophers by date and shows the results in a table</li> 
</ul> 
<p><img alt="" class="alignnone wp-image-42704 size-full" height="691" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2022/09/15/Screen-Shot-2022-09-15-at-3.38.30-PM.png" width="650" /></p> 
<p>Create some content in the <strong>Greek Philosopher</strong> section by entering a date range of -800 to 0, as shown in the example. Then choose <strong>Find</strong>.</p> 
<p>Now play the audio by choosing <strong>Play</strong> in the audio control.</p> 
<p>Behind the scenes, the page runs the following code to render and play the audio:</p> 
<div class="hide-language"> 
 <pre><code class="lang-js">   buildSSMLFromDefault();
   chooseRenderAudio();
   setVoice();</code></pre> 
</div> 
<p>First it calls the function <code>buildSSMLFromDefault</code> in <code>PRTP.js</code> to extract most of the text from the HTML page body. That function walks the DOM tree, looking for text in common elements such as <code>p</code>, <code>h1</code>, <code>pre</code>, <code>span</code>, and <code>td</code>. It ignores text in elements that usually don’t contain text to be read aloud, such as <code>audio</code>, <code>option</code>, and <code>script</code>. It builds SSML markup to be input to Amazon Polly. The following is a snippet showing extraction of the first row from the <code>philosopher</code> table:</p> 
<div class="hide-language"> 
 <pre><code class="lang-html">&lt;speak&gt;
...
  &lt;p&gt;&lt;mark name="/HTML[1]/BODY[1]/DIV[3]/DIV[1]/DIV[1]/TABLE[1]/TBODY[1]/TR[2]/TD[1]"/&gt;<strong>Thales</strong>&lt;/p&gt;
  &lt;p&gt;&lt;mark name="/HTML[1]/BODY[1]/DIV[3]/DIV[1]/DIV[1]/TABLE[1]/TBODY[1]/TR[2]/TD[2]"/&gt;<strong>-624 to -546</strong>&lt;/p&gt;
  &lt;p&gt;&lt;mark name="/HTML[1]/BODY[1]/DIV[3]/DIV[1]/DIV[1]/TABLE[1]/TBODY[1]/TR[2]/TD[3]"/&gt;<strong>Miletus</strong>&lt;/p&gt;
  &lt;p&gt;&lt;mark name="/HTML[1]/BODY[1]/DIV[3]/DIV[1]/DIV[1]/TABLE[1]/TBODY[1]/TR[2]/TD[4]"/&gt;<strong>presocratic</strong>&lt;/p&gt;
...
&lt;/speak&gt;
</code></pre> 
</div> 
<p>The <code>chooseRenderAudio</code> function in <code>PRTP.js</code> begins by initializing the AWS SDK for Amazon Cognito, Amazon S3, and Amazon Polly. This initialization occurs only once. If <code>chooseRenderAudio</code> is invoked again because the content of the page has changed, the initialization is skipped. See the following code:</p> 
<div class="hide-language"> 
 <pre><code class="lang-js">AWS.config.region = env.REGION
AWS.config.credentials = new AWS.CognitoIdentityCredentials({
            IdentityPoolId: env.IDP});
audioTracker.sdk.connection = {
   polly: new AWS.Polly({apiVersion: '2016-06-10'}),
   s3: new AWS.S3()
};
</code></pre> 
</div> 
<p>It generates MP3 audio from Amazon Polly. The generation is synchronous for small SSML inputs and asynchronous (with output sent to the S3 bucket) for large SSML inputs (greater than 6,000 characters). In the synchronous case, we ask Amazon Polly to provide the MP3 file using a presigned URL. When the synthesized output is ready, we set the <code>src</code> attribute of the audio control to that URL and load the control. We then request the marks file and load it the same way as in the static case. See the following code:</p> 
<div class="hide-language"> 
 <pre><code class="lang-js">// create signed URL
const signer = new AWS.Polly.Presigner(pollyAudioInput, audioTracker.sdk.connection.polly);

// call Polly to get MP3 into signed URL
signer.getSynthesizeSpeechUrl(pollyAudioInput, function(error, url) {
  // Audio control uses signed URL
  audioTracker.audioControl.src =
    audioTracker.sdk.audio[audioTracker.voice];
  audioTracker.audioControl.load();

  // call Polly to get marks
  audioTracker.sdk.connection.polly.synthesizeSpeech(
    pollyMarksInput, function(markError, markData) {
    const marksStr = new
      TextDecoder().decode(markData.AudioStream);
    // load marks into page the same as with static
    doLoadMarks(marksStr);
  });
});</code></pre> 
</div> 
<h3>Config-driven case</h3> 
<p>In your browser, navigate to <code>PRTPDynamic.html?dynOption=config</code>. Play the audio. The audio playback is similar to the default case, but there are minor differences. In particular, some content is skipped.</p> 
<p>Behind the scenes, when using the <code>config</code> option, the page extracts content differently than in the default case. In the default case, the page uses <code>buildSSMLFromDefault</code>. In the config-driven case, the page specifies the sections it wants to include and exclude:</p> 
<div class="hide-language"> 
 <pre><code class="lang-js">const ssml = buildSSMLFromConfig({
	 "inclusions": [ 
	 	{"id": "title"}, 
	 	{"id": "main"}, 
	 	{"id": "maintable"}, 
	 	{"id": "phil-result"},
	 	{"id": "qbtable"}, 
	 ],
	 "exclusions": [
	 	{"id": "wrapup"}
	 ]
	});</code></pre> 
</div> 
<p>The <code>buildSSMLFromConfig</code> function, defined in <code>PRTP.js</code>, walks the DOM tree in each of the sections whose ID is provided under <code>inclusions</code>. It extracts content from each and combines them together, in the order specified, to form an SSML document. It excludes the sections specified under <code>exclusions</code>. It extracts content from each section in the same way <code>buildSSMLFromDefault</code> extracts content from the page body.</p> 
<h3>Customization case</h3> 
<p>In your browser, navigate to <code>PRTPDynamic.html?dynOption=custom</code>. Play the audio. There are three noticeable differences. Let’s note these and consider the custom code that runs behind the scenes:</p> 
<ul> 
 <li>It reads the main tiles in NW, SW, NE, SE order. The custom code gets each of these cell blocks from <code>maintable</code> and adds them to the SSML in NW, SW, NE, SE order:</li> 
</ul> 
<div class="hide-language"> 
 <pre><code class="lang-js">const nw = getElementByXpath("//*[@id='maintable']//tr[1]/td[1]");
const sw = getElementByXpath("//*[@id='maintable']//tr[2]/td[1]");
const ne = getElementByXpath("//*[@id='maintable']//tr[1]/td[2]");
const se = getElementByXpath("//*[@id='maintable']//tr[2]/td[2]");
[nw, sw, ne, se].forEach(dir =&gt; buildSSMLSection(dir, []));
</code></pre> 
</div> 
<ul> 
 <li>“Tom Brady” is spoken loudly. The custom code puts “Tom Brady” text inside an SSML <code>prosody</code> tag:</li> 
</ul> 
<div class="hide-language"> 
 <pre><code class="lang-js">if (cellText == "Tom Brady") {
   addSSMLMark(getXpathOfNode( node.childNodes[tdi]));
   startSSMLParagraph();
   startSSMLTag("prosody", {"volume": "x-loud"});
   addSSMLText(cellText);
   endSSMLTag();
   endSSMLParagraph();
}</code></pre> 
</div> 
<ul> 
 <li>It reads only the first three rows of the quarterback table. It reads the column headers for each row. Check the code in the GitHub repo to discover how this is implemented.</li> 
</ul> 
<h2>Clean up</h2> 
<p>To avoid incurring future charges, delete the CloudFormation stack.</p> 
<h2>Conclusion</h2> 
<p>In this post, we demonstrated a technical solution to a high-value business problem: how to use Amazon Polly to read the content of a webpage and highlight the content as it’s being read. We showed this using both static and dynamic pages. To extract content from the page, we used DOM traversal and XSLT. To facilitate highlighting, we used the speech marks capability in Amazon Polly.</p> 
<p>Learn more about Amazon Polly by visiting its <a href="https://aws.amazon.com/polly/" rel="noopener noreferrer" target="_blank">service page</a>.</p> 
<p>Feel free to ask questions in the comments.</p> 
<hr /> 
<h3>About the authors</h3> 
<p style="clear: both;"><img alt="" class="size-full wp-image-41918 alignleft" height="95" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2022/08/31/image009-2.jpg" width="95" /><strong>Mike Havey</strong> is a Solutions Architect for AWS with over 25 years of experience building enterprise applications. Mike is the author of two books and numerous articles. Visit his Amazon <a href="https://www.amazon.com/Michael-Havey/e/B001IO9JBI" rel="noopener noreferrer" target="_blank">author page</a> to read more.</p> 
<p style="clear: both;"><a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2022/08/31/Vineet-Kacchawaha-.jpeg"><img alt="" class="size-full wp-image-41911 alignleft" height="118" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2022/08/31/Vineet-Kacchawaha-.jpeg" width="100" /></a><strong>Vineet Kachhawaha</strong> is a Solutions Architect at AWS with expertise in machine learning. He is responsible for helping customers architect scalable, secure, and cost-effective workloads on AWS.</p>
