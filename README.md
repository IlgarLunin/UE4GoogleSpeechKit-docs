# **UE4 Google Speech Kit**

![](pics/Icon128.png)

This is UE4 wrapper for Google's [Cloud Text-to-Speech](https://cloud.google.com/text-to-speech/) and syncronous [Cloud Speech-to-Text](https://cloud.google.com/speech-to-text/) speech recognition.

Plugin was battle tested in several commercial simulator projects. It is small, lean and simple to use.

# Preparation
1) Go to [google cloud](https://console.cloud.google.com) and create payment account.
2) Enable [Cloud Speech-to-Text API](https://console.cloud.google.com/apis/library/speech.googleapis.com) and [Cloud Text-to-Speech API](https://console.cloud.google.com/apis/library/texttospeech.googleapis.com).
3) Create credentials to access your enabled APIs. See instructions [here](https://cloud.google.com/docs/authentication).

![](pics/api_key.png)

4) Create environment variable `GOOGLE_API_KEY` with created key as value.

![](pics/googlespeechkeyenv.png)

Find out more in documentation for corresponding sections.
* [Supported voices](https://cloud.google.com/text-to-speech/docs/voices) ([WaveNet](https://en.wikipedia.org/wiki/WaveNet) are the best)

# Speech synthesis

You need to supply text to async node, as well as voice variant. As output you will get
sound wave object which can be played by engine.

![](pics/googletts.png)

# Speech recognition

Consists of two parts. First, we need to record voice from microphone. To do that, use **UMicrophoneCapture**
actor component as shown below. Next, construct recognition parameters and pass them to
**Google STT** async node.

![](pics/googlestt.png)

Another way to perform recognition is to use **Google STT Variants** node. Which, instead of returning result with highest confidence, returns an array of variants.

![](pics/googlesttvariants.png)

# Important steps

To make microphone work, you need to add following lines to `Engine.ini` of the project.
```
[Voice]
bEnabled=true
```

To not loose pauses in between words, you probably want to check silence detection treshold `voice.SilenceDetectionThreshold`, value `0.01` is good.
This also goes to `Engine.ini`.

```
[SystemSettings]
voice.SilenceDetectionThreshold=0.01
```
