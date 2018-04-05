# WEB DEVELOPMENT WITH NODE AND EXPRESS

### CAP 1 - Introducing Express

Estamos en la era de JavaScript. Ya no es necesario un cambio de contexto para pasar de la parte de cliente a la de servidor. No es que todo lo que acontece a servidor sea únicamente relevante por el lenguaje de programación pero es más fácil trabajar con él si es el mismo lenguaje.

Se describe a Express en su página web como un minimalista y flexible framework para hacer aplicaciones web:

* Minimalista: Trata de dar algo potente pero sencillo con lo que puedas expresar todas tus ideas.
* Flexible: En vez de tener multitud de características y sólo usar unas pocas, express viene con pocas y según vamos necesitando podemos ir añadiendo al gusto.
* Framework de aplicaciones web: Una aplicación web es un sitio web, pero también puede ser algo que provee de funcionalidad a otro sitio web. 
* Single Page Aplication: Se descarga todo o casi todo el código de la web en la primera petición y luego se va construyendo sobre él según el usuario va navegando. 
* Multipágina: Es el enfoque típico en el que cada página de una web es recibida a través de una petición. Se puede mezclar esto con SPA y hacer algo **híbrido**. 

Express está inspirado en Sinatra de Ruby. Además está muy relacionado con Connect, que es una librería para conectar plugins/middleware con los que tratar las peticiones recibidas.

Node, al igual que express, está hecho para ser sencillo de levantar y configurar. Otra de las características únicas es que corre en un único hilo de ejecución. El resto de tecnologías multi-hilo son muy potentes pero son difíciles de configurar para poder utilizar al máximo el hardware sobre el que se ejecuten. 

En este libro se referirá a "JavaScript stack" por lo que incluya a Node, Express y MongoDB. 

Licencias: Al utilizar multitud de paquetes con npm puede ser complicado saber que licencia utilizar. Lo más fácil es utilizar la licencia MIT ya que tiene un rango muy amplio. De todos modos podemos utilizar herramientas para saber que licencia tienen los paquetes que utilizamos y sus dependencias como license-sniffer o license-spelunker. También es típico utilizar varias licencias con el proyecto para no cerrar puertas a proyectos que luego encadenen con ese. Por ejemplo está bien licenciar con GPL y MIT.



### CAP 2 - Getting started with Node

Familiarizarse con uso de la terminal y un editor como vi o emacs. Para gestionar los paquetes de node lo haremos con npm, utilizando la opción -g si queremos instalar el paquete de manera global en el sistema.

Para poder cambiar entre versiones de node podemos utilizar **nvm**.

**localhost**: hace referencia al propio PC en el que te encuentres. La dirección es 127.0.0.1 en IPv4 y ::1 en IPv6. 

Node ofrece un paradigma diferente al resto de soluciones ya que la aplicación que escribimos se tratará del propio servidor web. Se hace en unas pocas líneas.

En el cap7 indicará por qué no quiere introducir HTML en ficheros JS.

La filosofía principal detrás de node es la **programación orientada a eventos** (como por ejemplo puede ser la recepción de una petición http). 

**Routing:** Se refiere al mecanismo de servir al cliente lo que ha solicitado. Para aplicaciones web será a través de la URL. Como ejemplo:

``` javascript
var http = require('http');

http.createServer(function(req,res){
        // normalize url by removing querystring, optional
        // trailing slash, and making it lowercase
        var path = req.url.replace(/\/?(?:\?.*)?$/, '').toLowerCase();
        switch(path) {
                case '':
                        res.writeHead(200, { 'Content-Type': 'text/plain' });
                        res.end('Homepage');
                        break;
                case '/about':
                        res.writeHead(200, { 'Content-Type': 'text/plain' });
                        res.end('About');
                        break;
                default:
                        res.writeHead(404, { 'Content-Type': 'text/plain' });
                        res.end('Not Found');
                        break;
        }
}).listen(3000);
```

>**CONSEJO:** Servir recursos estáticos con node es sólo recomendable para desarrollo o pequeños proyectos. Pero para proyectos más grandes lo recomendable es tener un servidor proxy como nginx o CDN por encima para servir dichos ficheros.

La razón por la que no es recomendable servir ficheros estáticos es porque el servidor tendrá que leerlos. Para hacerlo de manera asíncrona lo haremos con **fs.readFile**.



### CAP 3 - Saving Time with Express

**Scaffolding:** Se trata de la generación automática del esqueleto principal sobre el que construir la página. El primero en introducir algo así fue Ruby on Rails.

Una vez tengamos creada la carpeta del proyecto (se recomienda tener un directorio site separado de documentación y otros ficheros referentes al proyecto), lanzamos el comando npm para inicializar el proyecto y generar el fichero *package.json*:
``` javascript
npm init
```

Lo siguiente que hacemos es instalar express:
``` javascript
npm install --save express
```

Para evitar que se sincronice con nuestro repositorio el directorio de módulos hay que crear el fichero *.gitignore* con el siguiente contenido:
``` 
# ignore packages installed by npm
node_modules
```

Pese a lo que indican otros tutoriales de llamar al fichero principal *app.js* o *index.js*, él recomienda llamarlo como al nombre del proyecto para que así resulte más fácil distinguirlo de otros proyectos cuando tengamos abiertos varios en el editor. npm init utiliza *index.js* por defecto.

El **puerto con el que escuchamos** en el servidor lo inicializamos con lo siguiente, utilizando la variable de entorno PORT y si no existe utilizaremos el 3000:
```
app.set(port, process.env.PORT || 3000)
```

Para añadir rutas que controlar utilizaremos **app.get**. Se puede utilizar con otros métodos (post, delete, ...). Tomaría cualquier url que entre dentro de la ruta especificada. El código se lee de arriba a abajo por lo que algo más específico debe ir arriba y más genérico abajo. Cuando se cumpla la ruta llamará a la función que indique pasándole los objetos de petición y de respuesta. Para responder utilizamos **res.send**, utilizando .type, .set y .status para indicar los distintos campos de la cabecera y poder enviarla. Por ejemplo:
``` javascript
app.get('/about', function(req, res){
        res.type('text/plain');
        res.send('About Meadowlark Travel');
});
```

Para generar las páginas 404, 500, ... lo hacemos de manera diferente. Utilizamos **app.use** que es la manera que tiene Express de introducir middleware (en cap10). Habrá que poner estos .use detrás de los .get para que no impidan que se puedan comprobar dichas rutas. (Los errores se verán en profundidad en cap10 y cap12)
``` javascript
// custom 404 page
app.use(function(req, res){
        res.type('text/plain');
        res.status(404);
        res.send('404 - Not Found');
});

// custom 500 page
app.use(function(err, req, res, next){
        console.error(err.stack);
        res.type('text/plain');
        res.status(500);
        res.send('500 - Server Error');
});
```

De momento Express nos ahorra mucho del trabajo que tuvimos que estar haciendo en el capítulo anterior para poder parsear la url a algo que si pudiésemos tratar.

**Vistas:** Es lo que se le entrega al usuario. En este caso es HTML. Podría ser PNG, PDF, ... es decir cualquier cosa que renderice el usuario. Nos centraremos en HTML. Una vista difiere ante un recurso estático en cuanto a que la vista se puede generar en tiempo real. Para generarlas haces uso de motores como puede ser **Jade** o **Handlebars**, que es el que va a utilizar porque no hace uso de un lenguaje abstracto. 
``` javascript
npm install --save express3-handlebars
```

Tendremos un directorio *views* y dentro de él otro llamado *layouts*, que contendrán plantillas sobre las que construiremos HTML. Con esto, si queremos modificar algo del aspecto de la web sólo tendremos que tocar un fichero y no varios. Este código será como el siguiente, donde lo único que hay raro es la expresión {{{body}}} que es lo que completaremos desde el código de nuestra aplicación para cada vista: 
``` javascript
<!doctype html>
<html>
<head>
    <title>Meadowlark Travel</title>
</head>
<body>
    {{{body}}}
</body>
</html>
```

A menos que especifiquemos otro layout utilizará este ya que es lo hemos puesto por defecto. Sobre ello tenemos que ir creando las vistas (tendremos que crear tantos archivos como vistas). Por ejemplo en *views/home.handlebars*:
```
<h1>Welcome to Meadowlark Travel</h1>
```

Para mostrar estas views haremos lo siguiente:
``` javascript
app.get('/', function(req, res) {
        res.render('home');
});
```

Ya no hace falta indicar que tipo de contenido es el que vamos a mandar, sino que lo hará automáticamente handlebars (no hace falta indicar el content-type ni el código).

Para el manejo de estáticos haremos uso de un middleware, **static**. Le especificaremos los directorios en los que almacenaremos los estáticos y los irá sirviendo sin ningún trato especial. Se meterán en un directorio llamado *public* ya que se servirán los recursos sin cuestionarse nada. Dentro de ese directorio crearemos la carpeta *img* que contendrá las imágenes. Antes de declarar las rutas lo añadiremos:
``` javascript
app.use(express.static(__dirname + '/public'));
```

No hace falta especificar *public* ya que lo detectará automáticamente. Con indicar "/img/logo.png" es suficiente.

Las vistas no sirven exclusivamente para entregar contenido estático sino que se pueden modificar dinámicamente. Si por ejemplo en *about.handlebars* modificamos para que tenga una parte variable:
``` javascript
<h1>About Meadowlark Travel</h1>

<p>Your fortune for the day:</p>
<blockquote>{{fortune}}</blockquote>
```

Ahora podemos modificar el punto de nuestra aplicación donde decidimos cuando se va a esa ruta y pasarle distintos valores de la siguiente manera:
``` javascript
app.get('/about', function(req, res){
        var randomFortune =
                fortunes[Math.floor(Math.random() * fortunes.length)];
        res.render('about', { fortune: randomFortune });
});
``` 

*Esta generación de páginas dinámicamente se verá en profundidad en el cap7.*


### CAP 4 - Tidying up

En este capítulo se enseñan buenas prácticas para llevar a cabo un proyecto. 

La primera vez que haces algo, si lo haces bien te llevará quizás 5 veces más que si lo hubieras hecho chapuceramente. En cambio, cuanto más tengas que repetirlo mayor será la ventaja de haberlo hecho como es debido ya que te ahorrará trabajo. Practicar el hacer las cosas bien (perfectas) conseguirá que puedas perfeccionar tu labor, será tu rutina.

Puntos favorables del **control de versiones**: 

* Documentación: Poder volver atrás en el tiempo del proyecto te da idea de por qué se hicieron las cosas como se hicieron. 
* Atribución: Si encuentras algo poco claro en el código en un proyecto entre distintas personas puedes ver quién lo introdujo/modificó y preguntarle.
* Experimentación: Permite fácilmente crear ramas nuevas donde probar cosas. Si van bien puedes incorporarlo al proyecto, sino siempre puedes desechar la idea y volver a donde estabas.

Con el fichero *.gitignore* conseguiremos evitar añadir ficheros que no deberíamos. Por cada línea podemos indicarle distintos ficheros o directorios que queremos excluir del control de versiones. Acepta comodines (por ejemplo para borrar backups podemos utilizar *\*~*). Interesa poner *node_modules* para que no los suba al repositorio. En los macs también hay que evitar *.DS_STORE*. Las máscaras de ficheros también afectan en los subdirectorios por lo que por ejemplo no se tendrán en cuenta tampoco ficheros de backup que haya dentro de un directorio.

Crearemos las ramas de experimentación con el siguiente comando:
```
git checkout -b experiment
```

**NPM**: Los node_modules se indican en el *package.json*. El versionado de los paquetes se hace según las reglas de semver. Todos los paquetes de node que utilicemos deberán estar ahí para que posteriormente, cuando queramos utilizar la aplicación en otro PC podamos conseguir todos los paquetes necesarios con:
```
npm install
```

Los **metadatos del proyecto** se introducirán dentro del mismo fichero. Se puede ver como hay que indicarlos (al iniciarlo sale un formulario) en la [documentación de npm](https://docs.npmjs.com/getting-started/using-a-package.json).

Otro fichero importante es **README.md** que contiene la información básica para que alguien nuevo en el proyecto pueda utilizarlo.

Podemos crear **módulos propios**. No es recomendable hacerlo en *node_modules* sino en otro directorio para poder diferenciarlos. Recomendable hacerlo en *lib*. Por ejemplo creamos el fichero *lib/fortune.js* que contiene el código de nuestro módulo (sólo será visible lo que esté dentro de exports, el resto estará encapsulado y libre de que pueda ser modificado desde fuera):

``` javascript
var fortuneCookies = [
        "Conquer your fears or they will conquer you.",
        "Rivers need springs.",
        "Do not fear what you don't know.",
        "You will have a pleasant surprise.",
        "Whenever possible, keep it simple.",
];

exports.getFortune = function() {
        var idx = Math.floor(Math.random() * fortuneCookies.length);
        return fortuneCookies[idx];
};
```

Y requeriremos dicho módulo en nuestra aplicación principal:
``` javascript
var fortune = require('./lib/fortune.js');
```

Y luego podemos usar las funciones del módulo exportado:
``` javascript
app.get('/about', function(req, res) {
        res.render('about', { fortune: fortune.getFortune() } );
});
```


### CAP 5 - Quality assurance


### CAP 6 -


### CAP 7 -



### DUDAS: 

* ¿Por qué con node no se utiliza un servidor web por encima y con otros si que se utiliza un apache o lo que se necesite? ¿O estoy equivocado?
