---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 03/11/2020
ms.author: trbye
ms.openlocfilehash: e9560d9af095193451fd4276d830a06d03f2cc6c
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93135983"
---
Um dos principais recursos do serviço de Fala é a capacidade de reconhecer e transcrever a fala humana (frequentemente denominada conversão de fala em texto). Neste guia de início rápido, você aprende a usar o SDK de Fala em seus aplicativos e produtos para executar uma conversão de fala em texto de alta qualidade.

## <a name="skip-to-samples-on-github"></a>Pular para os exemplos no GitHub

Se você quiser pular diretamente para o código de exemplo, confira os [exemplos do guia de início rápido do Python](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/python) no GitHub.

## <a name="prerequisites"></a>Pré-requisitos

Este artigo pressupõe que:

* Você tem uma conta do Azure e uma assinatura do serviço de Fala. Se você não tiver uma conta e uma assinatura – [Experimente o serviço de Fala gratuitamente](../../../overview.md#try-the-speech-service-for-free).

## <a name="install-and-import-the-speech-sdk"></a>Instalar e importar o SDK de Fala

Antes de fazer qualquer coisa, você precisará instalar o SDK de Fala.

```Python
pip install azure-cognitiveservices-speech
```

Se você estiver no macOS e encontrar problemas de instalação, talvez seja necessário executar esse comando primeiro.

```Python
python3 -m pip install --upgrade pip
```

Depois que o SDK de Fala for instalado, importe-o para seu projeto Python.

```Python
import azure.cognitiveservices.speech as speechsdk
```

## <a name="create-a-speech-configuration"></a>Criar uma configuração de fala

Para chamar o serviço de Fala usando o SDK de Fala, você precisa criar um [`SpeechConfig`](https://docs.microsoft.com/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechconfig?view=azure-python). Essa classe inclui informações sobre sua assinatura, como sua chave e região, ponto de extremidade, host ou token de autorização associados. Crie um [`SpeechConfig`](https://docs.microsoft.com/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechconfig?view=azure-python) usando sua chave e região. Confira a página [suporte a regiões](https://docs.microsoft.com/azure/cognitive-services/speech-service/regions#speech-sdk) para encontrar o identificador de sua região.

```Python
speech_key, service_region = "<paste-your-subscription-key>", "<paste-your-region>"
speech_config = speechsdk.SpeechConfig(subscription=speech_key, region=service_region)
```

Há algumas outras maneiras de inicializar um [`SpeechConfig`](https://docs.microsoft.com/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechconfig?view=azure-python):

* Com um ponto de extremidade: passe um ponto de extremidade do serviço de Fala. Uma chave ou um token de autorização é opcional.
* Com um host: passe um endereço de host. Uma chave ou um token de autorização é opcional.
* Com um token de autorização: passe um token de autorização e a região associada.

> [!NOTE]
> Independentemente se você estiver executando o reconhecimento de fala, a síntese de fala, tradução ou reconhecimento de intenção, você sempre criará uma configuração.

## <a name="recognize-from-microphone"></a>Reconhecer do microfone

Para reconhecer fala usando o microfone do dispositivo, basta criar um `SpeechRecognizer` sem passar `AudioConfig` e passar `speech_config`.

```Python
speech_recognizer = speechsdk.SpeechRecognizer(speech_config=speech_config)

print("Speak into your microphone.")
result = speech_recognizer.recognize_once_async().get()
print(result.text)
```

Se você quiser usar um dispositivo de entrada de áudio *específico* , precisará especificar a identificação do dispositivo no `AudioConfig`, além de passá-la para o parâmetro `audio_config` do construtor `SpeechRecognizer`. Saiba [como obter a identificação do dispositivo](../../../how-to-select-audio-input-devices.md) de entrada de áudio.

## <a name="recognize-from-file"></a>Reconhecer do arquivo

Se quiser reconhecer fala de um arquivo de áudio em vez de usar um microfone, crie um [`AudioConfig`](https://docs.microsoft.com/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.audio.audioconfig?view=azure-python&preserve-view=true) e use o parâmetro `filename`.

```Python
audio_input = speechsdk.AudioConfig(filename="your_file_name.wav")
speech_recognizer = speechsdk.SpeechRecognizer(speech_config=speech_config, audio_config=audio_input)

result = speech_recognizer.recognize_once_async().get()
print(result.text)
```

## <a name="recognize-speech"></a>Reconhecer fala

A [classe Reconhecedor](https://docs.microsoft.com/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.recognizer?view=azure-python&preserve-view=true) do SDK de Fala do Python expõe alguns métodos que você pode usar para o reconhecimento de fala.

### <a name="single-shot-recognition"></a>Reconhecimento pontual

O reconhecimento pontual reconhece de forma assíncrona um único enunciado. O fim de um único enunciado é determinado pela escuta de silêncio no fim ou até o máximo de 15 segundos de áudio processado. Veja um exemplo de reconhecimento pontual assíncrono com [`recognize_once_async()`](https://docs.microsoft.com/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.recognizer?view=azure-python#recognize-once-async------azure-cognitiveservices-speech-resultfuture):

```Python
result = speech_recognizer.recognize_once_async().get()
```

Será necessário escrever código para iterar o resultado. Este exemplo avalia o [`result.reason`](https://docs.microsoft.com/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.resultreason?view=azure-python):

* Imprime o resultado de reconhecimento: `speechsdk.ResultReason.RecognizedSpeech`
* Se não houver correspondência com o reconhecimento, informe o usuário: `speechsdk.ResultReason.NoMatch `
* Se encontrar um erro, imprima a mensagem de erro: `speechsdk.ResultReason.Canceled`

```Python
if result.reason == speechsdk.ResultReason.RecognizedSpeech:
    print("Recognized: {}".format(result.text))
elif result.reason == speechsdk.ResultReason.NoMatch:
    print("No speech could be recognized: {}".format(result.no_match_details))
elif result.reason == speechsdk.ResultReason.Canceled:
    cancellation_details = result.cancellation_details
    print("Speech Recognition canceled: {}".format(cancellation_details.reason))
    if cancellation_details.reason == speechsdk.CancellationReason.Error:
        print("Error details: {}".format(cancellation_details.error_details))
```

### <a name="continuous-recognition"></a>Reconhecimento contínuo

O reconhecimento contínuo é um pouco mais complexo que o pontual. Ele requer que você se conecte ao `EventSignal` para receber os resultados de reconhecimento, e para interromper o reconhecimento, você precisa chamar [stop_continuous_recognition()](https://docs.microsoft.com/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.recognizer?view=azure-python#stop-continuous-recognition--) ou [stop_continuous_recognition()](https://docs.microsoft.com/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.recognizer?view=azure-python#stop-continuous-recognition-async--). Veja um exemplo de como o reconhecimento contínuo é realizado em um arquivo de entrada de áudio.

Comece definindo a entrada e inicializando um [`SpeechRecognizer`](https://docs.microsoft.com/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechrecognizer?view=azure-python&preserve-view=true):

```Python
audio_config = speechsdk.audio.AudioConfig(filename=weatherfilename)
speech_recognizer = speechsdk.SpeechRecognizer(speech_config=speech_config, audio_config=audio_config)
```

Em seguida, crie uma variável para gerenciar o estado do reconhecimento de fala. Para começar, defina-a como `False`, porque, no início do reconhecimento, podemos pressupor com segurança que ela ainda não está concluída.

```Python
done = False
```

Agora criaremos um retorno de chamada para interromper o reconhecimento contínuo quando um `evt` for recebido. Há algumas coisas que você deve considerar.

* Quando um `evt` é recebido, a mensagem `evt` é impressa.
* Depois que `evt` for recebido, [stop_continuous_recognition()](https://docs.microsoft.com/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.recognizer?view=azure-python#stop-continuous-recognition--) será chamado para interromper o reconhecimento.
* O estado de reconhecimento é alterado para `True`.

```Python
def stop_cb(evt):
    print('CLOSING on {}'.format(evt))
    speech_recognizer.stop_continuous_recognition()
    nonlocal done
    done = True
```

Este exemplo de código mostra como conectar retornos de chamada a eventos enviados do [`SpeechRecognizer`](https://docs.microsoft.com/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.recognizer?view=azure-python#start-continuous-recognition--).

* [`recognizing`](https://docs.microsoft.com/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.recognizer?view=azure-python#recognizing): sinal para os eventos que contêm resultados de reconhecimento intermediários.
* [`recognized`](https://docs.microsoft.com/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.recognizer?view=azure-python#recognized): sinal para eventos que contêm resultados de reconhecimento finais (indicando uma tentativa de reconhecimento bem-sucedida).
* [`session_started`](https://docs.microsoft.com/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.recognizer?view=azure-python#session-started): sinal para eventos que indicam o início de uma sessão de reconhecimento (operação).
* [`session_stopped`](https://docs.microsoft.com/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.recognizer?view=azure-python#session-stopped): sinal para eventos que indicam o fim de uma sessão de reconhecimento (operação).
* [`canceled`](https://docs.microsoft.com/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.recognizer?view=azure-python#canceled): sinal para eventos que contêm resultados de reconhecimento cancelados (indicando uma tentativa de reconhecimento que foi cancelada em decorrência de uma solicitação de cancelamento direta ou uma falha de transporte ou protocolo).

```Python
speech_recognizer.recognizing.connect(lambda evt: print('RECOGNIZING: {}'.format(evt)))
speech_recognizer.recognized.connect(lambda evt: print('RECOGNIZED: {}'.format(evt)))
speech_recognizer.session_started.connect(lambda evt: print('SESSION STARTED: {}'.format(evt)))
speech_recognizer.session_stopped.connect(lambda evt: print('SESSION STOPPED {}'.format(evt)))
speech_recognizer.canceled.connect(lambda evt: print('CANCELED {}'.format(evt)))

speech_recognizer.session_stopped.connect(stop_cb)
speech_recognizer.canceled.connect(stop_cb)
```

Com tudo configurado, podemos chamar [start_continuous_recognition()](https://docs.microsoft.com/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.recognizer?view=azure-python#session-stopped).

```Python
speech_recognizer.start_continuous_recognition()
while not done:
    time.sleep(.5)
```

### <a name="dictation-mode"></a>Modo de Ditado

Ao usar o reconhecimento contínuo, você pode habilitar o processamento de ditado usando a função "habilitar ditado" correspondente. Esse modo fará a instância de configuração de fala interpretar as descrições das estruturas de frase, como pontuação. Por exemplo, o enunciado "Você mora na cidade ponto de interrogação" seria interpretado como o texto "Você mora na cidade?".

Para habilitar o modo de ditado, use o método [`enable_dictation()`](https://docs.microsoft.com/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechconfig?view=azure-python#enable-dictation--) no [`SpeechConfig`](https://docs.microsoft.com/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechconfig?view=azure-python&preserve-view=true).

```Python 
SpeechConfig.enable_dictation()
```

## <a name="change-source-language"></a>Alterar idioma de origem

Uma tarefa comum no reconhecimento de fala é especificar o idioma da entrada (ou origem). Veja como você alteraria o idioma de entrada para alemão. No código, localize SpeechConfig e adicione esta linha logo abaixo.

```Python
speech_config.speech_recognition_language="de-DE"
```

[`speech_recognition_language`](https://docs.microsoft.com/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechconfig?view=azure-python#speech-recognition-language) é um parâmetro que usa uma cadeia de caracteres como argumento. Você pode fornecer qualquer valor na lista de [localidades/idiomas](../../../language-support.md) compatíveis.

## <a name="improve-recognition-accuracy"></a>Aprimorar a precisão do reconhecimento

Existem algumas maneiras de aprimorar a precisão do reconhecimento com o SDK de Fala. Vamos examinar Listas de Frases. As Listas de Frases são usadas para identificar frases conhecidas nos dados do áudio, como o nome de uma pessoa ou um local específico. Palavras ou frases completas podem ser adicionadas a uma Lista de Frases. Durante o reconhecimento, uma entrada em uma lista de frases será usada se uma correspondência exata à frase inteira estiver incluída no áudio. Se uma correspondência exata à frase não for encontrada, o reconhecimento não será assistido.

> [!IMPORTANT]
> O recurso Lista de Frases só está disponível em inglês.

Para usar uma lista de frases, primeiro crie um objeto [`PhraseListGrammar`](https://docs.microsoft.com/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.phraselistgrammar?view=azure-python&preserve-view=true). Em seguida, adicione palavras e frases específicas com [`addPhrase`](https://docs.microsoft.com/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.phraselistgrammar?view=azure-python#addphrase-phrase--str-).

Todas as alterações em [`PhraseListGrammar`](https://docs.microsoft.com/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.phraselistgrammar?view=azure-python&preserve-view=true) entrarão em vigor no próximo reconhecimento ou após uma reconexão com o serviço de Fala.

```Python
phrase_list_grammar = speechsdk.PhraseListGrammar.from_recognizer(reco)
phrase_list_grammar.addPhrase("Supercalifragilisticexpialidocious")
```

Se precisar limpar a lista de frases: 

```Python
phrase_list_grammar.clear()
```

### <a name="other-options-to-improve-recognition-accuracy"></a>Outras opções para aprimorar a precisão do reconhecimento

As listas de frases são só uma opção para aprimorar a precisão do reconhecimento. Também é possível: 

* [Aprimorar a precisão com a Fala Personalizada](../../../how-to-custom-speech.md)
* [Aprimorar a precisão com modelos de locatário](../../../tutorial-tenant-model.md)
