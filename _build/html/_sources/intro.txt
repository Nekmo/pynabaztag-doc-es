===============================
Introducción: Qué es PyNabaztag
===============================
PyNabaztag es un servidor completo en Python para Nabaztag, que incluye las 2 partes indispensables de un servidor Nabaztag: el servidor HTTP y el servidor XMPP. Ambas partes, tienen su uso determinado para el funcionamiento del conejo.

**HTTP**
    Nabaztag realiza una primera petición a través de HTTP (puerto 80) para solicitar un binario que ejecutará (el sistema) y las direcciones de conexión. Después, cuando ya esté correctamente identificado a través de XMPP, solicitará al servidor archivos de audio y realizará determinadas peticiones por HTTP. A parte, la página de configuración y administración se sirve a través de HTTP, aunque esto no se encuentra estrictamente relacionado con el funcionamiento del Nabaztag.
**XMPP**
    Tras obtener las direcciones de conexión por HTTP (entre las que se encuentra la dirección del servidor XMPP), Nabaztag se identificará con el servidor XMPP (puerto 5222). XMPP es el acrónimo de "eXtensible Messaging and Presence Protocol", el cual se usa como protocolo de Mensajería Instantánea para las comunicaciones entre servidor y cliente. Nabaztag usará XMPP para comunicarse con el servidor, y viceversa (bidireccional). Por ejemplo, Nabaztag usará XMPP para comunicar de los eventos de pulsación de botón o de movimiento de orejas. En cambio, usará HTTP para peticiones como el envío del archivo de audio de cuando se habla con Nabaztag. El servidor SIEMPRE se comunicará con Nabaztag mediante XMPP, ya que HTTP no soporta PUSH (es decir, no es bidireccional). Aún así, el servidor cuando tiene que ofrecerle un archivo binario o muy grande a Nabaztag, le ofrecerá una URL para descargar o hacer streaming. Ejemplo: El servidor quiere que Nabaztag haga sonar una canción. En tal caso, el servidor enviará por XMPP una URL HTTP con la orden "Reproduce este sonido".

--------------------------
Organización de PyNabaztag
--------------------------
La siguiente es la organización de los directorios y archivos en PyNabaztag

**share/**
    Archivos que se entregarán a Nabaztag a través de HTTP.
**pynabaztag/**
    Código fuente del servidor.
    
    **pynabaztag/blocks/**
        Funciones Packet y Blocks para comunicación S->C.
    **pynabaztag/xmppmod/**
        Servidor XMPP.
    **pynabaztag/httpmod/**
        Servidor HTTP.
    **pynabaztag/plugins/**
        Plugins que expandirán las capacidades HTTP y XMPP de PyNabaztag.
    **pynabaztag/tts/**
        Módulo de sintetización de voz.
**server.py**
    Main. Ejecutable que pondrá en funcionamiento el servidor.
**urls.py**
    Rutas que se servirán por defecto a través de HTTP. Permite relacionar un path de URL con una función del sevidor, o que un path se relacione con un directorio para servir archivos estáticos.