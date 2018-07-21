# keras-complete-rest-api

https://www.pyimagesearch.com/2018/02/05/deep-learning-production-keras-redis-flask-apache/

여기서 소스 다운로드

# python3.6.6 64bit

## virtualenv windows

```
virtualenv keras_flask

PS D:\pibex> .\keras_flask\Scripts\activate
(keras_flask) PS D:\pibex> deactivate
```

## tensorflow 1.5.0 + CUDA 9.0 + cudnn 7.0.5

pip install tensorflow==1.5.0  (1.5.0인 이유는 딱히 없음)

또는 pip install tensorflow-gpu==1.5.0 (**둘 중 하나만 설치하자**, venv 따로 만들기)

**python과 tensorflow 와 cuda와 cudnn 버전을 다 맞춰줘야 한다**



cudart64_90.dll 요구 ->

https://developer.nvidia.com/cuda-90-download-archive  에서 CUDA 9.0 설치

위치 : C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v9.0\bin



cudnn64_7.dll 요구 ->

https://developer.nvidia.com/rdp/cudnn-archive 에서 cuDNN7.0.5 for cuda 9.0 다운 (library for windows)

http://solarisailab.com/archives/1581 처럼 CUDA에다 복사

# apache 2.4 (VC 14)

## no xampp just apache

https://www.apachelounge.com/download/VC14/ 다운받고 C:\Apache24 로 옮기자

C:\Apache24\bin 에 있는 httpd.exe로 서버 실행

``` 
httpd.exe -k install (서비스로 등록, 처음이면 실행)
httpd.exe -k uninstall
httpd.exe -k start
httpd.exe -k stop
httpd.exe -k restart
```



## wsgi - precompiled

```
pip install mod_wsgi (env : MOD_WSGI_APACHE_ROOTDIR C:/xampp/apache 로 설정, 근데 컴파일이 잘 안 된다)

대신 precompiled binary 사용
pip install .\mod_wsgi-4.6.4+ap24vc14-cp36-cp36m-win_amd64.whl
mod_wsgi-express module-config 에서 나온 설정을 C:\Apache24\conf\httpd.conf에 추가
```

참고한 사이트

- https://stackoverflow.com/questions/46363836/how-to-install-mod-wsgi-on-windowsxampp-in-2017
- https://stackoverflow.com/questions/42298503/how-to-install-mod-wsgi-for-apache-2-4-and-python-3-4-on-windows/42307082#42307082
- http://www.flask.moe/windows
- https://github.com/GrahamDumpleton/mod_wsgi/blob/develop/win32/README.rst



Python 3.6 == VC14 of MS compiler

- https://www.apachelounge.com/download/VC14/
- https://wiki.python.org/moin/WindowsCompilers
- https://www.lfd.uci.edu/~gohlke/pythonlibs/#mod_wsgi  
  - mod_wsgi‑4.6.4+ap24vc14‑cp36‑cp36m‑win_amd64.whl

**파이썬 버전의 VC 버전과 apache의 VC 버전과 mod_wsgi 버전이 맞아야 한다**



#### WSGIDaemonProcess, WSGIProcessGroup  

windows에서는 지원 안 함, 고로 세팅에서 제외

http://modwsgi.readthedocs.io/en/develop/configuration-directives/WSGIDaemonProcess.html

http://modwsgi.readthedocs.io/en/develop/configuration-directives/WSGIProcessGroup.html

## virtual host

virtual host 세팅을 해도 되고, 안 해도 딱히 상관없음 (httpd.conf에 있는 VirtualHost *:5000 태그는 localhost:5000을 따로 연결해주는 것 )

참고 사이트

http://zzaps.tistory.com/245



# redis

https://github.com/MicrosoftArchive/redis/releases

msi로 설치, 서비스에 등록되어 자동으로 컴터 키면 redis-server 실행된 상태가 된다 (%path% 자동추가)

끄려면 redis-cli 후 shutdown

# Flask Test

```python
##__init__.py
import flask
app = flask.Flask(__name__)
@app.route("/")

def hello_world():
   return "Hello World!"
```

```
##httpd.conf

LoadFile "c:/python36/python36.dll"
LoadModule wsgi_module "d:/pibex/keras_flask/lib/site-packages/mod_wsgi/server/mod_wsgi.cp36-win_amd64.pyd"
WSGIPythonHome "d:/pibex/keras_flask"

Listen 5000
<VirtualHost *:5000>
WSGIScriptAlias / D:\pibex\test\hello_world.wsgi
<Directory "D:/pibex/test">
 WSGIApplicationGroup %{GLOBAL}
 Order deny,allow
 Allow from all
 Require all granted
</Directory>
</VirtualHost>
```

```
##hello_world.wsgi
import sys
sys.path.insert(0, 'D:/pibex')

from test import app as application
```

localhost:5000에 hello world 뜨면 apache-wsgi 세팅 성공 확인

테스팅할때 내 디렉토리 구조는 D: / pibex / test / \_\_init\_\_.py, hello_world.wsgi

# real game

### setting

redis-server와 apache는 서비스로 설치되어 있기 때문에(설치 방법 따라 다르겠지만) 윈도우를 켜면 처음에 같이 켜짐

```
redis ON : redis-server
redis OFF : redis-cli -> shutdown
apache ON :  httpd.exe -k start
apache OFF : httpd.exe -k stop
```

redis-server, apache, run_model_server.py 3개 순서대로 키면 준비완료




설정파일

```
##httpd.conf
Listen 5522
<VirtualHost *:5522>
    WSGIScriptAlias / D:\pibex\keras-complete-rest-api\keras_rest_api_app.wsgi
    <Directory "D:/pibex/keras-complete-rest-api">
        WSGIApplicationGroup %{GLOBAL}
        Order deny,allow
        Allow from all
        Require all granted
    </Directory>
</VirtualHost>
```

```
##keras_rest_api_app.wsgi
# add our app to the system path
import sys
sys.path.insert(0, "D:/pibex/keras-complete-rest-api")

# import the application and away we go...
from run_web_server import app as application
```

테스팅할때 내 디렉토리 구조는 D: / pibex / keras-complete-rest-api

### one image test

```
curl -X POST -F image=@jemma.png 'http://imnotkind.ml:5522/predict'
```

curl -X나 -F는 윈도우에 없어서 리눅스로 실험, 잘 되는 것 확인

### stress test(cpu)

``` 
KERAS_REST_API_URL = "http://localhost:5522/predict" 
```

url 바꿔주기 (내가 지정한 곳으로)

cpu로 하니 avg batch size 32, 느림

### stress test(gpu)

run_model_server.py실행하면 gpu 이름들이 나와야 함

avg batch size 6~7,  매우 빠름
