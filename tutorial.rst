===============================
Tutorial: Creación de un plugin
===============================
Uno de los objetivos de PyNabaztag, es la modularidad. Toda capacidad añadida al servidor, debería de ser fácilmente activable y desactivable sin que ello requiera modificación alguna de código. Ello permite mayor personalización y encontrar más fácilmente errores.

En el siguiente tutorial se expondrá cómo crear un plugin que cumpla las siguientes características:

#. El plugin sólo se activará y funcionará si el usuario lo desea.
#. Nabaztag hablará al recibir un evento.
#. Dicho evento deberá poder ser configurado por el usuario.
#. El evento podrá ser reestablecido.
#. Habrá un texto por defecto que será dicho, el cual será "Hola Mundo". Dicho texto podrá ser cambiado por el usuario, sin afectar a los demás usuarios del servidor.
#. Las funcionalidades del plugin deberán poder ser desactivadas por el usuario para sí mismo, borrando datos de su presencia.
#. El plugin se llamará "saythis".

------------------------
1. Crear un plugin nuevo
------------------------
Lo primero de todo, crearemos un archivo en *pynabaztag/plugins/* llamado *saythis.py*, y pegaremos la siguiente plantilla que nos guiará en la creación de nuestro plugin.

.. code-block:: python

    # -*- coding: utf-8 -*-
    # from threading import Timer ## Tareas que se ejecutarán en X tiempo.
    # from .. blocks import Packet, MessageBlock ## Comunicación S->C.
    from . import base

    # Diccionario con configuraciones por defecto
    DEFAULT_CFG = {
        # 'rabbids': {}, ## configuraciones en cada conejo.
    }

    class Pluginname(base.Base):
        def plugin_init(self):
            # Se ejecutará justo en el momento de importarse el plugin. Úsese
            # solo para aquellos casos en que este plugin sirva de biblioteca
            # para otros plugins.
            pass
        def post_init(self):
            # Se ejecutará tras haberse ejecutado todos los demás plugins. Si
            # su plugin va a usar otro plugin de biblioteca (por ejemplo, el
            # plugin events), ponga aquí aquellas cosas que le hagan uso. De
            # lo contrario puede provocarse una excepción, ya que si se pone
            # en plugin_init, el plugin events podría no haberse importado
            # todavía.
            pass
        def web_user_options(self, hconn):
            # Página de configuración del plugin
            pass
        def enable_rabbid(self, hconn):
            # El Nabaztag ha sido activado a través de la página de plugins
            pass
        def disable_rabbid(self, hconn):
            # El Nabaztag fue desactivado en la lista de plugins.
            pass

La clase "Pluginname" será la que se instancie, y será donde trabajemos en nuestro plugin. El nombre de la clase deberá ser el mismo que el del plugin, pero con la primera letra mayúscula. En nuestro caso: ::

    class Saythis(base.Base):
        def plugin_init(self):
            # Se ejecutará justo en el momento de importarse el plugin. Úsese
            # solo para aquellos casos en que este plugin sirva de biblioteca
            # para otros plugins.
            pass