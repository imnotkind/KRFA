# python3 64bit

## virtualenv windows

```
virtualenv keras_flask

PS D:\pibex> .\keras_flask\Scripts\activate
(keras_flask) PS D:\pibex> deactivate
```

## tensorflow 1.5.0 + CUDA 9.0 + cudnn 7.0.5

pip install tensorflow==1.5.0  (1.5.0인 이유는 딱히 없음)

또는 pip install tensorflow-gpu==1.5.0 (둘 중 하나만, venv 따로 만들기)

need cuda 9.0!! NOT 9.2

```
ImportError: Could not find 'cudart64_90.dll'. TensorFlow requires that this DLL be installed in a directory that is named in your %PATH% environment variable. Download and install CUDA 9.0 from this URL: https://developer.nvidia.com/cuda-toolkit
```

 https://developer.nvidia.com/cuda-downloads?target_os=Windows&target_arch=x86_64&target_version=10&target_type=exelocal : 9.2(최신)

https://developer.nvidia.com/cuda-90-download-archive : 9.0

저 cudart64_90.dll 찾아주는 방법을 모르겠음!!! path추가해도 안 된다..  -> tensorflow나 tensorflow-gpu 둘 중 하나만 설치하니까 그냥 되는듯 -> 나중에 보니 path 굳이 추가 안 해도 되는듯?

```
C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v9.0\bin
```

cudnn64_7.dll 요구 ->

https://developer.nvidia.com/rdp/cudnn-archive 에서 cuDNN7.0.5 for cuda 9.0 다운 (library for windows)

http://solarisailab.com/archives/1581 처럼 CUDA에다 복사

# apache 2.4 (VC 14)

## no xampp just apache

https://github.com/GrahamDumpleton/mod_wsgi/issues/204

``` 
httpd -k install
httpd -k uninstall
httpd -k start
httpd -k stop
httpd -k restart
```

https://www.apachelounge.com/download/VC14/

## wsgi - precompiled

```
pip install mod_wsgi (not working, env : MOD_WSGI_APACHE_ROOTDIR C:/xampp/apache)
pip install .\mod_wsgi-4.6.4+ap24vc14-cp36-cp36m-win_amd64.whl
mod_wsgi-express module-config
```

https://stackoverflow.com/questions/46363836/how-to-install-mod-wsgi-on-windowsxampp-in-2017

https://stackoverflow.com/questions/42298503/how-to-install-mod-wsgi-for-apache-2-4-and-python-3-4-on-windows/42307082#42307082

http://www.flask.moe/windows
https://stackoverflow.com/questions/42298503/how-to-install-mod-wsgi-for-apache-2-4-and-python-3-4-on-windows

https://github.com/GrahamDumpleton/mod_wsgi/blob/develop/win32/README.rst



Python 3.6 == VC14 of MS compiler,

 https://www.apachelounge.com/download/VC14/

https://wiki.python.org/moin/WindowsCompilers

https://www.lfd.uci.edu/~gohlke/pythonlibs/#mod_wsgi  

- mod_wsgi‑4.6.4+ap24vc14‑cp36‑cp36m‑win_amd64.whl



#### WSGIDaemonProcess, WSGIProcessGroup  

windows에서는 지원 안 함

http://modwsgi.readthedocs.io/en/develop/configuration-directives/WSGIDaemonProcess.html

http://modwsgi.readthedocs.io/en/develop/configuration-directives/WSGIProcessGroup.html

## virtual host

/etc/apache2/sites-enabled/000-default.conf   ~~ C:\xampp\apache\conf\extra\httpd-vhosts.conf (근데 그냥 httpd.conf에서 해줘도 무방한듯)

http://zzaps.tistory.com/245



# redis

https://github.com/MicrosoftArchive/redis/releases

msi로 설치, 서비스에 등록되어 자동으로 컴터 키면 redis-server 실행된 상태가 된다

# Flask Test

```python
#init.py
import flask
app = flask.Flask(__name__)
@app.route("/")

def hello_world():
   return "Hello World!"
```

```
#httpd.conf

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
#hello_world.wsgi
import sys
sys.path.insert(0, 'D:/pibex')

from test import app as application
```

# real game

###tensorflow-cpu

redis-server와 apache는 서비스로 설치되어 있기 때문에 윈도우를 켜면 미리 켜져 있다

```
redis-cli 치고 shutdown 
으로 redis-server를 죽일 수 있다 
(apache는 httpd.exe -k stop)
```

설정파일

```
#httpd.conf
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



```
curl -X POST -F image=@jemma.png 'http://imnotkind.ml:5522/predict'
```

curl -X나 -F는 윈도우에 없어서 리눅스로 실험

###stress test

``` 
KERAS_REST_API_URL = "http://localhost:5522/predict" 
```

url 바꿔주기 (포트 5522로)

cpu로 하니 avg batch size 32, 느림

###tensorflow-gpu stress test

avg batch size 6~7,  매우 빠름