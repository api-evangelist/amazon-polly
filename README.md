# Amazon Polly (amazon-polly)
Amazon Polly is a cloud service that converts text into lifelike speech, enabling you to create applications that talk and build entirely new categories of speech-enabled products. Polly supports multiple voices, languages, and audio output formats including neural and generative engines for natural-sounding speech.

**URL:** [https://aws.amazon.com/polly/](https://aws.amazon.com/polly/)

**Run:** [Capabilities Using Naftiko](https://github.com/naftiko/fleet?utm_source=api-evangelist&utm_medium=readme&utm_campaign=company-api-evangelist&utm_content=repo)

## Tags:

 - AI, AWS, Machine Learning, Speech Synthesis, Text-To-Speech, TTS, Voice, SSML, Neural Engine, Generative AI

## Timestamps

- **Created:** 2024-01-15
- **Modified:** 2026-04-19

## APIs

### Amazon Polly API
The Amazon Polly API enables you to synthesize speech from text (plain text or SSML), manage custom pronunciation lexicons, list available voices across multiple languages and engines, and manage asynchronous synthesis tasks with S3 output.

**Human URL:** [https://aws.amazon.com/polly/](https://aws.amazon.com/polly/)

#### Tags:

 - Speech Synthesis, Text-To-Speech, Voice, SSML, Neural Engine

#### Properties

- [Documentation](https://docs.aws.amazon.com/polly/latest/dg/what-is.html)
- [APIReference](https://docs.aws.amazon.com/polly/latest/dg/API_Reference.html)
- [OpenAPI](openapi/amazon-polly-openapi.yml)
- [OpenAPI](openapi/amazon-polly-openapi-original.yaml)
- [Pricing](https://aws.amazon.com/polly/pricing/)
- [GettingStarted](https://aws.amazon.com/polly/getting-started/)
- [FAQ](https://aws.amazon.com/polly/faqs/)
- [Features](https://aws.amazon.com/polly/features/)
- [Quotas](https://docs.aws.amazon.com/polly/latest/dg/limits.html)
- [Authentication](https://docs.aws.amazon.com/general/latest/gr/signature-version-4.html)
- [RateLimits](https://docs.aws.amazon.com/polly/latest/dg/limits.html)

## Common Properties

- [Portal](https://console.aws.amazon.com/polly/)
- [Blog](https://aws.amazon.com/blogs/machine-learning/category/artificial-intelligence/amazon-polly/)
- [Support](https://aws.amazon.com/premiumsupport/)
- [CLI](https://docs.aws.amazon.com/cli/latest/reference/polly/)
- [SDK](https://aws.amazon.com/tools/)
- [StatusPage](https://status.aws.amazon.com/)
- [TermsOfService](https://aws.amazon.com/service-terms/)
- [PrivacyPolicy](https://aws.amazon.com/privacy/)
- [Documentation](https://docs.aws.amazon.com/polly/)
- [Pricing](https://aws.amazon.com/polly/pricing/)
- [GettingStarted](https://aws.amazon.com/polly/getting-started/)
- [FAQ](https://aws.amazon.com/polly/faqs/)
- [SignUp](https://portal.aws.amazon.com/billing/signup)
- [GitHubOrganization](https://github.com/aws)
- [StackOverflow](https://stackoverflow.com/questions/tagged/amazon-polly)
- [CodeExamples](https://docs.aws.amazon.com/code-library/latest/ug/polly_code_examples.html)
- [Compliance](https://aws.amazon.com/compliance/)
- [SpectralRules](rules/amazon-polly-spectral-rules.yml)
- [NaftikoCapability](capabilities/text-to-speech.yaml)
- [Vocabulary](vocabulary/amazon-polly-vocabulary.yaml)

## Features

| Name | Description |
|------|-------------|
| Neural Text-to-Speech | Produce natural-sounding speech using neural network-based text-to-speech technology. |
| Generative Engine | New generative engine delivers the highest quality, most human-like speech synthesis. |
| Multiple Voices and Languages | Choose from 60+ voices across 30+ languages including male, female, and child voices. |
| SSML Support | Use Speech Synthesis Markup Language (SSML) to control pronunciation, volume, pitch, and speech rate. |
| Custom Lexicons | Create custom pronunciation lexicons to control how specific words and phrases are spoken. |
| Speech Marks | Generate speech marks metadata to synchronize spoken text with animations or visual highlights. |
| Asynchronous Synthesis Tasks | Process large text bodies asynchronously with S3 output for long-form content. |
| Multiple Audio Formats | Output audio in MP3, OGG, PCM, and JSON (speech marks) formats. |

## Use Cases

| Name | Description |
|------|-------------|
| Voice Assistants | Build conversational interfaces that speak responses to users. |
| Accessibility Features | Add text-to-speech reading to applications for visually impaired users. |
| Podcast and Audio Content | Convert written articles and content into audio podcasts automatically. |
| E-Learning Narration | Add spoken narration to educational courses and training materials. |
| Call Center IVR | Create interactive voice response systems with natural-sounding speech. |
| Language Learning Apps | Provide native-speaker pronunciation examples for language education. |

## Integrations

| Name | Description |
|------|-------------|
| Amazon S3 | Store synthesized speech output from asynchronous synthesis tasks in S3 buckets. |
| Amazon Lex | Combine Polly speech synthesis with Lex conversational AI for voice chatbots. |
| AWS Lambda | Trigger speech synthesis from Lambda functions for event-driven voice applications. |
| Amazon Transcribe | Pair Polly text-to-speech with Transcribe speech-to-text for round-trip voice applications. |
| Amazon Connect | Power Amazon Connect contact center voice responses with Polly neural speech. |

## Artifacts

Machine-readable API specifications organized by format.

### OpenAPI

- [Amazon Polly OpenAPI (simple)](openapi/amazon-polly-openapi.yml)
- [Amazon Polly OpenAPI (full - 9 operations)](openapi/amazon-polly-openapi-original.yaml)

### JSON Schema

23 schema files covering Amazon Polly API resource types including voices, lexicons, and synthesis tasks.

### JSON Structure

24 JSON Structure files converted from JSON Schema with strict typing.

### JSON-LD

- [Amazon Polly Context](json-ld/amazon-polly-context.jsonld)

### Examples

16 example JSON files generated from JSON Schema definitions.

## Capabilities

Naftiko capabilities organized as shared per-API definitions composed into customer-facing workflows.

### Shared Per-API Definitions

- [Amazon Polly API](capabilities/shared/amazon-polly.yaml) — 7 operations for text-to-speech synthesis

### Workflow Capabilities

| Workflow | APIs Combined | Tools | Persona |
|----------|--------------|-------|---------|
| [Text-to-Speech](capabilities/text-to-speech.yaml) | Amazon Polly API | 7 | Application Developer, Content Creator |

## Vocabulary

- [Amazon Polly Vocabulary](vocabulary/amazon-polly-vocabulary.yaml) — Unified taxonomy mapping resources, actions, workflows, and personas across operational (OpenAPI) and capability (Naftiko) dimensions

## Rules

- [Amazon Polly Spectral Rules](rules/amazon-polly-spectral-rules.yml) — 22 rules across 10 categories enforcing Amazon Polly API conventions

## Maintainers

**FN:** Kin Lane

**Email:** kin@apievangelist.com
