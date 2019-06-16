# **Общее**

Почти у всех речевых сервисов есть [**TTS**](https://ru.wikipedia.org/wiki/%D0%A1%D0%B8%D0%BD%D1%82%D0%B5%D0%B7_%D1%80%D0%B5%D1%87%D0%B8) (Text to speech) и [**STT**](https://ru.wikipedia.org/wiki/%D0%A0%D0%B0%D1%81%D0%BF%D0%BE%D0%B7%D0%BD%D0%B0%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5_%D1%80%D0%B5%D1%87%D0%B8) (Speech to text).
Для каждого сервиса есть отдельная папка с реализацией.


Синтез речи. В асинхронную ноду передается текст и параметры для синтеза речи. На выходе
аудио, которое можно воспроизвести или передать в аудио компонент, поменять тон,
сохранить в файл и тд.

![](pics/googletts.png)

Распознавание речи. Состоит из двух частей.
Сначала записывается голос с микрофона специальным актор компонентом, затем передается
в асинхронную ноду, в которой происходит запрос к сервису и возврат распознанного текста.

![](pics/googlestt.png)

# **Speechace**

Немного отличается сервис [Speechace](https://www.speechace.com/). Он оценивает правильность произнесенной фразы,
и возвращает структурку с оценками по слогам, словам, и другую информацию.
**Тут для запроса нужно передавать не только аудио с микрофона, но и сам текст того что было произнесено**.

![](pics/speechacescore.png)

# **Yandex speech**

Для генерации данных аутентификации Яндекс провайдера, есть дополнительная [утилита](http://gitlab.vrtech.global/unreal/voicetoolbox/jwt-util)
для подписи jwt ключей. Аргументом принимает путь к словарю данных яндекса.

```bash
jwt_sign.exe -k full_path
```

Из анрила это вызывается так

```cpp
FString GenerateSignedJWT(bool& result)
{
    FString _outToken;
    result = false;
    // Try to generate new IAM
    FString ThirdPartyDir = FPaths::Combine(FPaths::ProjectPluginsDir(), TEXT("VoiceToolboxPlugin/ThirdParty"));
    FString JwtAbsolutePath = FPaths::Combine(ThirdPartyDir, TEXT("jwt/jwt_sign.exe"));

    if (FPaths::FileExists(JwtAbsolutePath))
    {
        void* PipeRead = nullptr;
        void* PipeWrite = nullptr;
        FPlatformProcess::CreatePipe(PipeRead, PipeWrite);

        int32 ReturnCode = -1;
        FString args = FString::Printf(TEXT("-k %s"), *speechKeyDataAbsolutePath);
        FProcHandle pHandle = FPlatformProcess::CreateProc(*JwtAbsolutePath, *args, false, true, true, nullptr, 0, nullptr, PipeWrite);
        if (pHandle.IsValid())
        {
            while (FPlatformProcess::IsProcRunning(pHandle))
            {
                _outToken += FPlatformProcess::ReadPipe(PipeRead);
                FPlatformProcess::Sleep(0.1f);
            }
            _outToken += FPlatformProcess::ReadPipe(PipeRead);
            FPlatformProcess::GetProcReturnCode(pHandle, &ReturnCode);

            _outToken = _outToken.Replace(TEXT("\r"), TEXT(""));
            _outToken = _outToken.Replace(TEXT("\n"), TEXT(""));

            result = ReturnCode == 0;
        }
        else
        {
            UE_LOG(LogYaSpeech, Error, TEXT("Failed to launch jwt_sign.exe"));
        }
        FPlatformProcess::ClosePipe(PipeRead, PipeWrite);
    }
    return _outToken;
}
```

Чтобы сгенерировать этот словарь с данными аутентификации, используется [вспомогательная утилита](http://gitlab.vrtech.global/unreal/voicetoolbox/yandex-cloud-api-tools),
это делается один раз. [Пошаговая инструкция](https://docs.google.com/document/d/1dt0TGEDiWFjwve_iqH-jgf-WOMBdu-lYOleoKG6Rl-c/edit?usp=sharing)

Для яндекс провайдера можно крутить настройки таймаута в `DefaultEngine.ini` или `Engine.ini`

```
[HTTP]
HttpTimeout=30
HttpConnectionTimeout=30
HttpReceiveTimeout=30
HttpSendTimeout=30
HttpMaxConnectionsPerServer=16
bEnableHttp=true
bUseNullHttp=false
HttpDelayTime=0
```

# **Google speech**

Для доступа к апи используется ключ доступа. Он указывается через переменные среды

![](pics/googlespeechkeyenv.png)

Ключ доступа создается через консоль на [google cloud](https://cloud.google.com). Так же вся документация по апи,
варианты голосов и другая информация.
