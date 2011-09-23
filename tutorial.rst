Tutorial: Creación de un plugin
===============================
Uno de los objetivos de PyNabaztag, es la modularidad. Toda capacidad añadida al servidor, debería de ser fácilmente activable y desactivable sin que ello requiera modificación alguna de código. Ello permite mayor personalización y encontrar más fácilmente errores.

En el siguiente tutorial se expondrá cómo crear un plugin que cumpla las siguientes características:

#. El plugin sólo se activará y funcionará si el usuario lo desea.
#. Nabaztag hablará al recibir un evento (pulsación de botón, objeto RFID...).
#. Dicho evento deberá poder ser configurado por el usuario.
#. El evento podrá ser reestablecido.
#. Habrá un texto por defecto que será dicho, el cual será "Hola Mundo". Dicho texto podrá ser cambiado por el usuario, sin afectar a los demás usuarios del servidor.
#. Las funcionalidades del plugin deberán poder ser desactivadas por el usuario para sí mismo, borrando datos de su presencia.
#. El plugin se llamará "saythis".

1. Crear un plugin nuevo
------------------------
Lo primero de todo, crearemos un archivo en el directorio **pynabaztag/plugins/** llamado **saythis.py**, y pegaremos la siguiente plantilla que nos guiará en la creación de nuestro plugin.

.. code-block:: python

    # -*- coding: utf-8 -*-
    # from threading import Timer ## Tareas que se ejecutarán en X tiempo.
    # import gettext ## Locales
    # from .. blocks import Packet, MessageBlock ## Comunicación S->C.
    from . import base

    # Diccionario con configuraciones por defecto
    DEFAULT_CFG = {
        # 'rabbids': {}, ## configuraciones en cada conejo.
    }
    
    # Descomentar la siguiente línea para las locales
    # _ = gettext.gettext ## Traducir texto mediante _("Texto")
    
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

Finalizado esto, haremos que en la próxima ejecución de PyNabaztag el plugin que acabamos de crear se importe. Para ello modificaremos el archivo **pynabaztag/plugins/__init__.py** añadiendo el nombre del fichero del plugin:::

    __all__ = ['events', 'taichi', (...), 'saythis']
    
.. NOTE::
   En futuras versiones de PyNabaztag, el anterior paso puede no ser necesario, siendo sustituido por una página de administración para gestionar los plugins activos.

2. Secciones de nuestro nuevo plugin
------------------------------------
Ya tenemos una plantilla de lo que será nuestro futuro plugin, pero antes de empezar a programar, es necesario comprender el uso que tendrá cada sección del código.::

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

Ambos métodos se ejecutarán en el inicio del plugin, pero la primera se ejecutará cuando los plugins aún se están importando, y la segunda cuando todos los plugins ya se han importado. En el uso general, se debe usar la segunda. La primera solo deberá usarse en casos específicos, cuando se vaya a crear algo de lo que haga uso otros plugins. 

::
    
        def web_user_options(self, hconn):
            # Página de configuración del plugin
            pass
        def enable_rabbid(self, hconn):
            # El Nabaztag ha sido activado a través de la página de plugins
            pass
        def disable_rabbid(self, hconn):
            # El Nabaztag fue desactivado en la lista de plugins.
            pass

Las 3 se encuentran relacionadas con la página de gestión del plugin a través de la web de PyNabaztag. En el caso de la primera, se mostrará la página de configuración y uso del plugin. La segunda cuando el plugin se ha habilitado por el usuario, y la tercera cuando se ha deshabilitado. Es importante remarcar, que en las 3, estaremos trabajando destinados a 1 conejo. Esto quiere decir, que mientras que en los métodos de init estábamos haciendo cosas destinadas a todos los Nabaztag que usen el plugin, en estas 3 lo haremos con 1 conejo en específico.

3. Habilitar el plugin, configuración inicial y traducción
----------------------------------------------------------
En nuestro plugin, establecíamos que por defecto, el texto a decir por Nabaztag cuando recibía un evento, debía ser "Hola mundo". Esto texto lo estableceremos como una configuración que estableceremos justo al habilitarse el plugin en la lista de plugins en la página de configuración. Ahora, surje el siguiente problema, ¿cómo almacenar configuraciones? PyNabaztag trae una solución para ello, que consiste en un diccionario individual por cada plugin, accesible mediante:::
    
    self.cfg[key]

Y aquí es donde entra en juego *DEFAULT_CFG*, que será el contenido por defecto de dicho diccionario al importarse el plugin por primera vez. Guardaremos las configuraciones por cada conejo de manera que sean accesibles de la siguiente forma:::
    
    self.cfg['rabbids'][<id del conejo>]

Para que "rabbids" sea un diccionario que almacene configuraciones de los conejos, descomentaremos "rabbids" en *DEFAULT_CFG*::

    # Diccionario con configuraciones por defecto
    DEFAULT_CFG = {
        'rabbids': {}, # configuraciones en cada conejo.
    }

.. WARNING::
   Tras haberse modificado el DEFAULT_CFG, debe tenerse en cuenta que si el plugin ya fue iniciado por primera vez, no se crearán las nuevas cosas que hayan añadido al DEFAULT_CFG. Deberemos borrar la configuración ya creada para que se vuelva a crear, con las nuevas cosas por defecto que se hayan añadido. El archivo de configuración se encontrará en *~/.config/pynabaztag/plugins/saythis.json*.

Ya tenemos una base para lo que serán nuestras configuraciones para todos los conejos que usen el plugin. Ahora, la pregunta es, *¿cuál es esa "id del conejo" y cómo se consigue?* En el argumentos "hconn", tenemos un método llamado **hconn.rabbid**, el cual es un objeto Rabbid con varias cosas interesantes para trabajar con el Nabaztag en cuestión (aunque esto será otra historia... :) ). El problema, es que el objeto Rabbid no puede ser la clave del diccionario. Pero no debemos preocuparnos, porque el objeto Rabbid (cada objeto Rabbid es específico de un Nabaztag) nos ofrecerá un número de identificación mediante::
    
    str(hconn.rabbid)

Dicha identificación será el número de dirección MAC del Nabaztag, el cual se puede consultar en la base. Cada MAC es única y específica para un Nabaztag.

.. TIP::
   **Rabbid** es la unión de los nombres **Rabbit + IDentification**. Es decir, "Identificación del Conejo".

El siguiente código sería una solución al problema planteado::
    
    def enable_rabbid(self, hconn):
        """ \
        Crear la configuración inicial para el Nabaztag al activar el plugin.
        """
        # Creamos un diccionario como valor a la clave del conejo,
        # que almacene las configuraciones para el conejo
        self.cfg[str(hconn.rabbid)] = {}
        # Texto a decir por defecto en el nombre de clave "to_say".
        self.cfg[str(hconn.rabbid)]['to_say'] = 'Hola Mundo'
        # Lo siguiente guardará la configuración en el archivo
        # de configuración, para que no se pierda. Sería algo
        # así como un commit
        self.cfg.write()

Como se puede leer en el código fuente, tras hacer cambios ne la configuración, es necesario hacer al final lo siguiente para que se guarde en el archivo de configuración::
    
    self.cfg.write()

Algo que no se ha planteado como un requisito, pero que sería recomendable, es permitir la localización (traducción a otros idiomas) de nuestro plugin. De esta manera, el texto por defecto se encontrará en su propio idioma. Para habilitar la localización, descomentaremos lo siguiente::
    
    import gettext # Locales
    [...]
    # Descomentar la siguiente línea para las locales
    _ = gettext.gettext # Traducir texto mediante _("Texto")

Y modificaremos la línea donde se establece el texto por defecto como se muestra aquí::
    
    def enable_rabbid(self, hconn):
        """ \
        Crear la configuración inicial para el Nabaztag al activar el plugin.
        """
        # Creamos un diccionario como valor a la clave del conejo,
        # que almacene las configuraciones para el conejo
        self.cfg[str(hconn.rabbid)] = {}
        # Texto a decir por defecto en el nombre de clave "to_say".
        # Será traducido al idioma del usuario con el _(...)
        self.cfg[str(hconn.rabbid)]['to_say'] = _('Hola Mundo')
        # Lo siguiente guardará la configuración en el archivo
        # de configuración, para que no se pierda. Sería algo
        # así como un commit
        self.cfg.write()

4. Formulario de configuración y uso
------------------------------------
Uno de los puntos más notables de PyNabaztag es, su framework para creación de formularios de configuración. En sólo unas pocas líneas, es posible crear un formulario que recoja unos datos, los verifique en el lado del cliente, los valide en el lado del servidor y se guarden en la configuración. Los formularios se componen de 3 partes fundamentes: Una primera donde se crea el formulario, una segunda donde se definen los campos, y una tercera donde se hará lo necesario tras tener los datos dados por el usuario. ::

    def web_user_options(self, hconn):
        # Diccionario con la configuración del conejo
        rabbid_cfg = self.cfg['rabbids'][str(hconn.rabbid)]
        # Se crea el objeto formulario. El segundo argumento será un
        # diccionario sobre el que se guarden los valores devueltos por
        # el usuario en los campos. El atributo title será el título que
        # encabece el formulario.
        form = base.Form(hconn, rabbid_cfg, title='Texto a decir')
        # ... Se establecen los campos del formulario aquí ...
        if form.no_data(): return form
        # ... A partir de aquí, se poseen los valores devueltos por el usuario. ...

Recordemos las configuraciones que necesitaremos poder parametrar:
# El texto a decir por el conejo
# Los eventos que activarán la función "decir el texto". 

Para ambas cosas, hay campos en el formulario que nos posibilitarán hacer esto muy fácilmente. ::
    
    # Campo del formulario to_say. El primer argumento es la clave en el 
    # diccionario rabbid_cfg. La segunda el texto de ayuda del campo.
    form.text('to_say', 'Texto a decir')
    # Campo del formulario para seleccionar los eventos. El primer argumento
    # Es la función que se ejecutará al recibirse el evento.
    form.events(self.say_text)
        
.. NOTE::
   Al usar los formularios no debe preocuparse de la localización. Aquellos argumentos que se considere que son de ayuda y visibles al usuario, se traducirán.
   
Antes que nada, avisar de que hemos definido una función que se ejecutará al recibirse el evento llamada *self.say_text*, como se puede ver en el código.

En este ejemplo, solo hemos usado una pequeña parte de las posibilidades que nos ofrecen los campos del Framework para Formularios de PyNabaztag. Hay más posibilidades que se irán ampliando, para facilitar la programación. Llegados a este punto, pueden surjir ciertas dudas: *¿qué pasa si el usuario no rellena el campo? ¿Se mostrará en el campo el valor que se colocó la última vez?*  A la primera pregunta, es necesario saber que el campo text obligará a rellenar el campo. Si no se rellena, saltará un aviso. Para desactivar esto, añada el argumento *req=False*. A la segunda pregunta, el campo pondrá el texto rellenado por última vez, sacándolo de rabbid_cfg. Esto se puede impedir con el argumento *default='texto a poner'*.

PyNabaztag se encargará, como ya hemos dicho, de guardar las configuraciones en el diccionario de configuración den conejo. Lo único que tendremos que hacer será añadir al final, en la tercera sección del formulario, que se graben los datos aportados en el archivo de configuración. ::
    
    # ... A partir de aquí, se poseen los valores devueltos por el usuario. ...
    self.cfg.write()

El resultado final sería el siguiente::
    
    def web_user_options(self, hconn):
        # Diccionario con la configuración del conejo
        rabbid_cfg = self.cfg['rabbids'][str(hconn.rabbid)]
        # Se crea el objeto formulario. El segundo argumento será un
        # diccionario sobre el que se guarden los valores devueltos por
        # el usuario en los campos. El atributo title será el título que
        # encabece el formulario.
        form = base.Form(hconn, rabbid_cfg, title='Texto a decir')
        # ... Se establecen los campos del formulario aquí ...
        # Campo del formulario to_say. El primer argumento es la clave en el 
        # diccionario rabbid_cfg. La segunda el texto de ayuda del campo.
        form.text('to_say', 'Texto a decir')
        # Campo del formulario para seleccionar los eventos. El primer argumento
        # Es la función que se ejecutará al recibirse el evento.
        form.events(self.say_text)
        if form.no_data(): return form
        # ... A partir de aquí, se poseen los valores devueltos por el usuario. ...
        self.cfg.write()

Esto generará un formulario con 2 campos: Un campo de texto para introducir el texto, y un cuadro de selección múltiple para elegir los eventos.

4. Comunicación Servidor -> Cliente
-----------------------------------