Tested in Termux.  
Library depends: w4py.  
Python version: 3.  

### Install ws4py
```shell
pip3 install --user ws4py
```

How to call Speech API without edge:

```python
#!/usr/bin/env python3
# -*- coding:utf-8 -*-

from ws4py.client.threadedclient import WebSocketClient
import binascii

class WSClient(WebSocketClient):
    def __init__(self, url, text, filename):
        self.fp = open(filename, 'wb')
        self.text = text
        super(WSClient, self).__init__(url)

    def opened(self):
        self.send('Content-Type:application/json; charset=utf-8\r\n\r\nPath:speech.config\r\n\r\n{"context":{"synthesis":{"audio":{"metadataoptions":{"sentenceBoundaryEnabled":"false","wordBoundaryEnabled":"true"},"outputFormat":"webm-64khz-32bit-mono-opus"}}}}\r\n')
        self.send("X-RequestId:fe83fbefb15c7739fe674d9f3e81d38f\r\nContent-Type:application/ssml+xml\r\nPath:ssml\r\n\r\n<speak version='1.0' xmlns='http://www.w3.org/2001/10/synthesis' xml:lang='en-US'><voice  name='Microsoft Server Speech Text to Speech Voice (zh-TW, HsiaoChenNeural)'><prosody pitch='+0Hz' rate ='+0%' volume='+0%'>"+self.text+"</prosody></voice></speak>\r\n")

    def received_message(self, m):
        if b'turn.end' in m.data:
            self.close()
            self.fp.close()
        elif b'Path:audio\r\n' in m.data:
            song_bytes = m.data.split(b'Path:audio\r\n')[1]
            self.fp.write(song_bytes)

        else:
            # print(m)
            pass


if __name__ == '__main__':
    url = 'wss://speech.platform.bing.com/consumer/speech/synthesize/readaloud/edge/v1?TrustedClientToken=6A5AA1D4EAFF4E9FB37E23D68491D6F4'
    text = '我有人间伤心事要忙'
    filename = './test.wav'
    ws = WSClient(url, text, filename)
    ws.connect()
    ws.run_forever()
```


output:

```shell
fuckme@localhost ~> file test.wav
test.wav: MPEG ADTS, layer III, v2,  32 kbps, 16 kHz, Monaural
```
