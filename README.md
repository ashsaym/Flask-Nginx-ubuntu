

##Step 1 — Installing the Components from the Ubuntu Repositories

sudo apt update
sudo apt install python3-pip python3-dev build-essential libssl-dev libffi-dev python3-setuptools

##Step 2 — Creating a Python Virtual Environment

sudo apt install python3-venv
mkdir ~/StudyWebApp
cd ~/StudyWebApp
python3 -m venv appvevn
source appvevn/bin/activate

##Step 3 — Setting Up a Flask Application

pip install wheel
pip install gunicorn flask

nano run.py

from flask import Flask
app = Flask(__name__)

@app.route("/")
def hello():
    return "<h1 style='color:blue'>Hello There!</h1>"

if __name__ == "__main__":
    app.run(host='0.0.0.0')


sudo ufw allow 5000
python run.py

##Step 4 :Creating the WSGI Entry Point

nano wsgi.py

from run import app

if __name__ == "__main__":
    app.run()

##Step 5 — Configuring Gunicorn

cd ~/StudyWebApp
gunicorn --bind 0.0.0.0:5000 wsgi:app
deactivate

##Step 6 : Create a service
sudo nano /etc/systemd/system/StudyWebApp.service

[Unit]
Description=Gunicorn instance to serve StudyWebApp
After=network.target

[Service]
User=saym
Group=www-data
WorkingDirectory=/home/saym/StudyWebApp
Environment="PATH=/home/saym/StudyWebApp/appvevn/bin"
ExecStart=/home/saym/StudyWebApp/appvevn/bin/gunicorn --workers 3 --bind unix:StudyWebApp.sock -m 007 wsgi:app

[Install]
WantedBy=multi-user.target


sudo systemctl start StudyWebApp
sudo systemctl enable StudyWebApp
sudo systemctl status StudyWebApp


##Step 7 — Configuring Nginx to Proxy Requests

sudo apt-get install nginx
cd /etc/nginx/sites-available/
sudo nano /etc/nginx/sites-available/StudyWebApp

server 
    listen 80;
    server_name saym.ddns.net;

    location / {
        include proxy_params;
        proxy_pass http://unix:/home/saym/StudyWebApp/StudyWebApp.sock;
    }
}


sudo ln -s /etc/nginx/sites-available/StudyWebApp /etc/nginx/sites-enabled

sudo nginx -t
sudo systemctl restart nginx
sudo ufw delete allow 5000
sudo ufw allow 'Nginx Full'


##Step 8 — Securing the Application
sudo add-apt-repository ppa:certbot/certbot
sudo apt install python3-certbot-nginx
sudo certbot --nginx -d saym.ddns.net

sudo ufw delete allow 'Nginx HTTP'
