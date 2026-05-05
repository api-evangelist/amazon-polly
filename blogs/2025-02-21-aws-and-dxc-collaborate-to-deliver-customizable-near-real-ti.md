---
title: "AWS and DXC collaborate to deliver customizable, near real-time voice-to-voice translation capabilities for Amazon Connect"
url: "https://aws.amazon.com/blogs/machine-learning/aws-and-dxc-collaborate-to-deliver-customizable-near-real-time-voice-to-voice-translation-capabilities-for-amazon-connect/"
date: "Fri, 21 Feb 2025 17:08:18 +0000"
author: "Milos Cosic"
feed_url: "https://aws.amazon.com/blogs/machine-learning/category/artificial-intelligence/amazon-polly/feed/"
---
<p>Providing effective multilingual customer support in global businesses presents significant operational challenges. Through collaboration between AWS and DXC Technology, we’ve developed a scalable voice-to-voice (V2V) translation prototype that transforms how contact centers handle multi-lingual customer interactions.</p> 
<p>In this post, we discuss how AWS and DXC used <a href="https://aws.amazon.com/connect/" rel="noopener" target="_blank">Amazon Connect</a> and other AWS AI services to deliver near real-time V2V translation capabilities.</p> 
<h2>Challenge: Serving customers in multiple languages</h2> 
<p>In Q3 2024, DXC Technology approached AWS with a critical business challenge: their global contact centers needed to serve customers in multiple languages without the exponential cost of hiring language-specific agents for the lower volume languages. Previously, DXC had explored several existing alternatives but found limitations in each approach – from communication constraints to infrastructure requirements that impacted reliability, scalability, and operational costs. DXC and AWS decided to organize a focused hackathon where DXC and AWS Solution Architects collaborated to:</p> 
<ul> 
 <li>Define essential requirements for real-time translation</li> 
 <li>Establish latency and accuracy benchmarks</li> 
 <li>Create seamless integration paths with existing systems</li> 
 <li>Develop a phased implementation strategy</li> 
 <li>Prepare and test an initial proof of concept setup</li> 
</ul> 
<h2>Business impact</h2> 
<p>For DXC, this prototype was used as an enabler, allowing technical talent maximization, operational transformation, and cost improvements through:</p> 
<ul> 
 <li>Best technical expertise delivery – Hiring and matching agents based on technical knowledge rather than spoken language, making sure customers get top technical support regardless of language barriers</li> 
 <li>Global operational flexibility – Removing geographical and language constraints in hiring, placement, and support delivery while maintaining consistent service quality across all languages</li> 
 <li>Cost reduction – Eliminating multi-language expertise premiums, specialized language training, and infrastructure costs through pay-per-use translation model</li> 
 <li>Similar experience to native speakers – Maintaining natural conversation flow with near real-time translation and audio feedback, while delivering premium technical support in customer’s preferred language</li> 
</ul> 
<h2>Solution overview</h2> 
<p>The Amazon Connect V2V translation prototype uses AWS advanced speech recognition and machine translation technologies to enable real-time conversation translation between agents and customers, allowing them to speak in their preferred languages while having natural conversations. It consists of the following key components:</p> 
<ul> 
 <li>Speech recognition – The customer’s spoken language is captured and converted into text using <a href="https://aws.amazon.com/transcribe/" rel="noopener" target="_blank">Amazon Transcribe</a>, which serves as the speech recognition engine. The transcript (text) is then fed into the machine translation engine.</li> 
 <li>Machine translation – <a href="https://aws.amazon.com/translate/" rel="noopener" target="_blank">Amazon Translate</a>, the machine translation engine, translates the customer’s transcript into the agent’s preferred language in near real time. The translated transcript is converted back into speech using <a href="https://aws.amazon.com/polly/" rel="noopener" target="_blank">Amazon Polly</a>, which serves as the text-to-speech engine.</li> 
 <li>Bidirectional translation – The process is reversed for the agent’s response, translating their speech into the customer’s language and delivering the translated audio to the customer.</li> 
 <li>Seamless integration – The V2V translation sample project integrates with Amazon Connect, enabling agents to handle customer interactions in multiple languages without any additional effort or training, using the <a href="https://github.com/amazon-connect/amazon-connect-streams" rel="noopener" target="_blank">Amazon Connect Streams JS</a> and <a href="https://github.com/aws/connect-rtc-js" rel="noopener" target="_blank">Amazon Connect RTC JS</a> libraries.</li> 
</ul> 
<p>The prototype can be extended with other AWS AI services to further customize the translation capabilities. It’s open source and ready for customization to meet your specific needs.</p> 
<p>The following diagram illustrates the solution architecture.</p> 
<p><img alt="" class="alignnone wp-image-99698 size-full" height="703" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/02/13/AmazonConnectV2V-AmazonConnectV2VArchitecture.png" style="margin: 10px 0px 10px 0px;" width="1262" /></p> 
<p>The following screenshot illustrates a sample agent web application.</p> 
<p><img alt="" class="alignnone wp-image-99702 size-full" height="992" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/02/13/AmazonConnectV2VScreenshot.png" style="margin: 10px 0px 10px 0px;" width="1904" /></p> 
<p>The user interface consists of three sections:</p> 
<ul> 
 <li>Contact Control Panel – A softphone client using Amazon Connect</li> 
 <li>Customer Controls – Customer-to-agent interaction controls, including Transcribe Customer Voice, Translate Customer Voice, and Synthesize Customer Voice</li> 
 <li>Agent controls – Agent-to-customer interaction controls, including Transcribe Agent Voice, Translate Agent Voice, and Synthesize Agent Voice</li> 
</ul> 
<h2>Challenges when implementing near real-time voice translation</h2> 
<p>The Amazon Connect V2V sample project was designed to minimize the audio processing time from the moment the customer or agent finishes speaking until the translated audio stream is started. However, even with the shortest audio processing time, the user experience still doesn’t match the experience of a real conversation when both are speaking the same language. This is due to the specific pattern of the customer only hearing the agent’s translated speech, and the agent only hearing the customer’s translated speech. The following diagram displays that pattern.</p> 
<p><img alt="" class="alignnone wp-image-99700 size-full" height="225" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/02/13/AmazonConnectV2V-NoStreamingAddOns.png" style="margin: 10px 0px 10px 0px;" width="1012" /></p> 
<p>The example workflow consists of the following steps:</p> 
<ol> 
 <li>The customer starts speaking in their own language, and speaks for 10 seconds.</li> 
 <li>Because the agent only hears the customer’s translated speech, the agent first hears 10 seconds of silence.</li> 
 <li>When customer finishes speaking, the audio processing time takes 1–2 seconds, during which time both the customer and agent hear silence.</li> 
 <li>The customer’s translated speech is streamed to the agent. During that time, the customer hears silence.</li> 
 <li>When the customer’s translated speech playback is complete, the agent starts speaking, and speaks for 10 seconds.</li> 
 <li>Because customer only hears the agent’s translated speech, the customer hears 10 seconds of silence.</li> 
 <li>When the agent finishes speaking, the audio processing time takes 1–2 seconds, during which time both the customer and agent hear silence.</li> 
 <li>The agent’s translated speech is streamed to the agent. During that time, the agent hears silence.</li> 
</ol> 
<p>In this scenario, the customer hears a single block of 22–24 seconds of a complete silence, from the moment they finished speaking until they hear the agent’s translated voice. This creates a suboptimal experience, because the customer might not be certain what is happening during these 22–24 seconds—for instance, if the agent was able to hear them, or if there was a technical issue.</p> 
<h2>Audio streaming add-ons</h2> 
<p>In a face-to-face conversation scenario between two people that don’t speak the same language, they might have another person as a translator or interpreter. An example workflow consists of the following steps:</p> 
<ol> 
 <li>Person A speaks in their own language, which is heard by Person B and the translator.</li> 
 <li>The translator translates what Person A said to Person B’s language. The translation is heard by Person B and Person A.</li> 
</ol> 
<p>Essentially, Person A and Person B hear each other speaking their own language, and they also hear the translation (from the translator). There’s no waiting in silence, which is even more important in non-face-to-face conversations (such as contact center interactions).</p> 
<p>To optimize the customer/agent experience, the Amazon Connect V2V sample project implements audio streaming add-ons to simulate a more natural conversation experience. The following diagram illustrates an example workflow.</p> 
<p><img alt="" class="alignnone wp-image-99701 size-full" height="225" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/02/13/AmazonConnectV2V-WithStreamingAddOns.png" style="margin: 10px 0px 10px 0px;" width="1012" /></p> 
<p>The workflow consists of the following steps:</p> 
<ol> 
 <li>The customer starts speaking in their own language, and speaks for 10 seconds.</li> 
 <li>The agent hears the customer’s original voice, at a lower volume (“Stream Customer Mic to Agent” enabled).</li> 
 <li>When the customer finishes speaking, the audio processing time takes 1–2 seconds. During that time, the customer and agent hear subtle audio feedback—contact center background noise—at a very low volume (“Audio Feedback” enabled).</li> 
 <li>The customer’s translated speech is then streamed to the agent. During that time, the customer hears their translated speech, at a lower volume (“Stream Customer Translation to Customer” enabled).</li> 
 <li>When the customer’s translated speech playback is complete, the agent starts speaking, and speaks for 10 seconds.</li> 
 <li>The customer hears the agent’s original voice, at a lower volume (“Stream Agent Mic to Customer” enabled).</li> 
 <li>When the agent finishes speaking, the audio processing time takes 1–2 seconds. During that time, the customer and agent hear subtle audio feedback—contact center background noise—at a very low volume (“Audio Feedback” enabled).</li> 
 <li>The agent’s translated speech is then streamed to the agent. During that time, the agent hears their translated speech, at a lower volume (“Stream Agent Translation to Agent” enabled).</li> 
</ol> 
<p>In this scenario, the customer hears two short blocks (1–2 seconds) of subtle audio feedback, instead of a single block of 22–24 seconds of complete silence. This pattern is much closer to a face-to-face conversation that includes a translator.</p> 
<p>The audio streaming add-ons provide additional benefits, including:</p> 
<ul> 
 <li>Voice characteristics – In cases when the agent and customer only hear their translated and synthesized speech, the actual voice characteristics are lost. For instance, the agent can’t hear if the customer was talking slow or fast, if the customer was upset or calm, and so on. The translated and synthesized speech doesn’t carry over that information.</li> 
 <li>Quality assurance – In cases when call recording is enabled, only the customer’s original voice and the agent’s synthesized speech are recorded, because the translation and the synthetization are done on the agent (client) side. This makes it difficult for QA teams to properly evaluate and audit the conversations, including the many silent blocks within it. Instead, when the audio streaming add-ons are enabled, there are no silent blocks, and the QA team can hear the agent’s original voice, the customer’s original voice, and their respective translated and synthesized speech, all in a single audio file.</li> 
 <li>Transcription and translation accuracy – Having both the original and translated speech available in the call recording makes it straightforward to detect specific words that would improve transcription accuracy (by using Amazon Transcribe custom vocabularies) or translation accuracy (using Amazon Translate custom terminologies), to make sure that your brand names, character names, model names, and other unique content are transcribed and translated to the desired result.</li> 
</ul> 
<h2>Get started with Amazon Connect V2V</h2> 
<p>Ready to transform your contact center’s communication? Our Amazon Connect V2V sample project is now available on <a href="https://github.com/aws-samples/connect-v2v-translation-with-cx-options/tree/main" rel="noopener" target="_blank">GitHub</a>. We invite you to explore, deploy, and experiment with this powerful prototype. You can it as a foundation for developing innovative multi-lingual communication solutions in your own contact center, through the following key steps:</p> 
<ol> 
 <li>Clone the GitHub repository.</li> 
 <li>Test different configurations for audio streaming add-ons.</li> 
 <li>Review the sample project’s limitations in the README.</li> 
 <li>Develop your implementation strategy: 
  <ol type="a"> 
   <li>Implement robust security and compliance controls that meet your organization’s standards.</li> 
   <li>Collaborate with your customer experience team to define your specific use case requirements.</li> 
   <li>Balance between automation and the agent’s manual controls (for example, use an Amazon Connect contact flow to automatically set contact attributes for preferred languages and audio streaming add-ons).</li> 
   <li>Use your preferred transcribe, translate, and text-to-speech engines, based on specific language support requirements and business, legal, and regional preferences.</li> 
   <li>Plan a phased rollout, starting with a pilot group, then iteratively optimize your transcription custom vocabularies and translation custom terminologies.</li> 
  </ol> </li> 
</ol> 
<h2>Conclusion</h2> 
<p>The Amazon Connect V2V sample project demonstrates how Amazon Connect and advanced AWS AI services can break down language barriers, enhance operational flexibility, and reduce support costs. Get started now and revolutionize how your contact center communicates across language barriers!</p> 
<hr /> 
<h3>About the Authors</h3> 
<p style="clear: both;"><strong><img alt="" class="size-full wp-image-99815 alignleft" height="133" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/02/14/mcosic.jpg" width="100" />Milos Cosic</strong> is a Principal Solutions Architect at AWS.</p> 
<p style="clear: both;"><img alt="" class="size-full wp-image-99814 alignleft" height="130" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/02/14/IMG_3095.jpg" width="100" /><strong>E</strong><strong>J F</strong><strong>errell</strong> is a Senior Solutions Architect at AWS.</p> 
<p style="clear: both;"><strong><img alt="" class="size-full wp-image-99813 alignleft" height="100" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2025/02/14/1725379171418.jpeg" width="100" /></strong><strong>Adam El Tanbouli</strong> is a Technical Program Manager for Prototyping and Support Services at DXC Modern Workplace.</p>
