# deploy-flask-use-apache-and-wsgi
deploy flask use apache and wsgi

Bình thường khi các bạn code xong app flask, các bạn thường để nó ở chế độ dev bằng cầu lệnh đơn giản python index.py. Sau đó mở host cho truy cập tất cả, như vậy là xong một con api đơn giản. Các bạn có thể tham khảo simple api của mình dùng flask. https://github.com/nghiahsgs/simple-flask-server. Trong bài viết này mình hướng dẫn các bạn deploy app flask sử dùng apache 2 và gắn domain cho app flask.

## Bước 1: Cài và kích hoạt chế độ mod_wsgi
WSGI (webserver gateway interface) là interface giữa web server và app flask. Cơ bản nó sẽ kết nối apache và flask với nhau.
```
sudo apt-get install libapache2-mod-wsgi python-dev
sudo a2enmod wsgi
```

## Bước 2: Tạo Flask app theo cấu trúc như sau
```
cd /var/www
sudo mkdir FlaskApp
cd FlaskApp
sudo mkdir FlaskApp
cd FlaskApp
sudo mkdir static templates
```
```
Cấu trúc thư mục sẽ kiểu 
|/var/www
|----FlaskApp
|---------FlaskApp
|--------------(__init__.py)
|--------------static
|--------------templates
```


Mở file ```__init.py__``` và cho đoạn code mẫu tạo 1 server sẽ hello khi vào trang chủ.
```
from flask import Flask
app = Flask(__name__)
@app.route("/")
def hello():
    return "Hello, I love Digital Ocean!"
if __name__ == "__main__":
    app.run()
```

## Bước 3: Cài đặt flask
Lưu ý là các câu lệnh phải chạy ở sudo nhé.
Cài pip hoặc pip3 (Tùy vào môi trường của bạn mà dùng pip hay pip3)
```
sudo apt-get install python-pip 
sudo apt-get install python3-pip
```

Dùng pip hoặc pip3 cài Flask
```
sudo pip install Flask 
sudo pip3 install Flask 
```

Sau khi cài xong thì chạy thử xem cài đặt flask thành công hay chưa.
```
sudo python __init__.py 
hoặc
sudo python3 __init__.py
```

## Bước 4: Cài đặt file cấu hình trong apache
```
vi /etc/apache2/sites-available/FlaskApp.conf
```
Mở file FlaskApp.conf cho nội dung sau vào

```
<VirtualHost *:80>
		ServerName mywebsite.com
		ServerAdmin admin@mywebsite.com
		WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
		<Directory /var/www/FlaskApp/FlaskApp/>
			Order allow,deny
			Allow from all
		</Directory>
		Alias /static /var/www/FlaskApp/FlaskApp/static
		<Directory /var/www/FlaskApp/FlaskApp/static/>
			Order allow,deny
			Allow from all
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
mywebsite.com chính là domain của các bạn cần trỏ vào, error.log là phần log lại khi web bạn có vấn đề gì. Xem nó ở thư mục /var/log/apache2/error.log

Sau khi tạo xong file cấu hình thì kích hoạt nó lên bằng câu lệnh sau
```
sudo a2ensite FlaskApp
```

## Bước 5: tạo file wsgi
```
cd /var/www/FaskApp
vi flaskapp.wsgi (tên file giống tên file bạn đã đặt trong file cấu hình apache)
```

```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/FlaskApp/")

from FlaskApp import app as application
application.secret_key = 'Add your secret key'
```

Tính đến hiện tại, cấu trúc thư mục sẽ như này
```
|--------FlaskApp
|----------------FlaskApp
|-----------------------static
|-----------------------templates
|-----------------------venv
|-----------------------__init__.py
|----------------flaskapp.wsgi
```
## Bước 6: restart apache
```
sudo service apache2 restart 
```
Sau đó bật trình duyệt, vào domain của bạn và tận hưởng (mywebsite.com). Bạn nào chưa có domain thì đổi nó ở file hosts
