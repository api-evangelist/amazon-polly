---
title: "Video auto-dubbing using Amazon Translate, Amazon Bedrock, and Amazon Polly"
url: "https://aws.amazon.com/blogs/machine-learning/video-auto-dubbing-using-amazon-translate-amazon-bedrock-and-amazon-polly/"
date: "Mon, 15 Jul 2024 17:00:24 +0000"
author: "Na Yu"
feed_url: "https://aws.amazon.com/blogs/machine-learning/category/artificial-intelligence/amazon-polly/feed/"
---
<p><em>This post is co-written with MagellanTV and Mission Cloud.&nbsp;</em></p> 
<p>Video dubbing, or content localization, is the process of replacing the original spoken language in a video with another language while synchronizing audio and video. Video dubbing has emerged as a key tool in breaking down linguistic barriers, enhancing viewer engagement, and expanding market reach. However, traditional dubbing methods are costly (<a href="https://aws.amazon.com/blogs/smb/how-one-smb-automates-content-localization-with-generative-artificial-intelligence/" rel="noopener" target="_blank">about $20 per minute with human review effort</a>) and time consuming, making them a common challenge for companies in the Media &amp; Entertainment (M&amp;E) industry. Video auto-dubbing that uses the power of <a href="https://aws.amazon.com/generative-ai/" rel="noopener" target="_blank">generative artificial intelligence (generative AI</a>) offers creators an affordable and efficient solution.</p> 
<p>This post shows you a cost-saving solution for video auto-dubbing. We use <a href="https://aws.amazon.com/translate/" rel="noopener" target="_blank">Amazon Translate</a> for initial translation of video captions and use <a href="https://aws.amazon.com/bedrock/" rel="noopener" target="_blank">Amazon Bedrock</a> for post-editing to further improve the translation quality. Amazon Translate is a neural machine translation service that delivers fast, high-quality, and affordable language translation.</p> 
<p>Amazon Bedrock is a fully managed service that offers a choice of high-performing foundation models (FMs) from leading AI companies such as AI21 Labs, Anthropic, Cohere, Meta, Mistral AI, Stability AI, and Amazon through a single API, along with a broad set of capabilities to help you build generative AI applications with security, privacy, and responsible AI.</p> 
<p><a href="https://www.magellantv.com/" rel="noopener" target="_blank">MagellanTV</a>, a leading streaming platform for documentaries, wants to broaden its global presence through content internationalization. Faced with manual dubbing challenges and prohibitive costs, MagellanTV sought out AWS Premier Tier Partner <a href="https://www.missioncloud.com/" rel="noopener" target="_blank">Mission Cloud</a> for an innovative solution.</p> 
<p>Mission Cloud’s solution distinguishes itself with idiomatic detection and automatic replacement, seamless automatic time scaling, and flexible batch processing capabilities with increased efficiency and scalability.</p> 
<h2>Solution overview</h2> 
<p>The following diagram illustrates the solution architecture. The inputs of the solution are specified by the user, including the folder path containing the original video and caption file, target language, and toggles for idiom detector and formality tone. You can specify these inputs in an Excel template and upload the Excel file to a designated <a href="https://aws.amazon.com/s3/" rel="noopener" target="_blank">Amazon Simple Storage Service (Amazon S3)</a> bucket. This will launch the whole pipeline. The final outputs are a dubbed video file and a translated caption file.</p> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/06/14/image001_video_dubbing.png"><img alt="image001_video_dubbing.png" class="aligncenter wp-image-78664 " height="489" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/06/14/image001_video_dubbing.png" width="856" /></a></p> 
<p>We use <a href="https://aws.amazon.com/translate/" rel="noopener" target="_blank">Amazon Translate</a> to translate the video caption, and <a href="https://aws.amazon.com/bedrock/" rel="noopener" target="_blank">Amazon Bedrock</a> to enhance the translation quality and enable automatic time scaling to synchronize audio and video. We use <a href="https://aws.amazon.com/augmented-ai/" rel="noopener" target="_blank">Amazon Augmented AI</a> for editors to review the content, which is then sent to <a href="https://aws.amazon.com/polly/" rel="noopener" target="_blank">Amazon Polly</a> to generate synthetic voices for the video. To assign a gender expression that matches the speaker, we developed a model to predict the gender expression of the speaker.</p> 
<p>In the backend, <a href="https://aws.amazon.com/step-functions/" rel="noopener" target="_blank">AWS Step Functions</a> orchestrates the preceding steps as a pipeline. Each step is run on <a href="https://aws.amazon.com/lambda/" rel="noopener" target="_blank">AWS Lambda</a> or <a href="https://aws.amazon.com/batch/" rel="noopener" target="_blank">AWS Batch</a>. By using the infrastructure as code (IaC) tool, <a href="https://aws.amazon.com/cloudformation/" rel="noopener" target="_blank">AWS CloudFormation</a>, the pipeline becomes reusable for dubbing new foreign languages.</p> 
<p>In the following sections, you will learn how to use the unique features of Amazon Translate for setting formality tone and for custom terminology. You will also learn how to use Amazon Bedrock to further improve the quality of video dubbing.</p> 
<h2>Why choose Amazon Translate?</h2> 
<p>We chose Amazon Translate to translate video captions based on three factors.</p> 
<ul> 
 <li>Amazon Translate supports over 75 languages. While the landscape of large language models (LLMs) has continuously evolved in the past year and continues to change, many of the trending LLMs support a smaller set of languages.</li> 
 <li>Our translation professional rigorously evaluated Amazon Translate in our review process and affirmed its commendable translation accuracy. <a href="https://www.welocalize.com/do-llms-or-mt-engines-perform-translation-better/" rel="noopener" target="_blank">Welocalize</a> benchmarks the performance of using LLMs and machine translations and recommends using LLMs as a post-editing tool.</li> 
 <li>Amazon Translate has <a href="https://aws.amazon.com/translate/faqs/#:~:text=Amazon" rel="noopener" target="_blank">various unique benefits</a>. For example, you can add custom terminology glossaries, while for LLMs, you might need fine-tuning that can be labor-intensive and costly.</li> 
</ul> 
<h3>Use Amazon Translate for custom terminology</h3> 
<p>Amazon Translate allows you to input a <a href="https://aws.amazon.com/blogs/machine-learning/customize-amazon-translate-output-to-meet-your-domain-and-organization-specific-vocabulary/" rel="noopener" target="_blank">custom terminology dictionary</a>, ensuring translations reflect the organization’s vocabulary or specialized terminology. We use the custom terminology dictionary to compile frequently used terms within video transcription scripts.</p> 
<p>Here’s an example. In a documentary video, the caption file would typically display “(speaking in foreign language)” on the screen as the caption when the interviewee speaks in a foreign language. The sentence “(speaking in foreign language)” itself doesn’t have proper English grammar: it lacks the proper noun, yet it’s commonly accepted as an English caption display. When translating the caption into German, the translation also lacks the proper noun, which can be confusing to German audiences as shown in the code block that follows.</p> 
<div class="hide-language"> 
 <pre><code class="lang-python">## Translate - without custom terminology (default)
import boto3
# Initialize a session of Amazon Translate
translate=boto3.client(service_name='translate', region_name='us-east-1', use_ssl=True)
def translate_text(text, source_lang, target_lang):
    result=translate.translate_text(
        Text=text, 
        SourceLanguageCode=source_lang, 
        TargetLanguageCode=target_lang)
    return result.get('TranslatedText')
text="(speaking in a foreign language)"
output=translate_text(text, "en", "de")
print(output)
# Output: (in einer Fremdsprache sprechen)</code></pre> 
</div> 
<p>Because this phrase “(speaking in foreign language)” is commonly seen in video transcripts, we added this term to the custom terminology CSV file <code>translation_custom_terminology_de.csv</code> with the vetted translation and provided it in the Amazon Translate job. The translation output is as intended as shown in the following code.</p> 
<div class="hide-language"> 
 <pre><code class="lang-python">## Translate - with custom terminology
import boto3
import json
# Initialize a session of Amazon Translate
translate=boto3.client('translate')
with open('translation_custom_terminology_de.csv', 'rb') as ct_file:
    translate.import_terminology(
        Name='CustomTerminology_boto3',
        MergeStrategy='OVERWRITE',
        Description='Terminology for Demo through boto3',
        TerminologyData={
            'File':ct_file.read(),
            'Format':'CSV',
            'Directionality':'MULTI'
        }
    )
text="(speaking in foreign language)"
result=translate.translate_text(
    Text=text,
    TerminologyNames=['CustomTerminology_boto3_2024'], 
    SourceLanguageCode="en",
    TargetLanguageCode="de"
)
print(result['TranslatedText'])
# Output: (Person spricht in einer Fremdsprache)</code></pre> 
</div> 
<h3>Set formality tone in Amazon Translate</h3> 
<p>Some documentary genres tend to be more formal than others. Amazon Translate allows you to define the desired level of <a href="https://docs.aws.amazon.com/translate/latest/dg/customizing-translations-formality.html" rel="noopener" target="_blank">formality</a> for translations to supported target languages. By using the default setting (<em>Informal</em>) of Amazon Translate, the translation output in German for the phrase, “[Speaker 1] Let me show you something,” is informal, according to a professional translator.</p> 
<div class="hide-language"> 
 <pre><code class="lang-python">## Translate - with informal tone (default) 
import boto3
# Initialize a session of Amazon Translate
translate=boto3.client(service_name='translate', region_name='us-east-1', use_ssl=True)
def translate_text(text, source_lang,target_lang):
    result=translate.translate_text(
        Text=text, 
        SourceLanguageCode=source_lang, 
        TargetLanguageCode=target_lang)
    return result.get('TranslatedText')
text="[Speaker 1] Let me show you something."
output=translate_text(text, "en", "de")
print(output)
# Output: [Sprecher 1] Lass mich dir etwas zeigen.</code></pre> 
</div> 
<p>By adding the <em>Formal</em> setting, the output translation has a formal tone, which fits the documentary’s genre as intended.</p> 
<div class="hide-language"> 
 <pre><code class="lang-python">## Translate - with formal tone 
import boto3
# Initialize a session of Amazon Translate
translate=boto3.client(service_name='translate', region_name='us-east-1', use_ssl=True)
def translate_text(text, source_lang, target_lang):
    result=translate.translate_text(
        Text=text, 
        SourceLanguageCode=source_lang, 
        TargetLanguageCode=target_lang,
        Settings={'Formality':'FORMAL'})
    return result.get('TranslatedText')
text="[Speaker 1] Let me show you something."
output=translate_text(text, "en", "de")
print(output)
# Output: [Sprecher 1] Lassen Sie mich Ihnen etwas zeigen.</code></pre> 
</div> 
<h2>Use Amazon Bedrock for post-editing</h2> 
<p>In this section, we use Amazon Bedrock to improve the quality of video captions after we obtain the initial translation from Amazon Translate.</p> 
<h3>Idiom detection and replacement</h3> 
<p>Idiom detection and replacement is vital in dubbing English videos to accurately convey cultural nuances. Adapting idioms prevents misunderstandings, enhances engagement, preserves humor and emotion, and ultimately improves the global viewing experience. Hence, we developed an idiom detection function using Amazon Bedrock to resolve this issue.</p> 
<p>You can turn the idiom detector on or off by specifying the inputs to the pipeline. For example, for science genres that have fewer idioms, you can turn the idiom detector off. While, for genres that have more casual conversations, you can turn the idiom detector on. For a 25-minute video, the total processing time is about 1.5 hours, of which about 1 hour is spent on video preprocessing and video composing. Turning the idiom detector on only adds about 5 minutes to the total processing time.</p> 
<p>We have developed a function <code>bedrock_api_idiom</code> to detect and replace idioms using Amazon Bedrock. The function first uses Amazon Bedrock LLMs to detect idioms in the text and then replace them. In the example that follows, Amazon Bedrock successfully detects and replaces the input text “well, I hustle” to “I work hard,” which can be translated correctly into Spanish by using Amazon Translate.</p> 
<div class="hide-language"> 
 <pre><code class="lang-python">## A rare idiom is well-detected and rephrased by Amazon Bedrock 
text_rephrased=bedrock_api_idiom(text)
print(text_rephrased)
# Output: I work hard
response=translate_text(text_rephrased, "en", "es-MX")
print(response)
# Output: yo trabajo duro
response=translate_text(response, "es-MX", "en")
print(response)
# Output: I work hard</code></pre> 
</div> 
<h3>Sentence shortening</h3> 
<p>Third-party video dubbing tools can be used for time-scaling during video dubbing, which can be costly if done manually. In our pipeline, we used Amazon Bedrock to develop a sentence shortening algorithm for automatic time scaling.</p> 
<p>For example, a typical caption file consists of a section number, timestamp, and the sentence. The following is an example of an English sentence before shortening.</p> 
<p>Original sentence:</p> 
<p><code>A large portion of the solar energy that reaches our planet is reflected back into space or absorbed by dust and clouds.</code></p> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/06/14/image002_video_dubbing.png"><img alt="image002_video_dubbing.pn" class="alignnone wp-image-78665 size-full" height="82" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/06/14/image002_video_dubbing.png" style="margin: 10px 0px 10px 0px;" width="500" /></a></p> 
<p>Here’s the shortened sentence using the sentence shortening algorithm. Using Amazon Bedrock, we can significantly improve the video-dubbing performance and reduce the human review effort, resulting in cost saving.</p> 
<p>Shortened sentence:</p> 
<p><code>A large part of solar energy is reflected into space or absorbed by dust and clouds.</code></p> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/06/14/image003_video_dubbing.png"><img alt="image003_video_dubbing.pn" class="alignnone wp-image-78666 size-full" height="69" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/06/14/image003_video_dubbing.png" style="margin: 10px 0px 10px 0px;" width="463" /></a></p> 
<h2>Conclusion</h2> 
<p>This new and constantly developing pipeline has been a revolutionary step for MagellanTV because it efficiently resolved some challenges they were facing that are common within Media &amp; Entertainment companies in general. The unique localization pipeline developed by Mission Cloud creates a new frontier of opportunities to distribute content across the world while saving on costs. Using generative AI in tandem with brilliant solutions for idiom detection and resolution, sentence length shortening, and custom terminology and tone results in a truly special pipeline bespoke to MagellanTV’s growing needs and ambitions.</p> 
<p>If you want to learn more about this use case or have a consultative session with the <a href="https://www.missioncloud.com/" rel="noopener" target="_blank">Mission</a> team to review your specific generative AI use case, feel free to request one through <a href="https://aws.amazon.com/marketplace/pp/prodview-cbpvma227mjym?sr=0-3&amp;ref_=beagle&amp;applicationId=AWSMPContessa" rel="noopener" target="_blank">AWS Marketplace</a>.</p> 
<hr /> 
<h3>About the Authors</h3> 
<p style="clear: both;"><strong><a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/06/14/Na_Yu.jpeg"><img alt="" class="wp-image-78754 size-full alignleft" height="100" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/06/16/Na_Yu-1.jpeg" width="100" /></a>Na Yu</strong> is a Lead GenAI Solutions Architect at Mission Cloud, specializing in developing ML, MLOps, and GenAI solutions in AWS Cloud and working closely with customers. She received her Ph.D. in Mechanical Engineering from the University of Notre Dame.</p> 
<p style="clear: both;"><strong><a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/06/14/max_goff.jpeg"><img alt="" class="wp-image-78753 size-full alignleft" height="100" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/06/16/max_goff-1.jpeg" width="100" /></a>Max Goff</strong> is a data scientist/data engineer with over 30 years of software development experience. A published author, blogger, and music producer he sometimes dreams in A.I.</p> 
<p style="clear: both;"><strong><a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/06/14/marco.jpg"><img alt="" class="wp-image-78752 size-full alignleft" height="123" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/06/16/marco-1.jpg" width="100" /></a>Marco Mercado</strong> is a Sr. Cloud Engineer specializing in developing cloud native solutions and automation. He holds multiple AWS Certifications and has extensive experience working with high-tier AWS partners. Marco excels at leveraging cloud technologies to drive innovation and efficiency in various projects.</p> 
<p style="clear: both;"><strong><a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/06/14/yaoqi.jpg"><img alt="" class="wp-image-78751 size-full alignleft" height="133" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/06/16/yaoqi.jpg" width="100" /></a>Yaoqi Zhang</strong> is a Senior Big Data Engineer at Mission Cloud. She specializes in leveraging AI and ML to drive innovation and develop solutions on AWS. Before Mission Cloud, she worked as an ML and software engineer at Amazon for six years, specializing in recommender systems for Amazon fashion shopping and NLP for Alexa. She received her Master of Science Degree in Electrical Engineering from Boston University.</p> 
<p style="clear: both;"><strong><a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/06/14/Adrian.jpeg"><img alt="" class="wp-image-78750 size-full alignleft" height="108" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/06/16/Adrian-1.jpeg" width="100" /></a>Adrian Martin</strong> is a Big Data/Machine Learning Lead Engineer at Mission Cloud. He has extensive experience in English/Spanish interpretation and translation.</p> 
<p style="clear: both;"><strong><a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/06/14/Ryan.jpg"><img alt="" class="wp-image-78749 size-full alignleft" height="136" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/06/16/Ryan.jpg" width="100" /></a>Ryan Ries</strong> holds over 15 years of leadership experience in data and engineering, over 20 years of experience working with AI and 5+ years helping customers build their AWS data infrastructure and AI models. After earning his Ph.D. in Biophysical Chemistry at UCLA and Caltech, Dr. Ries has helped develop cutting-edge data solutions for the U.S. Department of Defense and a myriad of Fortune 500 companies.</p> 
<p style="clear: both;"><strong><a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/06/14/Andrew.jpg"><img alt="" class="wp-image-78748 size-full alignleft" height="110" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/06/16/Andrew-1.jpg" width="100" /></a>Andrew Federowicz</strong> is the IT and Product Lead Director for Magellan VoiceWorks at MagellanTV. With a decade of experience working in cloud systems and IT in addition to a degree in mechanical engineering, Andrew designs builds, deploys, and scales inventive solutions to unique problems. Before Magellan VoiceWorks, Andrew architected and built the AWS infrastructure for MagellanTV’s 24/7 globally available streaming app. In his free time, Andrew enjoys sim racing and horology.</p> 
<p style="clear: both;"><strong><a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/06/14/work_img_cropped.png"><img alt="" class="wp-image-78747 size-full alignleft" height="108" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/06/16/work_img_cropped-1.png" width="100" /></a>Qiong Zhang</strong>, PhD, is a Sr. Partner Solutions Architect at AWS, specializing in AI/ML. Her current areas of interest include federated learning, distributed training, and generative AI. She holds 30+ patents and has co-authored 100+ journal/conference papers. She is also the recipient of the Best Paper Award at IEEE NetSoft 2016, IEEE ICC 2011, ONDM 2010, and IEEE GLOBECOM 2005.</p> 
<p style="clear: both;"><strong><a href="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/06/14/torres.jpg"><img alt="" class="wp-image-78755 size-full alignleft" height="100" src="https://d2908q01vomqb2.cloudfront.net/f1f836cb4ea6efb2a0b1b99f41ad8b103eff4b59/2024/06/16/torres-1.jpg" width="100" /></a>Cristian Torres</strong> is a Sr. Partner Solutions Architect at AWS. He has 10 years of experience working in technology performing several roles such as: Support Engineer, Presales Engineer, Sales Specialist and Solutions Architect. He works as a generalist with AWS services focusing on Migrations to help strategic AWS Partners develop successfully from a technical and business perspective.</p>
