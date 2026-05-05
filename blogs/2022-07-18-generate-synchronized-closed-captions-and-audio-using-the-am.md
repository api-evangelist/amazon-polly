---
title: "Generate synchronized closed captions and audio using the Amazon Polly subtitle generator"
url: "https://aws.amazon.com/blogs/machine-learning/generate-synchronized-closed-captions-and-audio-using-the-amazon-polly-subtitle-generator/"
date: "Mon, 18 Jul 2022 18:15:54 +0000"
author: "Abhishek Soni"
feed_url: "https://aws.amazon.com/blogs/machine-learning/category/artificial-intelligence/amazon-polly/feed/"
---
<p><a href="https://aws.amazon.com/polly/" rel="noopener noreferrer" target="_blank">Amazon Polly</a>, an AI generated text-to-speech service, enables you to automate and scale your interactive voice solutions, helping to improve productivity and reduce costs.</p> 
<p>As our customers continue to use Amazon Polly for its rich set of features and ease of use, we have observed a demand for the ability to simultaneously generate synchronized audio and subtitles or closed captions for a given text input. At AWS, we continuously work backward from our customer asks, so in this post, we outline a method to generate audio and subtitles at the same time for a given text.</p> 
<p>Although subtitles and captions are often used interchangeably, including in this post, there are subtle differences among them:</p> 
<ul> 
 <li><strong>Subtitles</strong> – In subtitles, text language displayed on the screen is different from the audio language and doesn’t display anything for non-dialogue like significant sounds. The primary objective is to reach the audience that doesn’t speak the audio language in the video.</li> 
 <li><strong>Captions (closed/open) </strong>– Captions display the dialogues being spoken in the audio in the same language. Its primary purpose is to increase accessibility in cases where the audio can’t be heard by the end consumer due to a range of issues. Closed captions are part of a different file than the audio/video source and can be turned off and on at the user’s discretion, whereas open captions are part of the video file and can’t be turned off by the user.</li> 
</ul> 
<h2>Benefits of using Amazon Polly to generate audio with subtitles or closed captions</h2> 
<p>Imagine the following use case: you prepare a slide-based presentation for an online learning portal. Each slide includes onscreen content and narration. The onscreen content is a basic outline, and the narration goes into detail. Instead of recording a human voice, which can be cumbersome and inconsistent, you can use Amazon Polly to generate the narration. Amazon Polly produces high-quality, consistent voices. There’s no need for post-production. In the future, if you need to update a portion of the presentation, you only need to update the affected slides. The voice matches the original slides. Additionally, when Amazon Polly generates your audio, captions are included that appear in time with the audio. You save time because there’s no manual recording involved, and save additional time when updates are needed. Your presentation also delivers more value because captions help students consume the content. It’s a win-win-win solution.</p> 
<p>There are a multitude of use cases for captions, such as advertisements in social spaces, gymnasiums, coffee shops, and other places where typically there is something on a television with the audio muted and music in the background; online training and classes; virtual meetings; public electronic announcements; watching videos while commuting without headphones and without disturbing co-passengers; and several more.</p> 
<p>Irrespective of the field of application, closed captioning can help with the following:</p> 
<ul> 
 <li><strong>Accessibility </strong>– People with hearing impairments can better consume your content.</li> 
 <li><strong>Retention </strong>– Online learning is easier for e-learners to grasp and retain when more human senses are involved.</li> 
 <li><strong>Reachability </strong>– Your content can reach people that have competing priorities, such as gaming and watching news simultaneously, or people who have a different native language than the audio language.</li> 
 <li><strong>Searchability </strong>– The content is searchable by search engines. Whereas videos can’t be searched optimally by most search engines, search engines can use the caption text files and make your content more discoverable.</li> 
 <li><strong>Social courtesy </strong>– Sometimes it may be rude to play audio because of your surroundings, or the audio could be difficult to hear because of the noise of your environment.</li> 
 <li><strong>Comprehension </strong>– The content is easier to comprehend irrespective of the accent of the speaker, native language of the speaker, or speed of speech. You can also take notes without repeatedly watching the same scene.</li> 
</ul> 
<h2>Solution overview</h2> 
<p>The library presented in this post uses Amazon Polly to generate sound and closed captions for an input text. You can easily integrate this library in your text-to-speech applications. It supports several audio formats, and captions in both VTT and SRT file formats, which are the most commonly used across the industry.</p> 
<p>In this post, we focus on the <code>PollyVTT()</code> syntax and options, and offer a few examples that demonstrate how to use the Python <code>SubtitleGeneratorForPolly</code> to simultaneously generate synchronous audio and subtitle files for a given text input. The output audio file format can be PCM(wav), OGG, or MP3, and the subtitle file format can be VTT or SRT. Furthermore, <code>SubtitleGeneratorForPolly</code> supports all Amazon Polly <code>synthesize_speech</code> parameters and adds to the rich Amazon Polly feature set.</p> 
<p>The <code>polly-vtt</code> library and its dependencies are available on <a href="https://github.com/aws-samples/amazon-polly-closed-caption-subtitle-generator" rel="noopener noreferrer" target="_blank">GitHub</a>.</p> 
<h2>Install and use the function</h2> 
<p>Before we look at some examples of using <code>PollyVTT()</code>, the function that powers <code>SubtitleGeneratorForPolly</code>, let’s look at the installation and syntax of it.</p> 
<p>Install the library using the following code:</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">pip install</code></pre> 
</div> 
<p>To run from the command line, you simply run <code>polly-vtt</code>:</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">Usage: polly-vtt [OPTIONS] BASE_FILENAME VOICE_ID OUTPUT_FORMAT TEXT</code></pre> 
</div> 
<p>The following code shows your options:</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">--caption-format TEXT 'srt' or 'vtt'
--help Show this message and exit. 

BASE_FILENAME: Base filename for both the audio and caption files 
VOICE_ID: Polly voice to use (Case-sensitive)
OUTPUT_FORMAT: Amazon Polly output format: pcm, mp3, ogg_vorbis 
TEXT: Full text to be digitized 
Caption format: srt or vtt</code></pre> 
</div> 
<p>Let’s look at a few examples now.</p> 
<h2>Example 1</h2> 
<p>This example generates a PCM audio file along with an SRT caption file for two simple sentences:</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">$ polly-vtt testfile Joanna pcm "this is a test. this is a second sentence." --caption-format srt 

testfile.wav written successfully.
testfile.wav.srt written successfully.
Total Audio Length: 0:00:03.017500 
# of Sentences: 2</code></pre> 
</div> 
<audio class="wp-audio-shortcode" controls="controls" id="audio-38489-4" preload="none" style="width: 100%;">
 <source src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2022/06/22/testfile.mp3?_=4" type="audio/mpeg" />
</audio> 
<h2>Example 2</h2> 
<p>This example demonstrates how to use a paragraph of text as input. This generates audio files in WAV, MP3, and OGG, and subtitles in SRT and VTT. The following example creates six files for the given input text:</p> 
<ul> 
 <li><code>pcm_testfile.wav</code></li> 
 <li><code>pcm_testfile.wav.vtt</code></li> 
 <li><code>mp3_testfile.mp3</code></li> 
 <li><code>mp3_testfile.mp3.vtt</code></li> 
 <li><code>ogg_testfile.ogg</code></li> 
 <li><code>ogg_testfile.ogg.srt</code></li> 
</ul> 
<p>See the following code:</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">from polly_vtt import PollyVTT 

text = "News content is shaped by its own unique characteristics. Sentences and paragraphs are usually short and highly in formative because writers have to compress information into a limited space. Depending on the theme, news articles may con tain relevant terminology, place names, abbreviations, people’s names, and quotes. Excellent news writing is clear, precis e, and avoids ambiguity. The writing is dynamic, especially in online articles, because content may get updated multiple times per day as new information becomes available." 

polly_vtt = PollyVTT() 

# pcm with VTT captions 
polly_vtt.generate( 
"pcm_testfile", 
Text=text, 
VoiceId="Joanna", 
OutputFormat="pcm", 
) 

# mp3 with VTT captions 
polly_vtt.generate( 
"mp3_testfile", 
Text=text, 
VoiceId="Joanna", 
OutputFormat="mp3", 
)
 
# ogg with SRT captions 
polly_vtt.generate( 
"ogg_testfile", 
"srt",
Text=text, 
VoiceId="Joanna", 
OutputFormat="ogg_vorbis", 
) </code></pre> 
</div> 
<audio class="wp-audio-shortcode" controls="controls" id="audio-38489-5" preload="none" style="width: 100%;">
 <source src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2022/07/13/testfile-2.mp3?_=5" type="audio/mpeg" />
</audio> 
<h2>Example 3</h2> 
<p>In most cases, however, you want to pass the text as an input file. The following is a Python example of this, with the same output as the previous example:</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">from polly_vtt import PollyVTT
import os
import boto3
import json

polly_vtt = PollyVTT()

try:
	f=open("input.txt", "r")
	print("file is opened")
	polly_vtt.generate(
	"pcm_testfile",
	Text=f.read(),
	VoiceId="Joanna",
	OutputFormat="pcm",
	)
	f.close()
except:
	print("error occurred while converting to PCM")
print("end of file")

# mp3 with VTT captions
try:
	f=open("input.txt", "r")
	print("file is opened")
	polly_vtt.generate(
	"mp3_testfile",
	Text=f.read(),
	VoiceId="Joanna",
	OutputFormat="mp3",
	)
	f.close()
except:
	print("error occurred while converting to MP3")
print("end of file")

# ogg with SRT captions
try:
	f=open("input.txt", "r")
	print("file is opened")
	polly_vtt.generate(
	"ogg_testfile",
	"srt",
	Text=f.read(),
	VoiceId="Joanna",
	OutputFormat="ogg_vorbis",
	)
	f.close()
except:
	print("error occurred while converting to OGG")
print("end of file")</code></pre> 
</div> 
<audio class="wp-audio-shortcode" controls="controls" id="audio-38489-6" preload="none" style="width: 100%;">
 <source src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2022/06/22/testfile-1.mp3?_=6" type="audio/mpeg" />
</audio> 
<p>The following is a testimonial post from the AWS internal training team of using Amazon Polly with closed captions:</p> 
<div class="wp-video" style="width: 640px;">
 <video class="wp-video-shortcode" controls="controls" height="360" id="video-38489-2" preload="metadata" width="640">
  <source src="https://d2908q01vomqb2.cloudfront.net/artifacts/07c4b951-46c8-461f-bc9c-df9a6cf27fea/machine-learning/testimonial-construct.mp4?_=2" type="video/mp4" />
 </video>
</div> 
<p>The following video offers a short demo of how the internal training team at AWS uses <code>PollyVTT()</code>:</p> 
<div class="wp-video" style="width: 640px;">
 <video class="wp-video-shortcode" controls="controls" height="360" id="video-38489-3" preload="metadata" width="640">
  <source src="https://d2908q01vomqb2.cloudfront.net/artifacts/07c4b951-46c8-461f-bc9c-df9a6cf27fea/machine-learning/convert-process.mp4?_=3" type="video/mp4" />
 </video>
</div> 
<h2>Conclusion</h2> 
<p>In this post, we shared a method to generate audio and subtitles at the same time for a given text. The <code>PollyVTT()</code> function and <code>SubtitleGeneratorForPolly</code> address a common requirement for subtitles in an efficient and effective manner. The Amazon Polly team continues to invent and offer simplified solutions to complex customer requirements.</p> 
<p>For more tutorials and information about Amazon Polly, check out the <a href="https://aws.amazon.com/blogs/machine-learning/category/artificial-intelligence/amazon-polly/" rel="noopener noreferrer" target="_blank">AWS Machine Learning Blog</a>.</p> 
<hr /> 
<h3>About the Authors</h3> 
<p style="clear: both;"><strong><a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2022/06/06/abhishek-soni.jpg"><img alt="" class="size-full wp-image-37365 alignleft" height="133" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2022/06/06/abhishek-soni.jpg" width="100" /></a>Abhishek Soni</strong> is a Partner Solutions Architect at AWS. He works with customers to provide technical guidance for the best outcome of workloads on AWS.</p> 
<p style="clear: both;"><strong><a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2022/06/22/image002-3.png"><img alt="" class="size-full wp-image-38497 alignleft" height="124" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2022/06/22/image002-3.png" width="100" /></a> Dan</strong> <strong>McKee</strong> uses audio, video, and coffee to distill content into targeted, modular, and structured courses. In his role as Curriculum Developer Project Manager for the NetSec Domain at Amazon Web Services, he leverages his experience in Data Center Networking to help subject matter experts bring ideas to life.</p> 
<p style="clear: both;"><strong><a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2022/06/22/image004-2.png"><img alt="" class="size-full wp-image-38498 alignleft" height="107" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2022/06/22/image004-2.png" width="100" /></a>Orlando Karam</strong> is a Technical Curriculum Developer at Amazon Web Services, which means he gets to play with cool new technologies and then talk about it. Occasionally, he also uses those cool technologies to make his job easier.</p>
