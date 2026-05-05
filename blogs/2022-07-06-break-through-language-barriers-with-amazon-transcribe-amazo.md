---
title: "Break through language barriers with Amazon Transcribe, Amazon Translate, and Amazon Polly"
url: "https://aws.amazon.com/blogs/machine-learning/break-through-language-barriers-with-amazon-transcribe-amazon-translate-and-amazon-polly/"
date: "Wed, 06 Jul 2022 22:37:08 +0000"
author: "Michael Tran"
feed_url: "https://aws.amazon.com/blogs/machine-learning/category/artificial-intelligence/amazon-polly/feed/"
---
<p><em><strong>April 2024: This post was reviewed and updated for accuracy.</strong></em></p> 
<p>Imagine a surgeon taking video calls with patients across the globe without the need of a human translator. What if a fledgling startup could easily expand their product across borders and into new geographical markets by offering fluid, accurate, multilingual customer support and sales, all without the need of a live human translator? What happens to your business when you’re no longer bound by language?</p> 
<p>It’s common today to have virtual meetings with international teams and customers that speak many different languages. Whether they’re internal or external meetings, meaning often gets lost in complex discussions and you may encounter language barriers that prevent you from being as effective as you could be.</p> 
<p>In this post, you will learn how to use three fully managed AWS services (<a href="https://aws.amazon.com/transcribe/" rel="noopener noreferrer" target="_blank">Amazon Transcribe</a>, <a href="https://aws.amazon.com/translate/" rel="noopener noreferrer" target="_blank">Amazon Translate</a>, and <a href="https://aws.amazon.com/polly/" rel="noopener noreferrer" target="_blank">Amazon Polly</a>) to produce a near-real-time speech-to-speech translator solution that can quickly translate a source speaker’s live voice input into a spoken, accurate, translated target language, all with zero machine learning (ML) experience.</p> 
<h2>Overview of solution</h2> 
<p>Our translator consists of three fully managed AWS ML services working together in a single Python script by using the <a href="https://aws.amazon.com/sdk-for-python/" rel="noopener noreferrer" target="_blank">AWS SDK for Python (Boto3)</a> for our text translation and text-to-speech portions, and an asynchronous streaming SDK for audio input transcription.</p> 
<h3>Amazon Transcribe: Streaming speech to text</h3> 
<p>The first service you use in our stack is Amazon Transcribe, a fully managed speech-to-text service that takes input speech and transcribes it to text. Amazon Transcribe has flexible ingestion methods, batch or streaming, because it accepts either stored audio files or streaming audio data. In this post, you use the <a href="https://aws.amazon.com/blogs/developer/transcribe-streaming-sdk-for-python-preview/" rel="noopener noreferrer" target="_blank">asynchronous Amazon Transcribe streaming SDK for Python</a>, which uses the HTTP/2 streaming protocol to stream live audio and receive live transcriptions.</p> 
<p>When we first built this prototype, Amazon Transcribe streaming ingestion didn’t support automatic language detection, but this is no longer the case as of November 2021. Both batch and streaming ingestion now support automatic language detection for all <a href="https://docs.aws.amazon.com/transcribe/latest/dg/supported-languages.html" rel="noopener noreferrer" target="_blank">supported languages</a>. In this post, we show how a parameter-based solution though a seamless multi-language parameterless design is possible through the use of streaming automatic language detection. After our transcribed speech segment is returned as text, you send a request to Amazon Translate to translate and return the results in our Amazon Transcribe <code>EventHandler</code> method.</p> 
<h3>Amazon Translate: State-of-the-art, fully managed translation API</h3> 
<p>Next in our stack is Amazon Translate, a neural machine translation service that delivers fast, high-quality, affordable, and customizable language translation. As of June of 2022, Amazon Translate supports translation across 75 languages, with new language pairs and improvements being made constantly. Amazon Translate uses deep learning models hosted on a highly scalable and resilient AWS Cloud architecture to quickly deliver accurate translations either in real time or batched, depending on your use case. Using Amazon Translate is straightforward and requires no management of underlying architecture or ML skills. Amazon Translate has several features, like creating and using a <a href="https://docs.aws.amazon.com/translate/latest/dg/how-custom-terminology.html" rel="noopener noreferrer" target="_blank">custom terminology</a> to handle mapping between industry-specific terms. For more information on Amazon Translate service limits, refer to <a href="https://docs.aws.amazon.com/translate/latest/dg/what-is-limits.html" rel="noopener noreferrer" target="_blank">Guidelines and limits</a>. After the application receives the translated text in our target language, it sends the translated text to Amazon Polly for immediate translated audio playback.</p> 
<h3>Amazon Polly: Fully managed text-to-speech API</h3> 
<p>Finally, you send the translated text to Amazon Polly, a fully managed text-to-speech service that can either send back lifelike audio clip responses for immediate streaming playback or batched and saved in <a href="http://aws.amazon.com/s3" rel="noopener noreferrer" target="_blank">Amazon Simple Storage Service</a> (Amazon S3) for later use. You can control various aspects of speech such as pronunciation, volume, pitch, speech rate, and more using standardized <a href="https://www.w3.org/TR/speech-synthesis11/" rel="noopener noreferrer" target="_blank">Speech Synthesis Markup Language</a> (SSML).</p> 
<p>You can synthesize speech for certain Amazon Polly <a href="https://docs.aws.amazon.com/polly/latest/dg/ntts-voices-main.html" rel="noopener noreferrer" target="_blank">Neural voices</a> using the Newscaster style to make them sound like a TV or radio newscaster. You can also detect when specific words or sentences in the text are being spoken based on the metadata included in the audio stream. This allows the developer to synchronize graphical highlighting and animations, such as the lip movements of an avatar, with the synthesized speech.</p> 
<p>You can modify the pronunciation of particular words, such as company names, acronyms, foreign words, or neologisms, for example “P!nk,” “ROTFL,” or “C’est la vie” (when spoken in a non-French voice), using custom lexicons.</p> 
<h2>Architecture overview</h2> 
<p>The following diagram illustrates our solution architecture.</p> 
<div class="wp-caption aligncenter" id="attachment_37310" style="width: 872px;">
 <img alt="Architectural Diagram" class="size-full wp-image-37310" height="421" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2022/06/03/ML-6711_ArchDiagram.png" width="862" />
 <p class="wp-caption-text" id="caption-attachment-37310">This diagram shows the data flow from the client device to Amazon Transcribe, Amazon Translate, and Amazon Polly</p>
</div> 
<p>The workflow is as follows:</p> 
<ol> 
 <li>Audio is ingested by the Python SDK.</li> 
 <li>Amazon Live Transcribe converts the speech to text, in 39 possible languages.</li> 
 <li>Amazon Translate converts the languages.</li> 
 <li>Amazon Polly converts text to speech.</li> 
 <li>Audio is outputted to speakers.</li> 
</ol> 
<h2>Prerequisites</h2> 
<p>You need a host machine set up with a microphone, speakers, and reliable internet connection. A modern laptop should work fine for this because no additional hardware is needed. Next, you need to set up the machine with some software tools.</p> 
<p>You must have Python 3.7+ installed to use the asynchronous Amazon Transcribe streaming SDK and for a Python module called <code>pyaudio</code>, which you use to control the machine’s microphone and speakers. This module depends on a C library called <code>portaudio.h</code>. If you encounter issues with <code>pyaudio</code> errors, we suggest checking your OS to see if you have the <code>portaudio.h</code> library installed.</p> 
<p>For authorization and authentication of service calls, you create an <a href="http://aws.amazon.com/iam" rel="noopener noreferrer" target="_blank">AWS Identity and Access Management</a> (IAM) service role with permissions to call the necessary AWS services. By configuring the <a href="http://aws.amazon.com/cli" rel="noopener noreferrer" target="_blank">AWS Command Line Interface</a> (AWS CLI) with this IAM service role, you can run our script on your machine without having to pass in keys or passwords, because the AWS libraries are written to use the configured AWS CLI user’s credentials. This is a convenient method for rapid prototyping and ensures our services are being called by an authorized identity. As always, follow the principle of least privilege when assigning IAM policies when creating an IAM user or role.</p> 
<p>To summarize, you need the following prerequisites:</p> 
<ul> 
 <li>A PC, Mac, or Linux machine with microphone, speakers, and internet connection</li> 
 <li>The <code>portaudio.h</code> C library for your OS (brew, apt get, wget), which is needed for pyaudio to work</li> 
 <li>AWS CLI 2.0 with properly authorized IAM user configured by running aws configure in the AWS CLI</li> 
 <li>Python 3.7+</li> 
 <li><a href="https://aws.amazon.com/blogs/developer/transcribe-streaming-sdk-for-python-preview/" rel="noopener noreferrer" target="_blank">The asynchronous Amazon Transcribe Python SDK</a></li> 
 <li>The following Python libraries: 
  <ul> 
   <li><code>boto3</code></li> 
   <li><code>amazon-transcribe</code></li> 
   <li><code>pyaudio</code></li> 
   <li><code>asyncio</code></li> 
   <li><code>concurrent</code></li> 
  </ul> </li> 
</ul> 
<h2>Implement the solution</h2> 
<p>You will be relying heavily on the asynchronous Amazon Transcribe streaming SDK for Python as a starting point, and are going to build on top of that specific SDK. After you have experimented with the streaming SDK for Python, you add <a href="https://github.com/awslabs/amazon-transcribe-streaming-sdk/blob/develop/examples/simple_mic.py" rel="noopener noreferrer" target="_blank">streaming microphone</a> input by using <code>pyaudio</code>, a commonly used Python open-source library used for manipulating audio data. Then you add Boto3 calls to Amazon Translate and Amazon Polly for our translation and text-to-speech functionality. Finally, you stream out translated speech through the computer’s speakers again with <code>pyaudio</code>. The Python module <code>concurrent</code> gives you the ability to run blocking code in its own asynchronous thread to play back your returned Amazon Polly speech in a seamless, non-blocking way.</p> 
<p>Let’s import all our necessary modules, transcribe streaming classes, and instantiate some globals:</p> 
<div class="hide-language"> 
 <pre><code class="lang-python">import boto3
import asyncio
import pyaudio
import concurrent
from amazon_transcribe.client import TranscribeStreamingClient
from amazon_transcribe.handlers import TranscriptResultStreamHandler
from amazon_transcribe.model import TranscriptEvent

polly = boto3.client('polly', region_name = 'us-west-2')
translate = boto3.client(service_name='translate', region_name='us-west-2', use_ssl=True)
pa = pyaudio.PyAudio()

# for mic stream, 1024 should work fine
default_frames = 1024

# current params are set up for English to Mandarin, modify to your liking
params = {}
params['source_language'] = "en"
params['target_language'] = "zh"
params['lang_code_for_polly'] = "cmn-CN"
params['voice_id'] = "Zhiyu"
params['lang_code_for_transcribe'] = "en-US"</code></pre> 
</div> 
<p>First, you use <code>pyaudio</code> to obtain the input device’s sampling rate, device index, and channel count:</p> 
<div class="hide-language"> 
 <pre><code class="lang-python"># try grabbing the default input device and see if we get lucky
default_input_device = pa.get_default_input_device_info()
# verify this is your microphone device 
print(default_input_device)
# if correct then set it as your input device and define some globals
input_device = default_input_device
input_channel_count = input_device["maxInputChannels"]
input_sample_rate = input_device["defaultSampleRate"]
input_dev_index = input_device["index"]
##### BLOCK OPTIONAL #####
print ("Available devices:\n")
for i in range(0, pa.get_device_count()):
    info = pa.get_device_info_by_index(i)
    print (str(info["index"])  + ": \t %s \n \t %s \n" % (info["name"], pa.get_host_api_info_by_index(info["hostApi"])["name"]))

# select the correct index from the above returned list of devices, for example zero
dev_index = 0 
input_device = pa.get_device_info_by_index(dev_index)
# set globals for microphone stream
input_channel_count = input_device["maxInputChannels"]
input_sample_rate = input_device["defaultSampleRate"]
input_dev_index = input_device["index"]</code></pre> 
</div> 
<p>If this isn’t working, you can also loop through and print your devices as shown in the following code, and then use the device index to retrieve the device information with <code>pyaudio</code>:</p> 
<div class="hide-language"> 
 <pre><code class="lang-python">print ("Available devices:\n")
 for i in range(0, pa.get_device_count()):
     info = pa.get_device_info_by_index(i)
     print (str(info["index"])  + ": \t %s \n \t %s \n" % (info["name"], p.get_host_api_info_by_index(info["hostApi"])["name"]))

 # select the correct index from the above returned list of devices, for example zero
 dev_index = 0 
 input_device = pa.get_device_info_by_index(dev_index)

 #set globals for microphone stream
 input_channel_count = input_device["maxInputChannels"]
 input_sample_rate = input_device["defaultSampleRate"]
 input_dev_index = input_device["index"]</code></pre> 
</div> 
<p>You use <code>channel_count</code>, <code>sample_rate</code>, and <code>dev_index</code> as parameters in a mic stream. In that stream’s callback function, you use an <code>asyncio</code> nonblocking thread-safe callback to put the input bytes of the mic stream into an <code>asyncio</code> input queue. Take note of the loop and input_queue objects created with <code>asyncio</code> and how they’re used in the following code:</p> 
<div class="hide-language"> 
 <pre><code class="lang-python"># This function wraps the raw input stream from the microphone forwarding
# the blocks to an asyncio.Queue.
async def mic_stream():
    loop = asyncio.get_event_loop()
    input_queue = asyncio.Queue()

    def callback(indata, frame_count, time_info, status):
        loop.call_soon_threadsafe(input_queue.put_nowait, indata)
        return (indata, pyaudio.paContinue)

    # Be sure to use the correct parameters for the audio stream that matches
    # the audio formats described for the source language you'll be using:
    # https://docs.aws.amazon.com/transcribe/latest/dg/streaming.html

    print(input_device)

    #Open stream
    stream = pa.open(format = pyaudio.paInt16,
                channels = input_channel_count,
                rate = int(input_sample_rate),
                input = True,
                frames_per_buffer = default_frames,
                input_device_index = input_dev_index,
                stream_callback=callback)

    # Initiate the audio stream and asynchronously yield the audio chunks
    # as they become available.
    stream.start_stream()
    print("started stream")
    while True:
        indata = await input_queue.get()
        yield indata</code></pre> 
</div> 
<p>Now when the generator function <code>mic_stream()</code> is called, it continually yields input bytes as long as there is microphone input data in the input queue.</p> 
<p>Now that you know how to get input bytes from the microphone, let’s look at how to write Amazon Polly output audio bytes to a speaker output stream:</p> 
<div class="hide-language"> 
 <pre><code class="lang-python"># text will come from MyEventsHandler
def aws_polly_tts(text):
    response = polly.synthesize_speech(
        Engine = 'standard',
        LanguageCode = params['lang_code_for_polly'],
        Text = text,
        VoiceId = params['voice_id'],
        OutputFormat = "pcm",
    )
    output_bytes = response['AudioStream']

    #play to the speakers
    write_to_speaker_stream(output_bytes)

# how to write audio bytes to speakers
def write_to_speaker_stream(output_bytes):
    # Consumes bytes in chunks to produce the response's output
    print("Streaming started...")
    chunk_len = 1024
    channels = 1
    sample_rate = 16000

    if output_bytes:
        polly_stream = pa.open(
                    format = pyaudio.paInt16,
                    channels = channels,
                    rate = sample_rate,
                    output = True,
                    )

        # this is a blocking call - will sort this out with concurrent later
        while True:
            data = output_bytes.read(chunk_len)
            polly_stream.write(data)

        # If there's no more data to read, stop streaming
            if not data:
                output_bytes.close()
                polly_stream.stop_stream()
                polly_stream.close()
                break
        print("Streaming completed.")
    else:
        print("Nothing to stream.")</code></pre> 
</div> 
<p>Now let’s expand on what you built in the post <a href="https://aws.amazon.com/blogs/developer/transcribe-streaming-sdk-for-python-preview/" rel="noopener noreferrer" target="_blank">Asynchronous Amazon Transcribe Streaming SDK for Python</a>. In the following code, you create an executor object using the <code>ThreadPoolExecutor</code> subclass with three workers with concurrent. You then add an Amazon Translate call on the finalized returned transcript in the EventHandler and pass that translated text, the executor object, and our <code>aws_polly_tts()</code> function into an <code>asyncio</code> loop with <code>loop.run_in_executor()</code>, which runs our Amazon Polly function (with translated input text) asynchronously at the start of next iteration of the <code>asyncio</code> loop.</p> 
<div class="hide-language"> 
 <pre><code class="lang-python"># use concurrent package to create an executor object with 3 workers ie threads
executor = concurrent.futures.ThreadPoolExecutor(max_workers=3)
class MyEventHandler(TranscriptResultStreamHandler):
    async def handle_transcript_event(self, transcript_event: TranscriptEvent):
        # If the transcription is finalized, send it to translate
        results = transcript_event.transcript.results
        if len(results) &gt; 0:
            if len(results[0].alternatives) &gt; 0:
                transcript = results[0].alternatives[0].transcript
                print("transcript:", transcript)
                print(results[0].channel_id)

                # See partial results: https://docs.aws.amazon.com/transcribe/latest/dg/streaming-partial-results.html
                if hasattr(results[0], "is_partial") and results[0].is_partial == False:
                    # translate only 1 channel. the other channel is a duplicate
                    if results[0].channel_id == "ch_0":
                        trans_result = translate.translate_text(
                            Text = transcript,
                            SourceLanguageCode = params['source_language'],
                            TargetLanguageCode = params['target_language']
                        )
                        print("translated text:" + trans_result.get("TranslatedText"))
                        text = trans_result.get("TranslatedText")

                        # we run aws_polly_tts with a non-blocking executor at every loop iteration
                        await loop.run_in_executor(executor, aws_polly_tts, text) </code></pre> 
</div> 
<p>Finally, we have the <code>loop_me()</code> function. In it, you define <code>write_chunks()</code>, which takes an Amazon Transcribe stream as an argument and asynchronously writes chunks of streaming mic input to it. You then use <code>MyEventHandler()</code> with the output transcription stream as its argument and create a handler object. Then you use await with <code>asyncio.gather()</code> and pass in the write_chunks() and handler with the handle_events() method to handle the eventual futures of these coroutines. Lastly, you gather all event loops and loop the <code>loop_me()</code> function with <code>run_until_complete()</code>. See the following code:</p> 
<div class="hide-language"> 
 <pre><code class="lang-python">async def loop_me():
    # Setup up our client with our chosen AWS region
    client = TranscribeStreamingClient(region="us-west-2")
    stream = await client.start_stream_transcription(
        language_code=params['lang_code_for_transcribe'],
        media_sample_rate_hz=int(input_sample_rate),
        number_of_channels = 2,
        enable_channel_identification=True,
        media_encoding="pcm",
    )

    recorded_frames = []
    async def write_chunks(stream):
        # This connects the raw audio chunks generator coming from the microphone
        # and passes them along to the transcription stream.
        print("getting mic stream")
        async for chunk in mic_stream():
            recorded_frames.append(chunk)
            await stream.input_stream.send_audio_event(audio_chunk=chunk)
        await stream.input_stream.end_stream()

    handler = MyEventHandler(stream.output_stream)
    await asyncio.gather(write_chunks(stream), handler.handle_events())

# write a proper while loop here
loop = asyncio.get_event_loop()
loop.run_until_complete(loop_me())
loop.close() </code></pre> 
</div> 
<p>When the preceding code is run together without errors, you can speak into the microphone and quickly hear your voice translated to Mandarin Chinese. The automatic language detection feature for Amazon Transcribe and Amazon Translate translates any supported input language into the target language. You can speak for quite some time and because of the non-blocking nature of the function calls, all your speech input is translated and spoken, making this an excellent tool for translating live speeches.</p> 
<h2>Conclusion</h2> 
<p>Although this post demonstrated how these three fully managed AWS APIs can function seamlessly together, we encourage you to think about how you could use these services in other ways to deliver multilingual support for services or media like multilingual closed captioning for a fraction of the current cost. Medicine, business, and even diplomatic relations could all benefit from an ever-improving, low-cost, low-maintenance translation service.</p> 
<p>For more information about the proof of concept code base for this use case check out our <a href="https://github.com/aws-samples/amazon-live-translation-polly-transcribe" rel="noopener noreferrer" target="_blank">Github</a>.</p> 
<hr /> 
<h3>About the Authors</h3> 
<p style="clear: both;"><a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2022/06/06/miketran.jpg"><img alt="" class="wp-image-37320 size-full alignleft" height="133" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2022/06/06/miketran.jpg" width="100" /></a><strong>Michael Tran</strong> is a Solutions Architect with Envision Engineering team at Amazon Web Services. He provides technical guidance and helps customers accelerate their ability to innovate through showing the art of the possible on AWS. He has built multiple prototypes around AI/ML, and IoT for our customers. You can contact me @Mike_Trann on Twitter.</p> 
<p style="clear: both;"><strong><a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2022/06/27/profile2-1.jpg"><img alt="" class="size-full wp-image-38793 alignleft" height="100" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2022/06/27/profile2-1.jpg" width="100" /></a>Cameron Wilkes</strong> is a Prototyping Architect on the AWS Industry Accelerator team. While on the team he delivered several ML based prototypes to customers to demonstrate the “Art of the Possible” of ML on AWS. He enjoys music production, off-roading and design.</p> 
<p style="clear: both;"><img alt="" class="alignleft size-full wp-image-73851" height="133" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/04/09/simon.jpeg" width="100" /><strong>Simeon Brüggenjürgen</strong> is a Solutions Architect at Amazon Web Services based in Munich, Germany. He works with enterprises to design highly scalable, cost-efficient and resilient architectures. Using Machine Learning and AI, he is passionate to transform never-ending streams of data into actionable insights.</p> 
<p style="clear: both;"></p> 
<hr /> 
<h4>Audit History</h4> 
<p style="clear: both;">Last reviewed and updated in April 2024 by Simeon Brüggenjürgen | Solutions Architect</p>
