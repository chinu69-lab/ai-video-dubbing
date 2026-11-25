# 🎙️ Kaggle AI Dubbing Studio

> Transcribe, correct, translate, and dub audio or video into a new language — with speaker-aware voices — all inside a single Kaggle notebook.

This guide covers setup and use of [`kaggle_dubbing_gradio.ipynb`](./kaggle_dubbing_gradio.ipynb), a Gradio app for transcribing media, correcting its transcript with AI, translating it, and generating a speaker-aware dub.

---

## Table of Contents

- [Demo](#demo)
- [What the app does](#what-the-app-does)
- [Language support](#language-support)
- [Requirements](#requirements)
- [Setup](#setup)
- [Using the app](#using-the-app)
- [Voice references](#voice-references)
- [Optional controls](#optional-controls)
- [Processing components](#processing-components)
- [Troubleshooting](#troubleshooting)

---

## Demo

See the result before you try it. Both clips below are hosted on Google Drive.

| | Video | Link |
| --- | --- | --- |
| 🎬 **Before dubbing** | Original video with the source-language audio | [▶ Watch original video](https://drive.google.com/file/d/1LSQFiEm212ITliBhRdBVab-M1YARrX13/view) |
| 🔊 **After dubbing** | Same video with AI-generated dubbed audio | [▶ Watch dubbed video](https://drive.google.com/file/d/1W5t4VivUg2iTBq4Qtk3WfvfLnfOI7VPd/view) |  

> 💡 **Tip:** Watch the original first, then the dubbed version, to compare voice, timing, and background audio.

---

## What the app does

The notebook runs a one-page Gradio interface in Kaggle. Its workflow is:

1. **Analyze media** to separate vocals from background audio, transcribe speech, identify speakers, and prepare the transcript and voice references.
2. **Step 1 — Correct transcript with AI** to fix likely speech-recognition errors in the detected source language. Review and edit the result.
3. **Step 2 — Translate to selected language** to translate the corrected source transcript into a supported dubbing language. Review and edit the translation.
4. **Generate dub** to synthesize the translated lines, mix them with the original background audio, and prepare downloadable outputs.

The correction and translation steps are separate so you can review the source transcript before translating and check the translation before generating speech.

---

## Language support

The source-language menu contains the 100 language codes supported by the notebook's faster-whisper transcription model. It also offers **Auto-detect**.

**Source-language choices:**

> Afrikaans, Albanian, Amharic, Arabic, Armenian, Assamese, Azerbaijani, Bashkir, Basque, Belarusian, Bengali, Bosnian, Breton, Bulgarian, Burmese, Cantonese, Catalan, Chinese, Croatian, Czech, Danish, Dutch, English, Estonian, Faroese, Finnish, French, Galician, Georgian, German, Greek, Gujarati, Haitian Creole, Hausa, Hawaiian, Hebrew, Hindi, Hungarian, Icelandic, Indonesian, Italian, Japanese, Javanese, Kannada, Kazakh, Khmer, Korean, Lao, Latin, Latvian, Lingala, Lithuanian, Luxembourgish, Malagasy, Malay, Malayalam, Maltese, Maori, Marathi, Macedonian, Mongolian, Nepali, Norwegian, Nynorsk, Occitan, Pashto, Persian, Polish, Portuguese, Punjabi, Romanian, Russian, Sanskrit, Serbian, Shona, Sindhi, Sinhala, Slovak, Slovenian, Somali, Spanish, Sundanese, Swahili, Swedish, Tajik, Tagalog, Tamil, Tatar, Telugu, Thai, Tibetan, Turkish, Turkmen, Ukrainian, Urdu, Uzbek, Vietnamese, Welsh, Yiddish, Yoruba.

The dub-language menu contains the 30 languages supported for speech generation by VoxCPM2:

> Arabic, Burmese, Chinese, Danish, Dutch, English, Finnish, French, German, Greek, Hebrew, Hindi, Indonesian, Italian, Japanese, Khmer, Korean, Lao, Malay, Norwegian, Polish, Portuguese, Russian, Spanish, Swahili, Swedish, Tagalog, Thai, Turkish, and Vietnamese.

The source and target menus are intentionally different: faster-whisper recognizes more languages than VoxCPM2 can generate. For example, Cantonese is available as a recognized source language, while the target menu offers Chinese rather than a separate Cantonese voice option.

**Example language pairs:**

- Chinese → Hindi
- French → English
- English → Japanese
- Spanish → German

Groq performs translation between the selected source and target languages. Generated speech is limited to the VoxCPM2 target languages listed above.

---

## Requirements

- A Kaggle account and notebook session.
- **Internet** enabled in Kaggle notebook options.
- A GPU is recommended. The notebook can fall back to CPU for some operations, but processing may be much slower.
- Access to the Hugging Face model `pyannote/speaker-diarization-community-1`.
- A Hugging Face access token and a Groq API key, stored as Kaggle Secrets named:
  - `HF_TOKEN`
  - `GROQ_API_KEY`
- FFmpeg available in the Kaggle environment. The notebook stops with an error if it cannot find FFmpeg.
- **Input length:** you can upload a video of **up to 2 minutes** maximum.

---

## Setup

1. Open `kaggle_dubbing_gradio.ipynb` in Kaggle.
2. In **Notebook options**, enable Internet and, if available, GPU acceleration.
3. Accept the access conditions for `pyannote/speaker-diarization-community-1` on Hugging Face.
4. Add `HF_TOKEN` and `GROQ_API_KEY` under Kaggle **Add-ons → Secrets**. Grant the notebook access to both secrets.
5. Run the dependency-installation cell.
6. Restart the Kaggle session when the install cell asks you to.
7. Run all notebook cells from the beginning. The final cell launches the Gradio app.

The optional FunASR installation is used for experimental non-speech sound detection. If it is unavailable, the rest of the workflow can still run; you can add or edit keep-original sound ranges manually.

---

## Using the app

### 1. Choose media and languages

Upload one supported audio or video file:

- **Video:** MP4, MOV, MKV, WEBM, AVI, or M4V
- **Audio:** MP3, WAV, M4A, FLAC, or OGG

> ⏱️ **Maximum length: 2 minutes.** You can upload a video of up to 2 minutes.

Select the spoken language, or leave it at **Auto-detect**. Then select the dub language. You can provide optional glossary or story context to help disambiguate names, terms, and tone.

### 2. Analyze media

Click **Analyze media**. The app extracts audio, separates vocals and background audio, transcribes speech, and identifies speakers. When enabled, it also prepares a video summary, speaker profiles, scene information, and suggested non-speech ranges to keep from the original audio.

Analysis creates a transcript table and speaker voice references. Review the transcript and context before proceeding. The dialogue table is editable; do not change line IDs or timing values.

### 3. Step 1 — Correct transcript with AI

Click **1. Correct transcript with AI**. The app asks Groq to correct likely recognition errors, spelling, and punctuation while keeping the transcript in the source language. It is instructed to preserve meaning, names, numbers, negation, and tone, and not to translate or add dialogue.

Review the **Original** column and make any needed edits. If you change the keep-original sound ranges, run this step again so its transcript rows match those ranges.

### 4. Step 2 — Translate

Choose the desired dub language at the top of the app, then click **2. Translate to selected language**. Translation is enabled after Step 1 is complete. It uses the corrected source transcript and the selected target language.

Review the **Translation** column and edit it if needed. The translation is not sent to speech generation until you click **Generate dub**. If you change the sound ranges, repeat Step 1 and Step 2 before generating.

### 5. Generate and download the dub

Click **Generate dub** after reviewing the translated lines. The app generates speech with VoxCPM2, associates lines with their speakers' voice references where possible, and mixes dubbed dialogue with the original background audio.

The interface can provide:

- Full dubbed WAV audio
- A dubbed MP4 when the input is a video
- SRT subtitles
- Individual generated line audio

Download any files you need before the Kaggle session ends. Work files are stored temporarily under `/kaggle/working/ai_dubbing_studio`.

---

## Voice references

The app extracts per-speaker references from the separated vocals when enough distinct speech is available. You can also upload one shared reference clip to use instead. Clear, clean reference audio improves voice similarity; short or noisy clips may reduce quality.

> ⚠️ Use only reference audio and media that you have permission to process. Voice cloning is not identity verification, and generated speech should not be represented as proof that a real person said something.

---

## Optional controls

The **Advanced** section includes controls for:

- **Analyze summary, speakers, and scenes** — prepare extra context for transcript correction and translation.
- **Detect non-speech sounds** — suggest ranges such as laughter or other sounds to preserve from the original. Detection is approximate; review the ranges manually.
- **Automatic speaking-speed measurement** — measure pace using a sample in the selected dub language before translation. You can also use **Measure speaking speed**.
- **Keep original ranges** — edit time ranges for sounds that should remain from the source rather than be dubbed.
- **Background reduction and loudness matching** — control the mix under dubbed speech.
- **Room effect and normalization** — optional final-audio processing.
- **Timing and speed-up limits** — adjust how generated lines are fitted to their available time. Lines that do not fit are queued rather than omitted.

Speaker selection, an optional shared reference, VoxCPM2 guidance, and inference-step settings are also available. Model generation can take time, especially without a GPU.

---

## Processing components

| Component | Purpose |
| --- | --- |
| **Demucs** | Separating vocals from background audio |
| **faster-whisper** | Speech recognition and word timing |
| **pyannote** | Speaker diarization |
| **Groq** | AI-assisted transcript correction, context analysis, and multilingual translation |
| **VoxCPM2** | Multilingual speech generation and voice cloning |
| **Gradio** | User interface |

The notebook reads API credentials from Kaggle Secrets; it does not write those credentials into the project files. Media and text sent to hosted services are still subject to those services' terms and privacy practices. Do not upload sensitive material unless you are authorized to do so.

---

## Troubleshooting

| Problem | What to check |
| --- | --- |
| `HF_TOKEN is missing` | Add a Kaggle Secret named exactly `HF_TOKEN` and enable it for the notebook. |
| `GROQ_API_KEY is missing` | Add a Kaggle Secret named exactly `GROQ_API_KEY` and enable it for the notebook. |
| Hugging Face access or diarization fails | Accept the pyannote model conditions and confirm the token has access. |
| FFmpeg is not found | Enable Internet and ensure FFmpeg is installed in the Kaggle session. |
| No speech was detected | Try another source-language choice, use Auto-detect, or provide clearer audio. |
| Step 2 says to complete Step 1 | Click **1. Correct transcript with AI** first. If you changed keep-original ranges, repeat Step 1. |
| Translation or correction fails | Check the Groq secret, Internet access, and the error message. The app reports invalid or incomplete AI responses rather than silently accepting missing lines. |
| Dub sounds unlike the speaker | Try a clearer, longer reference clip, choose the correct speaker, or provide a shared reference. |
| Processing is very slow | Enable a Kaggle GPU and confirm it is available in the session. CPU fallback may be much slower. |
| Video is too long | Trim the upload to 2 minutes or less. |
