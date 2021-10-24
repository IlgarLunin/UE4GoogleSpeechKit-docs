# **UE4 Google Speech Kit**

![](pics/Icon128.png)

This is UE4 wrapper for Google's [Cloud Text-to-Speech](https://cloud.google.com/text-to-speech/) and syncronous [Cloud Speech-to-Text](https://cloud.google.com/speech-to-text/) speech recognition.

Plugin was battle tested in several commercial simulator projects. It is small, lean and simple to use.

# Preparation
1) Go to [google cloud](https://console.cloud.google.com) and create payment account.
2) Enable [Cloud Speech-to-Text API](https://console.cloud.google.com/apis/library/speech.googleapis.com) and [Cloud Text-to-Speech API](https://console.cloud.google.com/apis/library/texttospeech.googleapis.com).
3) Create credentials to access your enabled APIs. See instructions [here](https://cloud.google.com/docs/authentication).

![](pics/api_key.png)

4) There are two ways how you can use your credentials in project.

    * 4.1 By using environment variables. Create environment variable `GOOGLE_API_KEY` with created key as value.

    * 4.2 By assigning key directly in blueprints. This can be called anywhere.

    ![](pics/apikeybp.png)

    By default you need to set api key from nodes. To use environment variable, you need to set `Use Env Variable` to `true`.

> **ADVICE**: Pay attention to security and encrypt your assets before packaging.

![](pics/encryption.png)

# Speech synthesis

You need to supply text to async node, as well as voice variant, speech speed, pitch value and optionally audio effects. As output you will get
sound wave object which can be played by engine.

![](pics/googletts.png)

## Bonus!

Output raw samles can be used with oculus ovr lipsync in runtime.

![](pics/ovrframesequence.png)

Get node [here](https://github.com/IlgarLunin/UE4OVRLipSyncCookFrameSequence).

Demo:

[![IMAGE ALT TEXT HERE](https://img.youtube.com/vi/B78aQly2wrI/0.jpg)](https://www.youtube.com/watch?v=B78aQly2wrI)

# Speech recognition

Consists of two parts. First, we need to record voice from microphone. To do that, use provided **MicrophoneCapture**
actor component as shown below. Next, construct recognition parameters and pass them to **Google STT** async node.

![](pics/googlestt.png)

Another way to perform recognition is to use **Google STT Variants** node. Which, instead of returning result with highest confidence, returns an array of variants.

![](pics/googlesttvariants.png)

Probably you will need to send voice commands to you app, to increase recognition chances use `CompareStrings` node. Below call will return 0.666 value,
so we can treat those strings equal since they are simmilar on 66%.

![](pics/compare.png)

# Important steps

To make microphone work, you need to add following lines to `DefaultEngine.ini` of the project.
```
[Voice]
bEnabled=true
```

To not loose pauses in between words, you probably want to check silence detection treshold `voice.SilenceDetectionThreshold`, value `0.01` is good.
This also goes to `DefaultEngine.ini`.

```
[SystemSettings]
voice.SilenceDetectionThreshold=0.01
```
Starting from Engine version 4.25 also put
```
voice.MicNoiseGateThreshold=0.01
```

Another voice related variables worth playing with
```bash
voice.MicNoiseGateThreshold
voice.MicInputGain
voice.MicStereoBias
voice.MicNoiseAttackTime
voice.MicNoiseReleaseTime
voice.MicStereoBias
voice.SilenceDetectionAttackTime
voice.SilenceDetectionReleaseTime
```

To find available settings type `voice.` in editor console, and autocompletion widget will pop up.

![](pics/voicesettings.png)

Console variables can be modified in runtime like this

![](pics/silencenode.png)

To debug your microphone input you can convert output sound buffer to
unreal sound wave and play it.

![](pics/buffertosound.png)

Above values may differ depending on actual microphone characteristics.

# Platforms supported

**Windows** and **Mac**.

# Migration guide
<details>
<summary>Version 3.0</summary>

`EGoogleTTSLanguage` was removed. You need to pass [voice name](https://cloud.google.com/text-to-speech/docs/voices) as string (**Voice name** column).

![new_language_pin](pics/new_language_pin.png)

> **WARNING**: Since synthesys parameters has changed, TTS cache is no longer valid! Make sure you remove TTS cache if exists. **Editor/Game can freeze** if old cache wll be loaded. So make sure to remove `PROJECT_ROOT/Saved/GoogleTTSCache` folder. Or invoke `WipeTTSCache` node before GoogleTTS node is executed!

![](pics/wipe_cache.png)

![](pics/tts_cache_folder.png)

The reason for this is that the number of languages has exceeded 256, and we can't put this amount into 8 bit enums (This is Unreal's limitation)



</details>

# Links
Find out more in documentation for corresponding sections.
* [Supported TTS voices](https://cloud.google.com/text-to-speech/docs/voices) ([WaveNet](https://en.wikipedia.org/wiki/WaveNet) are the best)
* [Speech synthesis config](https://cloud.google.com/text-to-speech/docs/reference/rest/v1/text/synthesize#audioconfig)
* [Supported STT languages](https://cloud.google.com/speech-to-text/docs/languages)
* [Speech recognition config](https://cloud.google.com/speech-to-text/docs/reference/rest/v1/RecognitionConfig)
