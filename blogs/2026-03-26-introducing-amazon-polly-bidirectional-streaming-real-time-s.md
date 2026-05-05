---
title: "Introducing Amazon Polly Bidirectional Streaming: Real-time speech synthesis for conversational AI"
url: "https://aws.amazon.com/blogs/machine-learning/introducing-amazon-polly-bidirectional-streaming-real-time-speech-synthesis-for-conversational-ai/"
date: "Thu, 26 Mar 2026 17:10:20 +0000"
author: "Praveen Gadi"
feed_url: "https://aws.amazon.com/blogs/machine-learning/category/artificial-intelligence/amazon-polly/feed/"
---
<p>Building natural conversational experiences requires speech synthesis that keeps pace with real-time interactions. Today, we’re excited to announce the new <strong>Bidirectional Streaming API</strong> for <strong>Amazon Polly</strong>, enabling streamlined real-time text-to-speech (TTS) synthesis where you can start sending text and receiving audio simultaneously.</p> 
<p>This new API is built for conversational AI applications that generate text or audio incrementally, like responses from large language models (LLMs), where users must begin synthesizing audio before the full text is available. Amazon Polly already supports streaming synthesized audio back to users. The new API goes further focusing on bidirectional communication over HTTP/2, allowing for enhanced speed, lower latency, and streamlined usage.</p> 
<h3><strong>The challenge with traditional text-to-speech</strong></h3> 
<p>Traditional text-to-speech APIs follow a request-response pattern. This required you to collect the complete text before making a synthesis request. Amazon Polly streams audio back incrementally after a request is made, but the bottleneck is on the input side—you can’t begin sending text until it’s fully available. In conversational applications powered by LLMs, where text is generated token by token, this means waiting for the entire response before synthesis starts.</p> 
<p>Consider a virtual assistant powered by an LLM. The model generates tokens incrementally over several seconds. With traditional TTS, users must wait for:</p> 
<ol> 
 <li>The LLM to finish generating the complete response</li> 
 <li>The TTS service to synthesize the entire text</li> 
 <li>The audio to download before playback begins</li> 
</ol> 
<p>The new Amazon Polly bidirectional streaming API is designed to address these bottlenecks.</p> 
<h2><strong>What’s new: Bidirectional Streaming</strong></h2> 
<p>The <code>StartSpeechSynthesisStream</code> API introduces a fundamentally different approach:</p> 
<ul> 
 <li><strong>Send text incrementally</strong>: Stream text to Amazon Polly as it becomes available—no need to wait for complete sentences or paragraphs.</li> 
 <li><strong>Receive audio immediately</strong>: Get synthesized audio bytes back in real-time as they’re generated.</li> 
 <li><strong>Control synthesis timing</strong>: Use flush configuration to trigger immediate synthesis of buffered text.</li> 
 <li><strong>True duplex communication</strong>: Send and receive simultaneously over a single connection.</li> 
</ul> 
<p><strong>Key Components</strong></p> 
<table border="1px" cellpadding="10px" class="styled-table"> 
 <tbody> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Component</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Event Direction</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Direction</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Purpose</strong></td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><code>TextEvent</code></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Inbound</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Client → Amazon Polly</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Send text to be synthesized</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><code>CloseStreamEvent</code></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Inbound</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Client → Amazon Polly</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Signal end of text input</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><code>AudioEvent</code></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Outbound</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Amazon Polly → Client</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Receive synthesized audio chunks</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><code>StreamClosedEvent</code></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Outbound</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Amazon Polly → Client</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Confirmation of stream completion</td> 
  </tr> 
 </tbody> 
</table> 
<h2><strong>Comparison to traditional methods</strong></h2> 
<h3><strong>Traditional file separation implementations</strong></h3> 
<p>Previously, achieving low-latency TTS required application-level implementations:</p> 
<p><img alt="Architecture diagram showing traditional text-to-speech system with client application, chunking middleware server, and Amazon Polly connected via WebSocket and HTTP requests" class="alignnone size-full wp-image-126813" height="441" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/03/23/ml-18776-image-1.png" width="1265" /></p> 
<p>This approach required:</p> 
<ul> 
 <li>Server-side text separation logic</li> 
 <li>Multiple parallel Amazon Polly API calls</li> 
 <li>Complex audio reassembly</li> 
</ul> 
<h3><strong>After: Native Bidirectional Streaming</strong></h3> 
<p><img alt="Bidirectional streaming architecture diagram showing client application connected to Amazon Polly via single HTTP/2 stream with text input and audio output flowing in both directions" class="alignnone size-full wp-image-126814" height="403" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/03/23/ml-18776-image-2.png" width="891" /></p> 
<p>Benefits:</p> 
<ul> 
 <li>No separation logic required</li> 
 <li>Single persistent connection</li> 
 <li>Native streaming in both directions</li> 
 <li>Reduced infrastructure complexity</li> 
 <li>Lower latency</li> 
</ul> 
<h2><strong>Performance benchmarks</strong></h2> 
<p>To measure the real-world impact, we benchmarked both the traditional <code>SynthesizeSpeech</code> API and the new bidirectional <code>StartSpeechSynthesisStream</code> API against the same input: 7,045 characters of prose (970 words), using the Matthew voice with the Generative engine, MP3 output at 24kHz in us-west-2.</p> 
<p><strong>How we measured:</strong> Both tests simulate an LLM generating tokens at ~30 ms per word. The traditional API test buffers words until a sentence boundary is reached, then sends the complete sentence as a <code>SynthesizeSpeech</code> request and waits for the full audio response before continuing. These tests mirror how traditional TTS integrations work, because you must have the complete sentence before requesting synthesis. The bidirectional streaming API test sends each word to the stream as it arrives, allowing Amazon Polly to begin synthesis before the full text is available. Both tests use the same text, voice, and output configuration.</p> 
<table border="1px" cellpadding="10px" class="styled-table"> 
 <tbody> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Metric</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Traditional SynthesizeSpeech</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Bidirectional Streaming</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Improvement</strong></td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Total processing time</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">115,226 ms (~115s)</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">70,071 ms (~70s)</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>39% faster</strong></td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">API calls</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">27</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">1</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>27x fewer</strong></td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Sentences sent</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">27 (sequential)</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">27 (streamed as words arrive)</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">—</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Total audio bytes</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">2,354,292</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">2,324,636</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">—</td> 
  </tr> 
 </tbody> 
</table> 
<p>The key advantage is architectural: the bidirectional API allows sending input text and receiving synthesized audio simultaneously over a single connection. Instead of waiting for each sentence to accumulate before requesting synthesis, text is streamed to Amazon Polly word-by-word as the LLM produces it. For conversational AI, this means that Amazon Polly receives and processes text incrementally throughout generation, rather than receiving it all at once after the LLM finishes. The result is less time waiting for synthesis after generation completes—the overall end-to-end latency from prompt to fully delivered audio is significantly reduced.</p> 
<h2><strong>Technical implementation</strong></h2> 
<h3><strong>Getting started</strong></h3> 
<p>You can use the bidirectional streaming API with AWS SDK for Java-2x, JavaScript v3, .NET v4, C++, Go v2, Kotlin, PHP v3, Ruby v3, Rust, and Swift. Support for CLIs (AWS Command Line Interface (AWS CLI) v1 and v2, PowerShell v4 and v5), Python, .NET v3 are not currently supported. Here’s an example:</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash"><em>// Create the async Polly client</em>
PollyAsyncClient pollyClient = PollyAsyncClient.builder()
.region(Region.US_WEST_2)
.credentialsProvider(DefaultCredentialsProvider.create())
.build();

<em>// Create the stream request</em>
StartSpeechSynthesisStreamRequest request = StartSpeechSynthesisStreamRequest.builder()
.voiceId(VoiceId.JOANNA)
.engine(Engine.GENERATIVE)
.outputFormat(OutputFormat.MP3)
.sampleRate("24000")
.build();</code></pre> 
</div> 
<h3><strong>Sending text events</strong></h3> 
<p>Text is sent to Amazon Polly using a reactive streams Publisher. Each TextEvent contains text:</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">TextEvent textEvent = TextEvent.builder() .text("Hello, this is streaming text-to-speech!") .build();</code></pre> 
</div> 
<h3><strong>Handling audio events</strong></h3> 
<p>Audio arrives through a response handler with a visitor pattern:</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">StartSpeechSynthesisStreamResponseHandler responseHandler =
StartSpeechSynthesisStreamResponseHandler.builder()
.onResponse(response -&gt; System.out.println("Stream connected"))
.onError(error -&gt; handleError(error))
.subscriber(StartSpeechSynthesisStreamResponseHandler.Visitor.builder()
.onAudioEvent(audioEvent -&gt; {
<em>// Process audio chunk immediately</em>
byte[] audioData = audioEvent.audioChunk().asByteArray();
playOrBufferAudio(audioData);
})
.onStreamClosedEvent(event -&gt; {
System.out.println("Synthesis complete. Characters processed: "
+ event.requestCharacters());
})
.build())
.build();</code></pre> 
</div> 
<h3><strong>Complete example: streaming text from an LLM</strong></h3> 
<p>Here’s a practical example showing how to integrate bidirectional streaming with incremental text generation:</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">public class LLMIntegrationExample {

private final PollyAsyncClient pollyClient;
private Subscriber&lt;? super StartSpeechSynthesisStreamActionStream&gt; textSubscriber;

/**
<em> * Start a bidirectional stream and return a handle for sending text.</em>
 */
public CompletableFuture&lt;Void&gt; startStream(VoiceId voice, AudioConsumer audioConsumer) {
StartSpeechSynthesisStreamRequest request = StartSpeechSynthesisStreamRequest.builder()
.voiceId(voice)
.engine(Engine.GENERATIVE)
.outputFormat(OutputFormat.PCM)
.sampleRate("16000")
.build();

<em>// Publisher that allows external text injection</em>
Publisher&lt;StartSpeechSynthesisStreamActionStream&gt; textPublisher = subscriber -&gt; {
this.textSubscriber = subscriber;
subscriber.onSubscribe(new Subscription() {
@Override
public void request(long n) { /* Demand-driven by subscriber */ }
@Override
public void cancel() { textSubscriber = null; }
});
};

StartSpeechSynthesisStreamResponseHandler handler =
StartSpeechSynthesisStreamResponseHandler.builder()
.subscriber(StartSpeechSynthesisStreamResponseHandler.Visitor.builder()
.onAudioEvent(event -&gt; {
if (event.audioChunk() != null) {
audioConsumer.accept(event.audioChunk().asByteArray());
}
})
.onStreamClosedEvent(event -&gt; audioConsumer.complete())
.build())
.build();

return pollyClient.startSpeechSynthesisStream(request, textPublisher, handler);
}

/**
<em> * Send text file to the stream. Call this as LLM tokens arrive.</em>
 */
public void sendText(String text, boolean flush) {
if (textSubscriber != null) {
TextEvent event = TextEvent.builder()
.text(text)
.flushStreamConfiguration(FlushStreamConfiguration.builder()
.force(flush)
.build())
.build();
textSubscriber.onNext(event);
}
}

/**
<em> * Close the stream when text generation is complete.</em>
*/
public void closeStream() {
if (textSubscriber != null) {
textSubscriber.onNext(CloseStreamEvent.builder().build());
textSubscriber.onComplete();
}
}
}</code></pre> 
</div> 
<h3><strong>Integration pattern with LLM streaming</strong></h3> 
<p>The following shows how to integrate patterns with LLM streaming:</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash"><em>// Start the Polly stream</em>
pollyStreamer.startStream(VoiceId.JOANNA, audioPlayer::playChunk);// As LLM generates tokens...
llmClient.streamCompletion(prompt, token -&gt; {
<em> // Send each token to Polly</em>
<em>//Optionally Flush at sentence boundaries to force synthesis</em>
<em>//note the tradeoff here: you may get the audio sooner, but audio quality may be impacted</em>
boolean isSentenceEnd = token.endsWith(".") || token.endsWith("!") || token.endsWith("?");
pollyStreamer.sendText(token, isSentenceEnd);
});
<em>// When LLM completes</em>
pollyStreamer.closeStream();</code></pre> 
</div> 
<h2><strong>Business benefits</strong></h2> 
<h3><strong>Improved user experience</strong></h3> 
<p>Latency directly impacts user satisfaction. The faster users hear a response, the more natural and engaging the interaction feels. The bidirectional streaming API enables:</p> 
<ul> 
 <li><strong>Reduced perceived wait time</strong> – Audio playback begins while the LLM is still generating, masking backend processing time.</li> 
 <li><strong>Higher engagement</strong> – Faster, more responsive interactions lead to increased user retention and satisfaction.</li> 
 <li><strong>Streamlined implementation – </strong>The setup and management of the streaming solution is now a single API call with clear hooks and callbacks to remove the complexity.</li> 
</ul> 
<h3><strong>Reduced operational costs</strong></h3> 
<p>Streamlining your architecture translates directly to cost savings:</p> 
<table border="1px" cellpadding="10px" class="styled-table"> 
 <tbody> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Cost factor</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Traditional chunking</strong></td> 
   <td style="padding: 10px; border: 1px solid #dddddd;"><strong>Bidirectional Streaming</strong></td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Infrastructure</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">WebSocket servers, load balancers, chunking middleware</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Direct client-to-Amazon Polly connection</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Development</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Custom chunking logic, audio reassembly, error handling</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">SDK handles complexity</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Maintenance</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Multiple components to monitor and update</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Single integration point</td> 
  </tr> 
  <tr> 
   <td style="padding: 10px; border: 1px solid #dddddd;">API Calls</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Multiple calls per request (one per chunk)</td> 
   <td style="padding: 10px; border: 1px solid #dddddd;">Single streaming session</td> 
  </tr> 
 </tbody> 
</table> 
<p>Organizations can expect to reduce infrastructure costs by removing intermediate servers and decrease development time by using native streaming capability.</p> 
<h2><strong>Use cases</strong></h2> 
<p>The bidirectional streaming API is recommended for:</p> 
<ul> 
 <li><strong>Conversational AI Assistants</strong> – Stream LLM responses directly to speech</li> 
 <li><strong>Real-time Translation</strong> – Synthesize translated text as it’s generated</li> 
 <li><strong>Interactive Voice Response (IVR)</strong> – Dynamic, responsive phone systems</li> 
 <li><strong>Accessibility Tools</strong> – Real-time screen readers and text-to-speech</li> 
 <li><strong>Gaming</strong> – Dynamic NPC dialogue and narration</li> 
 <li><strong>Live Captioning</strong> – Audio output for live transcription systems</li> 
</ul> 
<h2><strong>Conclusion</strong></h2> 
<p>The new <strong>Bidirectional Streaming API</strong> for <strong>Amazon Polly</strong> represents a significant advancement in real-time speech synthesis. By enabling true streaming in both directions, it removes latency bottlenecks that have traditionally plagued conversational AI applications.</p> 
<p>Key takeaways:</p> 
<ol> 
 <li><strong>Reduced latency</strong> – Audio begins playing while text is still being generated</li> 
 <li><strong>Simplified architecture</strong> – No need for file separation workarounds or complex infrastructure</li> 
 <li><strong>Native LLM integration</strong> – Purpose-built for streaming text from language models</li> 
 <li><strong>Flexible control</strong> – Fine-grained control over synthesis timing with flush configuration</li> 
</ol> 
<p>Whether you’re building a virtual assistant, accessibility tool, or any application requiring responsive text-to-speech, the bidirectional streaming API provides the foundation for truly conversational experiences.</p> 
<h2><strong>Next steps</strong></h2> 
<p>The bidirectional streaming API is now Generally Available. To get started:</p> 
<ol> 
 <li>Update to the latest AWS SDK for Java 2.x with bidirectional streaming support</li> 
 <li>Review the <a href="https://docs.aws.amazon.com/polly/latest/dg/API_StartSpeechSynthesisStream.html" rel="noopener noreferrer" target="_blank">API documentation</a> for detailed reference</li> 
 <li>Try the example code in this post to experience the low-latency streaming</li> 
</ol> 
<p>We’re excited to see what you build with this new capability. Share your feedback and use cases with us!</p> 
<hr style="width: 80%;" /> 
<h2>About the authors</h2> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="Professional headshot of a middle-aged man with dark hair and graying beard wearing a light gray collared shirt against a white background" class="alignnone size-full wp-image-126815" height="301" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/03/23/ml-18776-image-3.png" width="233" />
  </div> 
  <h3 class="lb-h4">“Scott Mishra”</h3> 
  <p><a href="https://www.linkedin.com/in/scott-mishra/" rel="noopener" target="_blank">“Scott”</a> is Sr. Solutions Architect for Amazon Web Services. Scott is a trusted technical advisor helping enterprise customers architect and implement cloud solutions at scale. He drives customer success through technical leadership, architectural guidance, and innovative problem-solving while working with cutting-edge cloud technologies. Scott specializes in generative AI solutions.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="Professional headshot of a man with short dark hair and facial stubble wearing a white collared shirt against a dark background" class="alignnone size-full wp-image-126816" height="317" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/03/23/ml-18776-image-4.png" width="239" />
  </div> 
  <h3 class="lb-h4">“Praveen Gadi”</h3> 
  <p><a href="http://www.linkedin.com/in/pgadi" rel="noopener" target="_blank">“Praveen”</a> is a Sr. Solutions Architect for Amazon Web Services. Praveen is a trusted technical advisor to enterprise customers. He enables customers to achieve their business objectives and maximize their cloud investments. Praveen specializes in integration solutions and developer productivity.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="Professional headshot of a smiling Asian man with short dark hair wearing a gray and black checkered collared shirt against a dark background" class="alignnone wp-image-126817" height="290" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/03/23/ml-18776-image-5.png" width="235" />
  </div> 
  <h3 class="lb-h4">“Paul Wu”</h3> 
  <p><a href="https://www.linkedin.com/in/wuyp/" rel="noopener" target="_blank">“Paul”</a> is a Solutions Architect for Amazon Web Services. Paul is a trusted technical advisor to enterprise customers. He enables customers to achieve their business objectives and maximize their cloud investments</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="Professional headshot of a young man with brown hair and facial stubble wearing a gray t-shirt against a dark background" class="alignnone wp-image-126818" height="312" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2026/03/23/ml-18776-image-6.png" width="232" />
  </div> 
  <h3 class="lb-h4">“Damian Pukaluk”</h3> 
  <p><a href="https://www.linkedin.com/in/damian-pukaluk-4687a811b/">“Damian</a>” is a Software Development Engineer at AWS Polly.</p> 
 </div> 
</footer>
