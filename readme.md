**@Author：Runsen**

1876年，亚历山大·格雷厄姆·贝尔（Alexander Graham Bell）发明了一种电报机，可以通过电线传输音频。托马斯·爱迪生（Thomas Edison）于1877年发明了留声机，这是第一台记录声音并播放声音的机器。


最早的语音识别软件之一是由Bells Labs在1952年编写的，只能识别数字。1985年，IBM发布了使用“隐马尔可夫模型”的软件，该软件可识别1000多个单词。

几年前，一个`replace("?","")`代码价值一个亿

如今，在Python中Tensorflow，Keras，Librosa，Kaldi和语音转文本API等多种工具使语音计算变得更加容易。


今天，我使用gtts和speech_recognition，教大家如何通过三十行代码，打造一款简单的人工语音对话。思路就是将语音变成文本，然后文本变成语音。



# gtts
gtts是将文字转化为语音，但是需要在VPN下使用。这个因为要接谷歌服务器。 


具体[gtts的官方文档](https://gtts.readthedocs.io/en/latest/module.html)：


下面，让我们看一段简单的的代码


```python
from gtts import gTTS

def speak(audioString):
    print(audioString)
    tts = gTTS(text=audioString, lang='en')
    tts.save("audio.mp3")
    os.system("audio.mp3")
    
speak("Hi Runsen, what can I do for you?")
```


执行上面的代码，就可以生成一个mp3文件，播放就可以听到了`Hi Runsen, what can I do for you?`。这个MP3会自动弹出来的。



# speech_recognition
speech_recognition用于执行语音识别的库，支持在线和离线的多个引擎和API。

speech_recognition具体[官方文档](https://pypi.org/project/SpeechRecognition/)


安装speech_recognition可以会出现错误，对此解决的方法是通过[该网址](https://www.lfd.uci.edu/~gohlke/pythonlibs/
)安装对应的whl包




在官方文档中提供了具体的识别来自麦克风的语音输入的[代码](https://github.com/Uberi/speech_recognition/blob/master/examples/microphone_recognition.py)
![](https://img-blog.csdnimg.cn/20210519174200991.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDUxMDYxNQ==,size_16,color_FFFFFF,t_70)

下面就是 speech_recognition 用麦克风记录下你的话，这里我使用的是
recognize_google，speech_recognition 提供了很多的类似的接口。

```python
import time
import speech_recognition as sr

# 录下来你讲的话
def recordAudio():
    # 用麦克风记录下你的话
    print("开始麦克风记录下你的话")
    r = sr.Recognizer()
    with sr.Microphone() as source:
        audio = r.listen(source)
    data = ""
    try:
        data = r.recognize_google(audio)
        print("You said: " + data)
    except sr.UnknownValueError:
        print("Google Speech Recognition could not understand audio")
    except sr.RequestError as e:
        print("Could not request results from Google Speech Recognition service; {0}".format(e))
    return data

if __name__ == '__main__':
    time.sleep(2)
    while True:
        data = recordAudio()
        print(data)
```


下面是我乱说的英语
![](https://img-blog.csdnimg.cn/20210519175542406.png)

# 对话


上面，我们实现了用麦克风记录下你的话，并且得到了对应的文本，那么下一步就是字符串的文本操作了，比如说`how are you`，那回答`"I am fine”`，然后将`"I am fine”`通过gtts是将文字转化为语音


```clike
# @Author：Runsen
# -*- coding: UTF-8 -*-
import speech_recognition as sr
from time import ctime
import time
import os
from gtts import gTTS


# 讲出来AI的话
def speak(audioString):
    print(audioString)
    tts = gTTS(text=audioString, lang='en')
    tts.save("audio.mp3")
    os.system("audio.mp3")


# 录下来你讲的话
def recordAudio():
    # 用麦克风记录下你的话
    r = sr.Recognizer()
    with sr.Microphone() as source:
        audio = r.listen(source)

    data = ""
    try:
        data = r.recognize_google(audio)
        print("You said: " + data)
    except sr.UnknownValueError:
        print("Google Speech Recognition could not understand audio")
    except sr.RequestError as e:
        print("Could not request results from Google Speech Recognition service; {0}".format(e))

    return data


# 自带的对话技能（逻辑代码：rules）
def jarvis():
    while True:
        data = recordAudio()
        print(data)
        if "how are you" in data:
            speak("I am fine")
        if "time" in data:
            speak(ctime())
        if "where is" in data:
            data = data.split(" ")
            location = data[2]
            speak("Hold on Runsen, I will show you where " + location + " is.")
            # 打开谷歌地址
            os.system("open -a Safari https://www.google.com/maps/place/" + location + "/&amp;")

        if "bye" in data:
            speak("bye bye")
            break


if __name__ == '__main__':
    # 初始化
    time.sleep(2)
    speak("Hi Runsen, what can I do for you?")

    # 跑起
    jarvis()
```
![](https://img-blog.csdnimg.cn/20210519175812616.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDUxMDYxNQ==,size_16,color_FFFFFF,t_70)

当我说how are you？会弹出I am fine的mp3
![](https://img-blog.csdnimg.cn/20210519180016827.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDUxMDYxNQ==,size_16,color_FFFFFF,t_70)

当我说where is Chiana？会弹出Hold on Runsen, I will show you where China is.的MP3



![](https://img-blog.csdnimg.cn/20210519180204358.png)
同样也会弹出China的谷歌地图
![](https://img-blog.csdnimg.cn/20210519180123831.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDUxMDYxNQ==,size_16,color_FFFFFF,t_70)
本项目对应的[Github](https://github.com/MaoliRUNsen/Simple-intelligent-voice-dialogue)


 
 
