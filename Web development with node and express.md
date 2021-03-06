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

En este libro se referirá a "JavaScript stack" en lo referente a a Node, Express y MongoDB. 

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

**¿Acaso hay alguien que no quiera calidad en su software?** Hay dos tipos de organizaciones, las que son grandes y tienen separado el rol de Desarrollo y el de Aseguramiento de Calidad (que muchas veces se ven como enemigos) y otras empresas donde los dos roles recaen sobre la misma persona (cuando hay prisa por lo general lo primero que no se hace es el QA). 

¿Vale la pena el QA? Nadie quiere hacer un producto de calidad dudosa pero la presión por sacar un producto muchas veces es alta. 

Los **puntos clave** de la QA son:

* **Alcance:** Cuanta más gente llegue a tu sitio, mayor mercado tendrás al que vender tus productos. Posicionamiento (SEO). *Puede ser automatizado.*
* **Funcionalidad:** Cuanto mejor sea la funcionalidad y cumpla con lo indicado más harás retener al usuario. *Muchas veces se puede automatizar.*
* **Usabilidad:** Evalúa la interacción entre humano y máquina. ¿Es fácil de usar? Tienes que tener claro tu público objetivo. No es algo que se pueda automatizar, porque para eso necesitas un usuario, pero debes incluir pruebas de usuario.
* **Estética:** El más subjetivo. Hay que enseñarlo para ver que opina la gente, teniendo en cuenta que no todo el mundo es igual.  

Hay que tener una distinción clara entre lógica y presentación. Las cosas tienen que ser lo más claras o simples posibles en la parte lógica, mientras que en la de presentación tendrán que ser como se requiera. 

**Dos tipos de test, unitarios y de integración**. Los unitarios revisan funcionalidades muy completas mientras que los de integración revisan una cadena de acciones o bien el sistema completo.

Revisión de **técnicas** de QA:

* **Page testing:** Testea la presentación y funcionalidad del frontend de la página. Puede requerir test unitarios y de integración. *Lo haremos con **Mocha**.*
* **Cross-page testing:** Requiere ir navegando de una página a otra. Al requerir más de un componente por lo general cae dentro del rango de pruebas de integración. *Utilizaremos **zombie.js**.*
* **Logic Testing:** Lanzará pruebas unitarias y de integración contra la parte lógica, el backend, desconectado de la capa de presentación.
* **Linting:** No se trata de encontrar errores, sino errores potenciales. *Usaremos **JSHint** para ello*
* **Link checking:** Test unitarios que comprueban que los link que contiene la página se encuentran disponibles. *Utilizaremos **LinkChecker**.*

Puedes encargarte de parar y arrancar el servidor cada vez que hagas un cambio. O puedes delegar esto a herramientas para que si detectan un cambio reinicien el servidor. Ejemplo de ello son **nodemon** y **Grunt**. 

###### Page testing
La recomendación es que introduzcas test en tus páginas mientras te encuentras desarrollando la aplicación. 

Lo instalaremos como dependencia exclusiva para la parte de desarrollo, lo que evita tener esas dependencias en Producción:
``` javascript
npm install --save-dev mocha
```

Para poder realizar las pruebas en el cliente hay que mover los ficheros de mocha a la carpeta public. Lo ponemos en la carpeta vendor que es donde se debería poner código que no pertenece propiamente a la aplicación:

``` javascript
mkdir public/vendor
cp node_modules/mocha/mocha.js public/vendor
cp node_modules/mocha/mocha.css public/vendor
```

También necesitamos código que permita hacer asserts:

``` javascript
npm install --save-dev chai
cp node_modules/chai/chai.js public/vendor
```

Luego podemos poner código en nuestras vistas que se ejecute condicionalmente, solo en caso de que queramos correr los test y pudiendo indicar que se haga solo si es un entorno no Productivo. Lo indicaremos por ejemplo con el siguiente parámetro:
``` javascript
http://localhost:3000/?test=1
``` 

Con mocha puedes hacer test de distintas maneras:

* BDD: Behaviour-driven development. Describes componentes y comportamientos, y los test comprueban esos comportamientos. 
* TDD: Test-driven development. Defines unos test y unos resultados para esos test. Es la escogida en el libro. Un test de este estilo será por ejemplo (*test-global.js*):

``` javascript
suite('Global Tests', function(){
  test('page has a valid title', function(){
          assert(document.title && document.title.match(/\S/) &&
                  document.title.toUpperCase() !== 'TODO');
  });
});
```

Puedes hacer tests que sean globales, que sirvan para cualquier página, o puedes hacer tests específicos para cada página.


###### Cross-page testing

Se trata de simular el salto de unas páginas a otras. Para poder hacerlo sin tener que estar navegando a través de un navegador hay herramientas muy útiles como **zombie**. Para poder utilizarlo necesitaremos tener a mocha instalado globalmente y luego ejecutar el script creado:
``` javascript
var Browser = require('zombie'),
        assert = require('chai').assert;

var browser;

suite('Cross-Page Tests', function(){

        setup(function(){
                browser = new Browser();
        });

        test('requesting a group rate quote     from the hood river tour page' +
                'should populate the referrer field', function(done){
                var referrer = 'http://localhost:3000/tours/hood-river';
                browser.visit(referrer, function(){
                        browser.clickLink('.requestGroupRate', function(){
                                assert(browser.field('referrer').value
                                        === referrer);
                                done();
                        });
                });
        });
        
        //... (otros test)
```

Lanzamos la prueba:
``` javascript
mocha -u tdd -R spec qa/tests-crosspage.js 2>/dev/null
```

###### Logic testing

También utilizamos **mocha** para realizar los test lógicos. Se trata de llamar a las funciones para ver si devuelven algún valor que concuerde con lo esperado con **chai**. Se crea el fichero 
``` javascript
var fortune = require('../lib/fortune.js');
var expect = require('chai').expect;

suite('Fortune cookie tests', function(){

    test('getFortune() should return a fortune', function(){
        expect(typeof fortune.getFortune() === 'string');
    });

});
```

Y lo ejecutamos con:
``` javascript
mocha -u tdd -R spec qa/tests-unit.js
```

###### Linting

Es como tener un segundo par de ojos que revise el código. En principio estaba **JSLint**, pero de él se generó **JSHint** que es el recomendado actualmente. Lo puedes instalar de manera global para ejecutarlo sobre scripts pero lo normal es que ya esté instalado e integrado en el editor que utilices.

###### Link Checking

Se trata de comprobar que el sitio no tiene ningún link roto. Para ello instalamos el programa **LinkChecker** y pasamos el test:
``` javascript
linkchecker http://localhost:3000
```

###### Automatizando con Grunt

Todo lo que hemos estado viendo previamente es fácil de olvidar hacerlo. Para ello lo mejor es automatizarlo y para eso utilizaremos **Grunt**. 
``` javascript
sudo npm install -g grunt-cli
npm install --save-dev grunt
```

Funciona mediante [plugins](https://gruntjs.com/plugins) para poder hacer cada tarea. Para casos en los que no exista plugin (por ejemplo LinkChecker) se utilizará un plugin genérico para ejecutar comandos de shell.
``` javascript
npm install --save-dev grunt-cafe-mocha
npm install --save-dev grunt-contrib-jshint
npm install --save-dev grunt-exec
```

Luego creamos un fichero *Gruntfile.js* en el directorio del proyecto que contendrá las distintas automatizaciones: 
``` javascript
module.exports = function(grunt){

        // load plugins
        [
                'grunt-cafe-mocha',
                'grunt-contrib-jshint',
                'grunt-exec',
        ].forEach(function(task){
                grunt.loadNpmTasks(task);
        });

        // configure plugins
        grunt.initConfig({
                cafemocha: {
                        all: { src: 'qa/tests-*.js', options: { ui: 'tdd' }, }
                },
                jshint: {
                        app: ['meadowlark.js', 'public/js/**/*.js',
                                'lib/**/*.js'],
                        qa: ['Gruntfile.js', 'public/qa/**/*.js', 'qa/**/*.js'],
                },
                exec: {
                        linkchecker:
                                { cmd: 'linkchecker http://localhost:3000' }
                },
        });

        // register tasks
        grunt.registerTask('default', ['cafemocha','jshint','exec']);
};
``` 

Con ese código, cuando ejecutemos el comando **grunt**, se ejecutará el grupo de tareas definido como *default*, que es el que se ejecuta cuando no se indica un grupo de tareas.

###### Integración continua (CI)

Se trata de pasar automáticamente una serie de test cuando subes código a un servidor común. Es especialmente útil cuando trabajas en un equipo de varias personas pero no viene mal utilizarlo sólo para tener esa habilidad. Si falla enviará un email indicando que las cosas no están bien en el código subido. Es una manera de forzar a la gente a pasar todos los test antes de subir el código al repositorio. 

Hay soluciones que se adaptan a tu repositorio de github y que son muy cómodas, como es **Travis CI**, pero también hay servidores de CI que puedes introducir en tu servidor como es el caso de **Jenkins** con un plugin para node.




### CAP 6 - The Request and Response Objects

Cuando construyes un servidor con Express prácticamente todo lo que haces empieza con un objeto petición y acaba en un objeto respuesta.

Partes de la url (introducir imagen trozos de la url).

Los **métodos HTTP**, que es la manera que tiene de comunicarse el cliente con el servidor: GET, POST, PUT, DELETE. La combinación de método, URL y parámetros es la manera en que la app determina como actuar. Con Express escribiremos handlers para los métodos que queramos controlar.

Cabeceras:

* **Petición:** Información que remite con la petición al servidor para indicar cosas como el idioma, el navegador, OS, hardware... Es el parámetro **headers**. Si quieres ver la cabecera que envías: 

``` javascript
app.get('/headers', function(req,res){
    res.set('Content-Type','text/plain');
    var s = '';
    for(var name in req.headers) s += name + ': ' + req.headers[name] + '\n';
    res.send(s);
});
```

* **Respuesta:** El servidor también manda información al cliente que no será mostrada. Manda metadatos e información del servidor. Por ejemplo manda el content-type, que indica al navegador como debe interpretar los datos que reciba (priorizará lo que venga ahí a lo que indique con su extensión el objeto recibido). Puede indicar compresión, codificación. Para ver la cabecera de la respuesta puedes utilizar las **herramientas de desarrollo de chrome**. Para ocultar la información que mandas del servidor (y evitar puntos flacos que utilicen los hackers):

``` javascript
app.disable('x-powered-by');
```

**Content-type** viene marcado por *internet media type*, que consiste en un tipo, un subtipo y parámetros opcionales. Por ejemplo puede ser *text/html; charset=UTF-8*. 

**Cuerpo de la respuesta:** Una petición GET no tiene cuerpo pero sin embargo una POST si. Hay varias maneras de pasar esa información (en la url separando cada parámetro con &, para subir archivos se suele hacer con multipart/form-data y con ajax se puede utilizar JSON). 

**Objeto request:** Objeto con el que trabajaremos con la respuesta.  Se suele llamar req o request. Tiene todos los parámetros y métodos necesarios: parms, body, route, headers, cookies...

**Objeto response:** Objeto que tienes para preparar la respuesta. Se suele llamar res, resp o response. Parámetros y métodos: status, set, cookie, send, redirect, type, download...

Ejemplos de uso:

Example 6-1. Basic usage
``` javascript
// basic usage
app.get('/about', function(req, res){
        res.render('about');
});
```

Example 6-2. Response codes other than 200
``` javascript
app.get('/error', function(req, res){
        res.status(500);
        res.render('error');
});
// or on one line...
app.get('/error', function(req, res){
        res.status(500).render('error');
});
```

Example 6-3. Passing a context to a view, including querystring, cookie, and session values
``` javascript
app.get('/greeting', function(req, res){
        res.render('about', {
                message: 'welcome',
                style: req.query.style,
                userid: req.cookie.userid,
                username: req.session.username,
        });
});
```

Example 6-4. Rendering a view without a layout
``` javascript
// the following layout doesn't have a layout file, so views/no-layout.handlebars
// must include all necessary HTML
app.get('/no-layout', function(req, res){
        res.render('no-layout', { layout: null });
});
```

Example 6-5. Rendering a view with a custom layout
``` javascript
// the layout file views/layouts/custom.handlebars will be used
app.get('/custom-layout', function(req, res){
        res.render('custom-layout', { layout: 'custom' });
});
```

Example 6-6. Rendering plaintext output
``` javascript
app.get('/test', function(req, res){
        res.type('text/plain');
        res.send('this is a test');
});
```

Example 6-7. Adding an error handler
``` javascript
// this should appear AFTER all of your routes
// note that even if you don't need the "next"
// function, it must be included for Express
// to recognize this as an error handler
app.use(function(err, req, res, next){
        console.error(err.stack);
        res.status(500).render('error');
});
```

Example 6-8. Adding a 404 handler
``` javascript
// this should appear AFTER all of your routes
app.use(function(req, res){
        res.status(404).render('not-found');
});
```

Example 6-9. Basic form processing
``` javascript
// body-parser middleware must be linked in
app.post('/process-contact', function(req, res){
        console.log('Received contact from ' + req.body.name +
                ' <' + req.body.email + '>');
        // save to database....
        res.redirect(303, '/thank-you');
});
```

Example 6-10. More robust form processing
``` javascript
// body-parser middleware must be linked in
app.post('/process-contact', function(req, res){
        console.log('Received contact from ' + req.body.name +
                ' <' + req.body.email + '>');
        try {
                // save to database....

                return res.xhr ?
                        res.render({ success: true }) :
                        res.redirect(303, '/thank-you');
        } catch(ex) {
                return res.xhr ?
                        res.json({ error: 'Database error.' }) :
                        res.redirect(303, '/database-error');
        }
});
```

PROVIDING AN API
When you’re providing an API, much like processing forms, the parameters will usually be in req.query, though you can also use req.body. What’s different about APIs is that you’ll usually be returning JSON, XML, or even plaintext, instead of HTML, and you’ll often be using less common HTTP methods like PUT, POST, and DELETE. Providing an API will be covered in Chapter 15. Examples 6-11 and 6-12 use the following “products” array (which would normally be retrieved from a database):
```  javascript
var tours = [
        { id: 0, name: 'Hood River', price: 99.99 },
        { id: 1, name: 'Oregon Coast', price: 149.95 },
];
```

NOTE
The term “endpoint” is often used to describe a single function in an API.

Example 6-11. Simple GET endpoint returning only JSON
``` javascript
app.get('/api/tours'), function(req, res){
        res.json(tours);
});
```

Example 6-12. GET endpoint that returns JSON, XML, or text
``` javascript
app.get('/api/tours', function(req, res){
        var toursXml = '<?xml version="1.0"?><tours>' +
                products.map(function(p){
                        return '<tour price="' + p.price +
                                '" id="' + p.id + '">' + p.name + '</tour>';
                }).join('') + '</tours>'';
        var toursText = tours.map(function(p){
                        return p.id + ': ' + p.name + ' (' + p.price + ')';
                }).join('\n');
        res.format({
                'application/json': function(){
                        res.json(tours);
                },
                'application/xml': function(){
                        res.type('application/xml');
                        res.send(toursXml);
                },
                'text/xml': function(){
                        res.type('text/xml');
                        res.send(toursXml);
                }
                'text/plain': function(){
                        res.type('text/plain');
                        res.send(toursXml);
                }
        });
});
```

Example 6-13. PUT endpoint for updating
``` javascript
// API that updates a tour and returns JSON; params are passed using querystring
app.put('/api/tour/:id', function(req, res){
        var p = tours.some(function(p){ return p.id == req.params.id });
        if( p ) {
                if( req.query.name ) p.name = req.query.name;
                if( req.query.price ) p.price = req.query.price;
                res.json({success: true});
        } else {
                res.json({error: 'No such tour exists.'});
        }
});
```

Example 6-14. DEL endpoint for deleting
``` javascript
// API that deletes a product
api.del('/api/tour/:id', function(req, res){
        var i;
        for( var i=tours.length-1; i>=0; i-- )
                if( tours[i].id == req.params.id ) break;
        if( i>=0 ) {
                tours.splice(i, 1);
                res.json({success: true});
        } else {
                res.json({error: 'No such tour exists.'});
        }
});
```

### CAP 7 - Templating with Handlebars

Si no haces uso de plantillas, es de las cosas más valiosas que vas a obtener de este libro. Es la manera en la que puedes escribir en el lenguaje destino mientras que sigues teniendo la habilidad de insertar datos de manera dinámica.
``` javascript
<h1>Much Better</h1>
<p>No <span class="code">document.write</span> here!</p>
<p>Today's date is {{today}}.</p>
```

**Escoger un motor de plantillas:** Algunos de los criterios que nos pueden ayudar a decidir entre uno y otro son el rendimiento, si queremos que se puedan utilizar tanto en cliente como en servidor o la abstracción a la hora de tener que trabajar con el HTML. El autor del libro elige **handlebars**.

Para entender el trabajo con plantillas hay que entender el **contexto**. Cuando renderizas una plantilla le pasas un objeto contexto que es lo que contiene la información para completar los datos a reemplazar. 

>Meter imagen WDNE-handlebars.png

Los comentarios van de la siguiente manera (los super-secretos no saldrán del servidor): 
```
{{! super-secret comment }}
<!-- not-so-secret comment -->
``` 

Podemos enviar contextos como el siguiente:
```
{
        currency: {
                name: 'United States dollars',
                abbrev: 'USD',
        },
        tours: [
                { name: 'Hood River', price: '$99.95' },
                { name: 'Oregon Coast', price, '$159.95' },
        ],
        specialsUrl: '/january-specials',
        currencies: [ 'USD', 'GBP', 'BTC' ],
}
```

Con ese contexto podemos crear código que se ejecute con **condicionales**, **en bucles** hasta mostrar todos los casos (para acceder a parámetros de fuera del caso en el que estamos darse cuenta que lo hace con **../** para volver a un nivel anterior), podemos acceder a **propiedades de objetos** si es lo que se nos pasa.
```
<ul>
        {{#each tours}}
                {{! I'm in a new block...and the context has changed }}
                <li>
                        {{name}} - {{price}}
                        {{#if ../currencies}}
                                ({{../../currency.abbrev}})
                        {{/if}}
                </li>
        {{/each}}
</ul>
{{#unless currencies}}
        <p>All prices in {{currency.name}}.</p>
{{/unless}}
{{#if specialsUrl}}
        {{! I'm in a new block...but the context hasn't changed (sortof) }}
        <p>Check out our <a href="{{specialsUrl}}">specials!</p>
{{else}}
        <p>Please check back often for specials.</p>
{{/if}}
<p>
        {{#each currencies}}
                <a href="#" class="currency">{{.}}</a>
        {{else}}
                Unfortunately, we currently only accept {{currency.name}}.
        {{/each}}
</p>
```

Tanto con if como con each se puede utilizar **else**, ejecutándose para each cuando no hay ningún caso disponible. También tenemos **unless** que es como if pero sólamente se ejecuta si el argumento es falso.

Por último con **{{.}}** nos referimos al contexto actual (por ejemplo para utilizarlo con each). Es útil cuando tengamos una función y una propiedad con el mismo nombre:

>Accessing the current context with a lone period has another use: it can distinguish helpers (which we’ll learn about soon) from properties of the current context. For example, if you have a helper called foo and a property in the current context called foo, {{foo}} refers to the helper, and {{./foo}} refers to the property.

##### Server-Side Templating
Además de ocultarle código al usuario podemos cachear plantillas ya generadas reduciendo el tiempo de entrega. Por defecto viene desactivado en Desarrollo y activado en Producción. Para activarlo explícitamente tendremos que poner: 
```
app.set('view cache', true);.
```

Por defecto no viene el soporte con Express por lo que tenemos que instalarlo:
```
npm install --save express3-handlebars
```

Y enlazarlo desde nuestro código:
```
var handlebars = require('express3-handlebars')
        .create({ defaultLayout: 'main' });
app.engine('handlebars', handlebars.engine);
app.set('view engine', 'handlebars');
```

**View:** Normalmente representa una página individual dentro de un sitio web. Express busca por defecto las vistas en el directorio *views*.

**Layout:** Es un tipo especial de vista. Es una plantilla para plantillas. Es la manera de hacer que todas las páginas de un mismo sitio se vean similar sin tener que repetir el mismo código en todas. Los elementos comunes (footer, header, ...) suelen estar en el layout. Un ejemplo es (utilizamos tres "{{{" para que interprete bien el código HTML que tendrán las vistas):

``` 
<!doctype>
<html>
<head>
        <title>Meadowlark Travel</title>
        <link rel="stylesheet" href="/css/main.css">
</head>
<body>
        {{{body}}}
</body>
</html>
```

El orden de renderizado es que primero se hace la parte de la vista (view) y luego la del layout con lo que se ha generado.

El layout que se utilizará se define al cargar el módulo de handlebars (*main.handlebars* en *views/layouts/*):
```
var handlebars = require('express3-handlebars')
        .create({ defaultLayout: 'main' });
```

Si quieres utilizar otro layout (para no utilizar ninguno pondríamos null) para una vista en concreto:
```
app.get('/foo', function(req, res){
        res.render('foo', { layout: 'microsite' });
});
```

Hay que buscar un equilibrio entre tener un único layout que contenga todo (lo más fácil de mantener) a tener distintos layouts para distintas páginas con pequeñas variaciones (más costoso de mantener).

**Partial:** Se utiliza cuando quieres generar un parte de un página y que pueda ser reutilizable entre páginas (también se denominan **widgets**). Por ejemplo se puede utilizar para mostrar un cuadrito con el tiempo que hace y ponerlo en una misma página con distintas ciudades del mundo. Para incluirlo en una vista tienes que poner (teniendo el fichero *partial_name.handlebars* en *views/partials/*):
```
{{> partial_name}}
```

Los partials permiten subdirectorios por lo que si tienes muchos los puedes separar por temáticas e incluir dicho directorio cuando lo llames desde una vista.

**Sections:** Gracias a los helpers podemos tener distintas secciones a modificar dentro de un layout:
```
var handlebars = require('express3-handlebars').create({
    defaultLayout:'main',
    helpers: {
        section: function(name, options){
            if(!this._sections) this._sections = {};
            this._sections[name] = options.fn(this);
            return null;
        }
    }
});
```

Ahora usamos el helper **section** en la vista:
```
{{#section 'head'}}
        <!-- we want Google to ignore this page -->
        <meta name="robots" content="noindex">
{{/section}}

<h1>Test Page</h1>
<p>We're testing some jQuery stuff.</p>

{{#section 'jquery'}}
        <script>
                $('document').ready(function(){
                        $('h1').html('jQuery Works');
                });
        </script>
{{/section}}
``` 

Y en nuestro layout ponemos esas secciones tal y como se hace con body:
```
<!doctype html>
<html>
<head>
        <title>Meadowlark Travel</title>
        {{{_sections.head}}}
</head>
<body>
        {{{body}}}
        <script src="http://code.jquery.com/jquery-2.0.2.min.js"></script>
        {{{_sections.jquery}}}
</body>
</html>
```

##### Client-Side Templating
Útil cuando quieres tener contenido dinámico. Es especialmente útil cuando nos vamos a comunicar con APIs de terceros y nos devuelvan los datos con JSON. 



### CAP 8 -	Form Handling

En este capítulo vemos la manera de manejar formularios, validarlos y subir ficheros. 

Hay dos maneras de mandar información desde el usuario al servidor: GET y POST. Con GET iría en la URL y con POST va en el cuerpo de la respuesta. Las dos son iguales de seguras si se usa HTTPS y de inseguras si se utiliza HTTP, lo que pasa es que con GET el usuario puede ver toda la información en la querystring de la URL. Además el tamaño de la URL tiene un límite por lo que es recomendable utilizar POST para enviar formularios. 

Un ejemplo sencillo de formulario sería:
```
<form action="/process" method="POST">
    <input type="hidden" name="hush" val="hidden, but not secret!">
    <div>
        <label for="fieldColor">Your favorite color: </label>
        <input type="text" id="fieldColor" name="color">
    </div>
    <div>
        <button type="submit">Submit</button>
    </div>
</form>
```

El método (**method**) que viene indicado es POST. Si no indicas nada el por defecto es GET. El atributo **action** indica la URL a la que se mandará el formulario, y si no viene indicado será la URL desde la que se cargó (recomendable siempre poner un valor). El campo **name** es la manera en que identifica a los campos el servidor.

Es posible tener dos formularios en una misma página (por ejemplo uno para hacer búsquedas y otro para hacer login). 

Si no estás utilizando AJAX, suscribirás el formulario a través del navegador y la web se refrescará, estando determinado el comportamiento por el path de respuesta y por el contenido de la respuesta. Si mandas la información con POST puedes utilizar la misma URL ya que al utilizar distinto método que cuando recuperaste la información de la página puedes hacer cosas diferentes. Otra opción es utilizar un path diferente (por ejemplo /contact pasa a ser /process-contact). La segunda opción es mejor si tienes la opción de mandar ese formulario desde distintas URLs.

Para contestar al cliente con la información que haya pasado del formulario lo mejor es utilizar un **303 redirect**. Opciones de redirección:

* **Página de éxito/fracaso:** En función de si ha salido bien el tratamiento de los datos se manda a una página u otra. Viene bien porque en función de las peticiones que recibimos en cada página podemos saber el funcionamiento del servidor pero lo malo es que supone mantener todas esas páginas y que el usuario tendrá que volver atrás para estar donde estaban o ir a otro lado.
* **Mismo sitio con un mensaje:** Mandar a la misma web en que te encuentras pero poner un mensaje flash indicando al usuario que se ha recibido la información. La mejor opción es tener un campo oculto en el formulario que utilicemos para remitir la URL en la que nos encontramos y que así nos lleve allí otra vez.
* **Distinto sitio con un mensaje:** Cuando son formularios grandes generalmente no quieres volver al mismo sitio en el que te encontrabas. Por ejemplo cuando rellenas la información de un elemento lo normal es que cuando suscribas esa información la página a la que se te envíe sea en la que se listan todos los elementos y salga un mensaje que te indique que se ha registrado la información.


##### Manejar formularios con Express

Si utilizas **GET** tendrás los campos del formulario en el objeto **req.query**.

Si utilizas **POST** tendrás que utilizar un middleware para parsear la información enviada. Utilizamos el módulo **body-parser** (npm install --save body-parser):
```
app.use(require('body-parser')());
```

Tras ello podremos hacer uso de **req.body** para acceder a toda la información. Vamos a hacer un ejemplo para que el usuario se añada a una lista de emails. Contenido de */views/newsletter.handlebars*:
```
<h2>Sign up for our newsletter to receive news and specials!</h2>
<form class="form-horizontal" role="form"
        action="/process?form=newsletter" method="POST">
    <input type="hidden" name="_csrf" value="{{csrf}}">
    <div class="form-group">
        <label for="fieldName" class="col-sm-2 control-label">Name</label>
        <div class="col-sm-4">
            <input type="text" class="form-control"
            id="fieldName" name="name">
        </div>
    </div>
    <div class="form-group">
        <label for="fieldEmail" class="col-sm-2 control-label">Email</label>
        <div class="col-sm-4">
            <input type="email" class="form-control" required
                id="fieldName" name="email">
        </div>
    </div>
    <div class="form-group">
        <div class="col-sm-offset-2 col-sm-4">
            <button type="submit" class="btn btn-default">Register</button>
        </div>
    </div>
</form>
```

Y el código para poder acceder a los datos del usuario desde el servidor:
```
app.use(require('body-parser')());

app.get('/newsletter', function(req, res){
    // we will learn about CSRF later...for now, we just
    // provide a dummy value
    res.render('newsletter', { csrf: 'CSRF token goes here' });
});

app.post('/process', function(req, res){
    console.log('Form (from querystring): ' + req.query.form);
    console.log('CSRF token (from hidden form field): ' + req.body._csrf);
    console.log('Name (from visible form field): ' + req.body.name);
    console.log('Email (from visible form field): ' + req.body.email);
    res.redirect(303, '/thank-you');
});
```

Es importante utilizar un 303 en la respuesta (y no un 301) para que redirija pero no cachee el navegador esa redirección y tenga que preguntar en siguientes ejecuciones al servidor. 


##### Manejar formularios AJAX

Podemos manejar muy fácilmente información enviada con AJAX. HTML:
```
<div class="formContainer">
    <form class="form-horizontal newsletterForm" role="form"
            action="/process?form=newsletter" method="POST">
        <input type="hidden" name="_csrf" value="{{csrf}}">
        <div class="form-group">
            <label for="fieldName" class="col-sm-2 control-label">Name</label>
            <div class="col-sm-4">
                <input type="text" class="form-control"
                id="fieldName" name="name">
            </div>
        </div>
        <div class="form-group">
            <label for="fieldEmail" class="col-sm-2 control-label">Email</label>
            <div class="col-sm-4">
                <input type="email" class="form-control" required
                    id="fieldName" name="email">
            </div>
        </div>
        <div class="form-group">
            <div class="col-sm-offset-2 col-sm-4">
                <button type="submit" class="btn btn-default">Register</button>
            </div>
        </div>
    </form>
</div>
{{#section 'jquery'}}
    <script>
        $(document).ready(function(){
            $('.newsletterForm').on('submit', function(evt){
                evt.preventDefault();
                var action = $(this).attr('action');
                var $container = $(this).closest('.formContainer');
                $.ajax({
                    url: action,
                    type: 'POST',
                    success: function(data){
                        if(data.success){
                            $container.html('<h2>Thank you!</h2>');
                        } else {
                            $container.html('There was a problem.');
                        }
                    },
                    error: function(){
                        $container.html('There was a problem.');
                    }
                });
            });
        });
    </script>
{{/section}}
```

Y el código de la aplicación:
```
app.post('/process', function(req, res){
    if(req.xhr || req.accepts('json,html')==='json'){
        // if there were an error, we would send { error: 'error description' }
        res.send({ success: true });
    } else {
        // if there were an error, we would redirect to an error page
        res.redirect(303, '/thank-you');
    }
});
```


##### Subida de ficheros

Para manejar esas subidas utilizaremos el middleware **formidable** (npm install --save formidable), que nos permitirá acceder a los ficheros. Primero el HTML con el formulario:
```
<form class="form-horizontal" role="form"
        enctype="multipart/form-data" method="POST"
        action="/contest/vacation-photo/{year}/{month}">
    <div class="form-group">
        <label for="fieldName" class="col-sm-2 control-label">Name</label>
        <div class="col-sm-4">
            <input type="text" class="form-control"
            id="fieldName" name="name">
        </div>
    </div>
    <div class="form-group">
        <label for="fieldEmail" class="col-sm-2 control-label">Email</label>
        <div class="col-sm-4">
            <input type="email" class="form-control" required
                id="fieldName" name="email">
        </div>
    </div>
    <div class="form-group">
        <label for="fieldPhoto" class="col-sm-2 control-label">Vacation photo
        </label>
        <div class="col-sm-4">
            <input type="file" class="form-control" required accept="image/*"
                id="fieldPhoto" name="photo">
        </div>
    </div>
    <div class="form-group">
        <div class="col-sm-offset-2 col-sm-4">
            <button type="submit" class="btn btn-primary">Submit</button>
        </div>
    </div>
</form>
```

Hay que indicar enctype="multipart/form-data" y también restringimos los tipos de fichero a subir con el atributo accept. Y luego el código de la aplicación:
```
var formidable = require('formidable');

app.get('/contest/vacation-photo',function(req,res){
    var now = new Date();
    res.render('contest/vacation-photo',{
        year: now.getFullYear(),month: now.getMont()
    });
});

app.post('/contest/vacation-photo/:year/:month', function(req, res){
    var form = new formidable.IncomingForm();
    form.parse(req, function(err, fields, files){
        if(err) return res.redirect(303, '/error');
        console.log('received fields:');
        console.log(fields);
        console.log('received files:');
        console.log(files);
        res.redirect(303, '/thank-you');
    });
});
```

**Subida de ficheros con jQUERY**
Si quieres ofrecer una experiencia de usuario a la hora de subir ficheros que permita drag&drop, ver miniaturas de los ficheros subidos, saber el porcentaje completado... recomienda utilizar la solución de la [siguiente URL](http://blueimp.github.io/jQuery-File-Upload/).


### CAP 9 - Cookies and Sessions

HTTP es un protocolo que no contiene estado. Si se quedase ahí significaría que no se podría hacer login en las páginas, que no se podría hacer streaming... Para que sea posible entran a escena las **cookies** y las **sesiones**.

##### Cookies

Tienen mala fama debido al uso que se ha dado de ellas pero son imprescindibles para la web moderna (aunque local storage de HTML5 puede cubrir parte de su funcionalidad).

Su idea es simple: el servidor envía información y el navegador la almacena por un periodo de tiempo configurable. Hay varios puntos clave que conocer:

* **Cookies are note secret from the user:** Todas las cookies que envías están disponibles para observar por el cliente. Se pueden firmar pero no se pueden encriptar (ni tiene sentido).
* **The user can delete or disallow cookies:** Más allá de estar realizando pruebas no es normal que haga uso de ello el usuario.
* **Regular cookies can be tampered with:** Recomendable utilizar cookies firmadas para evitar ataques.
* **Cookies can be used for attacks:** Los hackers utilizan las cookies de vuelta al servidor para realizar ataques XSS. Para hacerlas más seguras se puede indicar que sólo puedan ser modificadas por el servidor y que vayan firmadas.
* **Users will notice if you abuse cookies:** Hay que tratar de mantenerlas al mínimo.
* **Prefer sessions over cookies:** Para casi todo puedes utilizar sesiones para mantener el estado y olvidarte de practicamente todos estos problemas. Se basan en las cookies pero con las sesiones Express hará todo el trabajo pesado.

**Credenciales**
Para hacer las cookies seguras hay que asignarles una clave con las que las firmaremos. Puede ser un string random, pero hay que almacenarlo. Para ello crearemos un fichero *credentials.js*, que contendrá esta y otra información de acceso, que mantendremos fuera de nuestro repositorio (incluyéndolo dentro de *.gitignore*):
```
module.exports = {
    cookieSecret: 'your cookie secret goes here',
};
```

Luego cargaremos esa clave desde nuestra aplicación:
```
var credentials = require('./credentials.js');
```

**Cookies en Express**
Necesitamos el middleware correspondiente **cookie-parse** (npm install --save cookie-parser). Para incluirlo:
```
app.use(require('cookie-parser')(credentials.cookieSecret));
``` 

Y luego podemos crear cookies firmadas o sin firmar:
```
res.cookie('monster', 'nom nom');
res.cookie('signed_monster', 'nom nom', { signed: true });
``` 

Para recuperar información de la cookie devuelta por el usuario accedemos a las propiedades del objeto request:
```
var monster = req.cookies.monster;
var signedMonster = req.signedCookies.monster;
``` 

Y para borrarla:
```
res.clearCookie('monster');
``` 

**Opciones** que se pueden especificar al crearla: **domain** (dominio con el que se asocia, no se puede indicar uno distinto al que corre el servidor), **path** (path del sitio en el que se aplica), **maxAge** (tiempo de vida de la cookie en el cliente en ms), **secure** (sólo se puede enviar por https), **httpOnly** (sólo puede ser modificada por el servidor para prevenir XSS) y **signed** (se firmará).


##### Sessions

Son una manera más conveniente de mantener el estado. Se almacena información un poco de información en el cliente y se manda un identificador al servidor para que este pueda reconocer al usuario y mande la información que le corresponda. 

Si quieres guardar datos de sesión en el servidor tienes que elegir como guardarla. El nivel básico es almacenarla en memoria, que es muy fácil de configurar pero tiene como pega que se pierde la información al reiniciar el servidor. Tampoco es una solución que sirva cuando tiene múltiples servidores, ya que depende de en que servidor caigas tendrá la información del usuario o no la tendrá. (En el cap13 veremos como almacenarla permanentemente).

Necesitamos el módulo **express-session** (npm install --save express-session), y luego lo referenciamos tras llamar a cookie-parser:
```
app.use(require('cookie-parser')(credentials.cookieSecret));
app.use(require('express-session')());
```

Este middleware tiene las siguientes opciones: **key** (el nombre de la cookie que almacenará el identificador de sesión), **store** (donde se almacena la información, siendo por defecto MemoryStore) y **cookie** (los parámetros que vimos en el apartado anterior concernientes a las cookies).

**Para usar las propiedades de la sesión** lo hacemos todo bajo el objeto **request** y sus atributos:
```
req.session.userName = 'Anonymous';
var colorScheme = req.session.colorScheme || 'dark';
```

Y para borrar información y eliminar la sesión:
```
req.session.userName = null;	// this sets 'userName' to null, but doesn't remove it
delete req.session.colorScheme;	// this removes 'colorScheme'
```

> Al final junta esto con los flash messages en función de la información de la sesión. Ver en casa el ejemplo práctico y sacarlo en claro viendo el funcionamiento.


### CAP 10 - Middleware

Middleware es encapsular una funcionalidad que se ejecutará cuando llegue una petición a tu aplicación. Se trata de una función con tres argumentos: request object, response object y función next. Hay otra forma que tiene un cuarto argumento y entonces el primero pasa a ser para tratar los errores.

El middleware es como si fuera una tubería en el que unas cosas se van ejecutando tras otras. La parte importante de la analogía es que **el orden importa**. Insertas el middleware haciendo uso de **app.use**.

Se pueden unir los **router** y los **use**. A partir de Express 4.0 el orden en que se invocan cuando están los dos relacionados se mantiene, haciendo más fácil entender el funcionamiento de la secuencia.

Es típico acabar la cadena de middleware con un handler que se encargue de coger cualquier tipo de respuesta para acabar devolviendo un 404 en caso de no haber podido manejar previamente la petición. Se pasa de un middleware a otro con la función **next()**, terminando en el que nos encontramos si no se llama a la función. En ese momento se le deberá enviar una respuesta al cliente (res.send, res.json, res.render...), sino se colgará la conexión. Si hubiese una respuesta en un middleware que contiene next() no se tendrán en cuenta las siguientes respuestas que continúen la cadena. 

Los route handlers necesitan que se les pase el path como primer parámetro (si es para cualquiera se indica con **/\***), mientras que con el middleware es opcional (si es omitido aceptará cualquier path). Los route handlers pueden ser considerados como middleware que maneja HTTP verbs (app.get, app.post...) mientras que el Middleware puede ser considerado como un route handler que maneja todos los HTTP verbs. 

Como ejemplo:
```
var app = require('express')();

app.use(function(req, res, next){
        console.log('\n\nALLWAYS');
        next();
});

app.get('/a', function(req, res){
        console.log('/a: route terminated');
        res.send('a');
});
app.get('/a', function(req, res){
        console.log('/a: never called');
});
app.get('/b', function(req, res, next){
        console.log('/b: route not terminated');
        next();
});
app.use(function(req, res, next){
        console.log('SOMETIMES');
        next();
});
app.get('/b', function(req, res, next){
        console.log('/b (part 2): error thrown' );
        throw new Error('b failed');
});
app.use('/b', function(err, req, res, next){
        console.log('/b error detected and passed on');
        next(err);
});
app.get('/c', function(err, req){
        console.log('/c: error thrown');
        throw new Error('c failed');
});
app.use('/c', function(err, req, res, next){
        console.log('/c: error deteccted but not passed on');
        next();
});

app.use(function(err, req, res, next){
        console.log('unhandled error detected: ' + err.message);
        res.send('500 - server error');
});

app.use(function(req, res){
        console.log('route not handled');
        res.send('404 - not found');
});

app.listen(3000, function(){
        console.log('listening on 3000');
});
```

Para hacer uso de un middleware puede ser una función directamente exportada. El módulo (*lib/tourRequiresWaiver.js*):
```
module.exports = function(req,res,next){
        var cart = req.session.cart;
        if(!cart) return next();
        if(cart.some(function(item){ return item.product.requiresWaiver; })){
                if(!cart.warnings) cart.warnings = [];
                cart.warnings.push('One or more of your selected tours' +
                        'requires a waiver.');
        }
        next();
}
```

Y hacemos uso de él (también puede ser que llamemos a los atributos que tenga un módulo, que es lo que haremos con muchos módulos externos):
```
app.use(require('./lib/requiresWaiver.js'));
```

##### Middleware común
Por como estuvieron unidos en el tiempo siempre se recomienda instalarlo de serie (npm install --save connect) y tenerlo disponible (var connect = require(connect);). 

Los middleware más comunes de utilizar son:

* **basicAuth** (contenido dentro de connect): Autenticación muy básica. Solo para hacer proyectos rápidamente y que funcionen bajo HTTPS.
* **body-parser**: Para parsear respuestas que nos vengan con información bien JSON o urlencoded (media type application/x-www-form-urlencoded).
* **compress**: Para comprimir la respuesta en gzip. Hay que linkarlo lo más pronto posible, antes de cualquier middleware que pueda mandar una respuesta.
* **cookie-parser**: Como vimos en el cap9 para soportar cookies.
* **cookie-session**: Mantener sesiones mediante almacenamiento en cookie. Cap9.
* **express-session**: ID de sesión mantenido en cookie, el resto en memoria, BBDD...
* **csurf**: Protección contra ataques CSRF (cross-site request forgery). Utiliza sesiones.
* **directory**: Listado de directorios para ficheros estáticos.
* **errorHandler**: Mensajes de error para el cliente. No usar en Producción ya que compromete la información del servidor. En Cap20.
* **static-favicon**: Para mejorar el servir el icono que aparece en la barra de título del navegador.
* **morgan**: Para logs de peticiones. En Cap20.
* **method-override**: Soporte para la cabecera de peticiones x-http-method-override.
* **query**: Parsea la querystring y la deja como la propiedad query en el objeto request. Viene linkado por defecto en Express.
* **response-time**: Añade la cabecera X-Response-Time a la respuesta, indicando el tiempo de respuesta en ms. Útil para tuning.
* **static**: Para servir estáticos (public). Se puede linkar varias veces para distintos directorios.
* **vhost**: Virtual Hosts. Para hacer más fácil la gestión de subdominios en Express.

**No hay una tienda expecífica para buscar middleware**. La mayor parte se puede encontrar en npm buscando por "Express", "Connect" y "Middleware".



### CAP 11 - Sending Email

Una forma de comunicarse desde tu sitio web con el mundo es el email. Password de acceso, emails promocionales. No viene por defecto ningún módulo pero recomienda instalar **nodemailer**.

Un email contiene cabecera y cuerpo. Al enviarlo debe tener la cabecera **from** informada. Hay que indicar que la dirección que aparece no responderá los correos (si es que nadie va a estar leyendo el correo que pueda entrar, como por ejemplo se indica con do-not-reply@...). El **formato** puede ser texto plano o HTML, y nosotros a través de nodemailer enviaremos los dos juntos. El HTML que permiten los correos está muy limitado, es cercano a lo que había en 1996.

[Artículo](https://kb.mailchimp.com/es/campaigns/ways-to-build/about-html-email) sobre escribir emails HTML. Para ahorrar tiempo muy recomendable [HTML Boilerplate](https://github.com/seanpowell/Email-Boilerplate). 

Para instalar el módulo (npm install --save nodemailer). Lo cargamos:
```
var nodemailer = require('nodemailer');

var mailTransport = nodemailer.createTransport('SMTP',{
        service: 'Gmail',
        auth: {
                user: credentials.gmail.user,
                pass: credentials.gmail.password,
        }
});
```

Y luego completamos el fichero de credenciales (*credentials.js*) con la nueva parte:
```
module.exports = {
        cookieSecret: 'your cookie secret goes here',
        gmail: {
                user: 'your gmail username',
                password: 'your gmail password',
        }
};
```

Para **mandar un email** a varios destinatarios (por ejemplo GMAIL limita a 100 destinatarios por correo, y tiene un límite de 500 correos por día, por lo que lo hacemos que nunca envíe un correo a más de 100 personas):
```
// largeRecipientList is an array of email addresses
var recipientLimit = 100;
for(var i=0; i<largeRecipientList.length/recipientLimit; i++){
        mailTransport.sendMail({
                from: '"Meadowlark Travel" <info@meadowlarktravel.com>',
                to: largeRecipientList
                        .slice(i*recipientLimit, i*(recipientLimit+1)).join(','),
                subject: 'Special price on Hood River travel package!',
                text: 'Book your trip to scenic Hood River now!',
        }, function(err){
                if(err) console.error( 'Unable to send email: ' + error );
        });
}
```

Para mandar **mails promocionales o newsletters** lo mejor es utilizar servicios como **Mailchimp** o **Campaign monitor**.

Nodemailer nos traducirá automáticamente a texto plano: 
```
mailTransport.sendMail({
        from: '"Meadowlark Travel" <info@meadowlarktravel.com>',
        to: 'joecustomer@gmail.com, "Jane Customer" ' +
                '<janecustomer@gyahoo.com>, frecsutomer@hotmail.com',
        subject: 'Your Meadowlark Travel Tour',
        html: '<h1>Meadowlark Travel</h1>\n<p>Thanks for book your trip with ' +
                'Meadowlark Travel.  <b>We look forward to your visit!</b>',
        generateTextFromHtml: true,
}, function(err){
        if(err) console.error( 'Unable to send email: ' + error );
});
```

Para incluir imágenes en tus emails la mejor práctica es sólo incluir links que apunten a imágenes contenidas en tu servidor web. Para ello crearemos un directorio *email* en nuestra instalación y así meter ahí todos los ficheros que necesitemos. En el Cap16 nos enseñará como apuntar a esos directorios sin tener que utilizar el localhost (no tiene sentido al no recibir el email en el PC en el que estás desarrollando la aplicación). 

Para no tener que poner código HTML en nuestro javascript vamos a crear vistas para nuestros correos y sobre ellos poner información concerniente al cliente que lo va a recibir (a partir de HTML Boilerplate creamos *views/email/cart-thank-you.handlebars*):
```
<body>
<table cellpadding="0" cellspacing="0" border="0" id="backgroundTable">
    <tr>
        <td valign="top">
            <table cellpadding="0" cellspacing="0" border="0" align="center">
                <tr>
                    <td width="200" valign="top"><img class="image_fix"
                        src="http://meadowlarktravel.com/email/logo.png"
                        alt="Meadowlark Travel" title="Meadowlark Travel"
                        width="180" height="220" /></td>
                </tr>
                <tr>
                    <td width="200" valign="top"><p>
                        Thank you for booking your trip with Meadowlark Travel,
                        {{cart.billing.name}}.</p><p>Your reservation number
                        is {{cart.number}}.</p></td>
                </tr>
                <tr>
                    <td width="200" valign="top">Problems with your reservation?
                    Contact Meadowlark Travel at
                    <span class="mobile_link">555-555-0123</span>.</td>
                </tr>
            </table>
        </td>
    </tr>
</table>
</body>
```

Como no tendremos acceso a nuestro servidor al estar en pruebas lo mejor es utilizar servicios como el de [placehold](http://placehold.it/100x100) para hacer las pruebas con imágenes del tamaño que necesitamos. 

Creamos la ruta en la aplicación, que tendrá dos render. El primero se ejecutará la primera vez para mandar el mail y el segundo ya el resto de veces para mostrar la página.
```
app.post('/cart/checkout', function(req, res){
        var cart = req.session.cart;
        if(!cart) next(new Error('Cart does not exist.'));
        var name = req.body.name || '', email = req.body.email || '';
        // input validation
        if(!email.match(VALID_EMAIL_REGEX))
                return res.next(new Error('Invalid email address.'));
        // assign a random cart ID; normally we would use a database ID here
        cart.number = Math.random().toString().replace(/^0\.0*/, '');
        cart.billing = {
                name: name,
                email: email,
        };
    res.render('email/cart-thank-you',
        { layout: null, cart: cart }, function(err,html){
                if( err ) console.log('error in email template');
                mailTransport.sendMail({
                    from: '"Meadowlark Travel": info@meadowlarktravel.com',
                    to: cart.billing.email,
                    subject: 'Thank You for Book your Trip with Meadowlark',
                    html: html,
                    generateTextFromHtml: true
                }, function(err){
                        if(err) console.error('Unable to send confirmation: '
                                + err.stack);
                });
            }
    );
    res.render('cart-thank-you', { cart: cart });
});
```

Podemos encapsular toda la lógica de envío de correo en un módulo(*lib/email.js*):
```
var nodemailer = require('nodemailer');

module.exports = function(credentials){

        var mailTransport = nodemailer.createTransport('SMTP',{
                service: 'Gmail',
                auth: {
                        user: credentials.gmail.user,
                        pass: credentials.gmail.password,
                }
        });

        var from = '"Meadowlark Travel" <info@meadowlarktravel.com>';
        var errorRecipient = 'youremail@gmail.com';

        return {
                send: function(to, subj, body){
                    mailTransport.sendMail({
                        from: from,
                        to: to,
                        subject: subj,
                        html: body,
                        generateTextFromHtml: true
                    }, function(err){
                        if(err) console.error('Unable to send email: ' + err);
                    });
                }),

                emailError: function(message, filename, exception){
                        var body = '<h1>Meadowlark Travel Site Error</h1>' +
                                'message:<br><pre>' + message + '</pre><br>';
                        if(exception) body += 'exception:<br><pre>' + exception
                                + '</pre><br>';
                        if(filename) body += 'filename:<br><pre>' + filename
                                + '</pre><br>';
                    mailTransport.sendMail({
                        from: from,
                        to: errorRecipient,
                        subject: 'Meadowlark Travel Site Error',
                        html: body,
                        generateTextFromHtml: true
                    }, function(err){
                        if(err) console.error('Unable to send email: ' + err);
                    });
                },
}
```

Y ya lo llamaremos simplemente de la siguiente manera:
```
var emailService = require('./lib/email.js')(credentials);

emailService.send('joecustomer@gmail.com', 'Hood River tours on sale today!',
        'Get \'em while they\'re hot!');
```


### CAP 12 - Production Concerns

Aunque puede ser pronto para hablar de Producción cuanto antes se traten ciertos temas más trabajo se puede ahorrar a futuro. 

Express soporta **distintos entornos de trabajo** (Producción, Desarrollo, Test...). Lo mejor es indicar dicho entorno con la variable global **NODE_ENV**:
``` 
http.createServer(app).listen(app.get('port'), function(){
    console.log( 'Express started in ' + app.get('env') +
        ' mode on http://localhost:' + app.get('port') +
        '; press Ctrl-C to terminate.' );
});
``` 

Cambiaremos el entorno desde el sistema exportando el valor que se necesite (si pones producción habrá Warnings indicándote de cosas que no son recomendables para Producción como puede ser el almacenar las sesiones en Memoria en vez de en BBDD):
```
export NODE_ENV=production
```

Por ejemplo podemos **escribir en logs de distinta manera para cada entorno**. Vamos a poner en Desarrollo log en colores y en Producción logs que roten cada 24 horas:
```
switch(app.get('env')){
    case 'development':
        // compact, colorful dev logging
        app.use(require('morgan')('dev'));
        break;
    case 'production':
        // module 'express-logger' supports daily log rotation
        app.use(require('express-logger')({
            path: __dirname + '/log/requests.log'
        }));
        break;
}
```

##### Escalar la aplicación
Hay distintas maneras de escalar: scaling up (hacer el servidor más potente) y scaling out (aumentar el número de servidores). La mejor opción suele ser **scaling out**. Para poder utilizar esa opción tiene que haber almacenamiento compartido entre instancias (BBDD, filesystem compartido...).

Para hacer que se ejecute en varias CPUs necesitamos un master y los worker posibles hasta gastar las CPUs que tenga la máquina. Primero le hacemos una pequeña modificación a *meadowlark.js*:
```
function startServer() {
    http.createServer(app).listen(app.get('port'), function(){
      console.log( 'Express started in ' + app.get('env') +
        ' mode on http://localhost:' + app.get('port') +
        '; press Ctrl-C to terminate.' );
    });
}

if(require.main === module){
    // application run directly; start app server
    startServer();
} else {
    // application imported as a module via "require": export function
    // to create server
    module.exports = startServer;
}
```

Si ha sido llamado directamente entrará en **require.main === module**. En caso de haber sido llamado utilizando require entrará en la otra parte. Ahora creamos el script *meadowlark_cluster.js*, que se encargará de levantar los worker necesarios y volver a levantar aquellos que se caigan:
```
var cluster = require('cluster');

function startWorker() {
    var worker = cluster.fork();
    console.log('CLUSTER: Worker %d started', worker.id);
}

if(cluster.isMaster){

    require('os').cpus().forEach(function(){
            startWorker();
    });

    // log any workers that disconnect; if a worker disconnects, it
    // should then exit, so we'll wait for the exit event to spawn
    // a new worker to replace it
    cluster.on('disconnect', function(worker){
        console.log('CLUSTER: Worker %d disconnected from the cluster.',
            worker.id);
    });

    // when a worker dies (exits), create a worker to replace it
    cluster.on('exit', function(worker, code, signal){
        console.log('CLUSTER: Worker %d died with exit code %d (%s)',
            worker.id, code, signal);
        startWorker();
    });

} else {

    // start our app on worker; see meadowlark.js
    require('./meadowlark.js')();

}
```

Si quieres ver como está trabajando cada worker puedes poner lo siguiente antes de recibir peticiones:
```
app.use(function(req,res,next){
    var cluster = require('cluster');
    if(cluster.isWorker) console.log('Worker %d received request',
        cluster.worker.id);
});
```

##### Manejo de excepciones
Es bueno siempre tener una página de errores propia (*views/500.handlebars*) para dar una imagen más profesional hacia los usuarios, además de que se puede hacer cierta monitorización de cuando eso ocurre para que se entere el equipo de desarrollo. Por ejemplo:
```
app.get('/fail', function(req, res){
    throw new Error('Nope!');
});

app.use(function(err, req, res, next){
    console.error(err.stack);
    app.status(500).render('500');
});
```

Si nos encontramos con una excepción en una llamada asíncrona lo que ocurrirá es que el servidor se caerá, como por ejemplo en el siguiente código:
```
app.get('/epic-fail', function(req, res){
    process.nextTick(function(){
        throw new Error('Kaboom!');
    });
});
```

Para manejar ese tipo de situaciones lo mejor es tener un cluster en el que el master revise si los hijos están bien (como lo puesto más arriba) y que se encargue de levantar un nuevo worker en caso de que se haya caído alguno. Con tener el master y un sólo hijo ya es suficiente para poder mantener ese tipo de arquitectura, aunque el tiempo en volver a dar servicio con las mismas condiciones será mayor. 

##### Apagar ante excepciones no controladas
Lo haremos utilizando **dominios**. Se trata de un contexto de ejecución que capturará los errores que ocurran en su interior. Permiten ser más flexibles en el manejo de errores por lo que son útiles en zonas de código propensas a generar errores. Un buena praxis sería procesar cada petición en un dominio y así pudiendo coger cualquier excepción que nos sea devuelta. Lo podemos conseguir añadiendo un middleware antes de las rutas o de otro middleware (si vamos a tener peticiones que se vayan mucho de tiempo podemos aumentar el tiempo a más de 5 segundos):
```
app.use(function(req, res, next){
    // create a domain for this request
    var domain = require('domain').create();
    // handle errors on this domain
    domain.on('error', function(err){
        console.error('DOMAIN ERROR CAUGHT\n', err.stack);
        try {
            // failsafe shutdown in 5 seconds
            setTimeout(function(){
                console.error('Failsafe shutdown.');
                process.exit(1);
            }, 5000);

            // disconnect from the cluster
            var worker = require('cluster').worker;
            if(worker) worker.disconnect();

            // stop taking new requests
            server.close();

            try {
                // attempt to use Express error route
                next(err);
            } catch(err){
                // if Express error route failed, try
                // plain Node response
                console.error('Express error mechanism failed.\n', err.stack);
                res.statusCode = 500;
                res.setHeader('content-type', 'text/plain');
                res.end('Server error.');
            }
        } catch(err){
            console.error('Unable to send 500 response.\n', err.stack);
        }
    });

    // add the request and response objects to the domain
    domain.add(req);
    domain.add(res);

    // execute the rest of the request chain in the domain
    domain.run(next);
});

// other middleware and routes go here

var server = http.createServer(app).listen(app.get('port'), function(){
    console.log('Listening on port %d.', app.get('port'));
});
```

El [siguiente artículo](http://bit.ly/100_percent_uptime) da ideas de como conseguir mantener un servidor node levantado al 100%.


##### Escalar a través de varios servidores
Para poder servir peticiones a distintos servidores tendremos que usar un proxy inverso que reparta las conexiones. Para ello es recomendable utilizar **nginx**. Para poder utilizar un proxy hay que indicarselo a Express:
``` 
app.enable('trust proxy');
```


##### Monitorizar la aplicación
Evita situaciones de indisponibilidad. Una de las acciones que se pueden realizar es lanzar peticiones automáticas desde un agente externo para comprobar que sigue respondiendo, y para ello podemos utilizar **UptimeRobot**. Desde una funcionalidad que falle se pueden mandar mails para indicar que no está funcionando adecuadamente (esto se vio en el tema anterior). También podemos realizar pruebas de estrés sobre la aplicación que podemos hacer con el módulo node **loadtest**.



### CAP 13 - Persistence

Todas las aplicaciones por sencillas que sean necesitan guardar algo de información de manera persistente. En el libro van a presentar varias opciones pero centrándose en las BBDD de documentos. 

##### Filesystem persistence
A través de ficheros planos almacenados en el filesystem. Se hace a través del módulo **fs**. No escala bien con el crecimiento de servidores a menos que tengas un almacenamiento compartido en red. Además al no tener estructura interna se hace difícil la búsqueda de datos en su interior. Por ello es preferible el uso de BBDD aunque este tipo de almacenamiento es muy útil para almacenar archivos binarios (imágenes, ficheros, vídeos...).
Hay que tener especial cuidado con lo que suben los usuarios al servidor ya que podrían meter ficheros corruptos o bien nombres del fichero que comprometan el código. En el libro viene un ejemplo para subir una foto al servidor.


##### Cloud persistence
Subir un fichero a un servicio como puede ser AWS o Azure.


##### Database persistence
Están las BBDD **relacionales** y las **NoSQL**. Aunque puede ser fácil conectar una BBDD relacional a node, las NoSQL son extremadamente sencillas.

Las **más famosas** son las BBDD basadas en **documentos** y las de **clave-valor**.

Las BBDD de documentos ofrecen un buen compromiso entre las constraints de las BBDD relacionales y la simplicidad de las BBDD de clave-valor. Nos centramos en **mongoDB** que es el líder en ese tipo.

Para profundizar en mongoDB y su tunning recomienda el libro MongoDB: The Definitive Guide (O’Reilly).

Hay servicios Online para alojar BBDD mongo. Tienen servicios gratuitos que están bien para hacer pruebas. Por ejemplo están **mongoLab** y **mongoHG**.

Para conectarnos desde node lo haremos utilizando un *ODM-Object Document Mapper* como es **mongoose**. Introduce schemas y models, que combinados hacen una función similar a las clases de los POO. Son flexibles pero dan una estructura necesaria para trabajar con la BBDD.

Instalamos:
```
npm install --save mongoose
```

E indicamos la conexión a nuestros entornos:
```
mongo: {
    development: {
        connectionString: 'your_dev_connection_string',
    },
    production: {
        connectionString: 'your_production_connection_string',
    },
},
```

Para luego conectarnos (keepalive nos sirve para prevenir errores de conexión):
```
var mongoose = require('mongoose');
var opts = {
    server: {
       socketOptions: { keepAlive: 1 }
    }
};
switch(app.get('env')){
    case 'development':
        mongoose.connect(credentials.mongo.development.connectionString, opts);
        break;
    case 'production':
        mongoose.connect(credentials.mongo.production.connectionString, opts);
        break;
    default:
        throw new Error('Unknown execution environment: ' + app.get('env'));
}
```

Para crear los schemas/models lo hacemos generando por ejemploel fichero *models/vacation.js*. Contiene propiedades y cada una tiene su tipo (strings, númericos, array de strings, booleanos). También contiene métodos.
```
var mongoose = require('mongoose');

var vacationSchema = mongoose.Schema({
    name: String,
    slug: String,
    category: String,
    sku: String,
    description: String,
    priceInCents: Number,
    tags: [String],
    inSeason: Boolean,
    available: Boolean,
    requiresWaiver: Boolean,
    maximumGuests: Number,
    notes: String,
    packagesSold: Number,
});
vacationSchema.methods.getDisplayPrice = function(){
    return '$' + (this.priceInCents / 100).toFixed(2);
};
var Vacation = mongoose.model('Vacation', vacationSchema);
module.exports = Vacation;
```

Para usar el modelo en la aplicación tendríamos que importarlo:
```
var Vacation = require('./models/vacation.js');
```

Para **llenar la BBDD** hacemos primero un find para detectar si ya existen registros. En caso de que los haya suponemos que ya hemos cargado estos registros y que no hace falta que lo volvamos a hacer. En caso de que no haya ningún registro cargamos estas 3 vacaciones:
```
Vacation.find(function(err, vacations){
    if(vacations.length) return;

    new Vacation({
        name: 'Hood River Day Trip',
        slug: 'hood-river-day-trip',
        category: 'Day Trip',
        sku: 'HR199',
        description: 'Spend a day sailing on the Columbia and ' +
            'enjoying craft beers in Hood River!',
        priceInCents: 9995,
        tags: ['day trip', 'hood river', 'sailing', 'windsurfing', 'breweries'],
        inSeason: true,
        maximumGuests: 16,
        available: true,
        packagesSold: 0,
    }).save();

    new Vacation({
        name: 'Oregon Coast Getaway',
        slug: 'oregon-coast-getaway',
        category: 'Weekend Getaway',
        sku: 'OC39',
        description: 'Enjoy the ocean air and quaint coastal towns!',
        priceInCents: 269995,
        tags: ['weekend getaway', 'oregon coast', 'beachcombing'],
        inSeason: false,
        maximumGuests: 8,
        available: true,
        packagesSold: 0,
    }).save();

    new Vacation({
        name: 'Rock Climbing in Bend',
        slug: 'rock-climbing-in-bend',
        category: 'Adventure',
        sku: 'B99',
        description: 'Experience the thrill of climbing in the high desert.',
        priceInCents: 289995,
        tags: ['weekend getaway', 'bend', 'high desert', 'rock climbing'],
        inSeason: true,
        requiresWaiver: true,
        maximumGuests: 4,
        available: false,
        packagesSold: 0,
        notes: 'The tour guide is currently recovering from a skiing accident.',
    }).save();
});
```

Para **consultar información** y mostrarla por pantalla hacemos la siguiente vista (*views/vacations.handlebars*):
```
<h1>Vacations</h1>
{{#each vacations}}
    <div class="vacation">
        <h3>{{name}}</h3>
        <p>{{description}}</p>
        {{#if inSeason}}
            <span class="price">{{price}}</span>
            <a href="/cart/add?sku={{sku}}" class="btn btn-default">Buy Now!</a>
        {{else}}
            <span class="outOfSeason">We're sorry, this vacation is currently
            not in season.
            {{! The "notify me when this vacation is in season"
                page will be our next task. }}
            <a href="/notify-me-when-in-season?sku={{sku}}">Notify me when
            this vacation is in season.</a>
        {{/if}}
    </div>
{{/each}}
```

Y la completamos con la siguiente ruta que le manda los registros de la BBDD:
```
// see companion repository for /cart/add route....

app.get('/vacations', function(req, res){
    Vacation.find({ available: true }, function(err, vacations){
        var context = {
            vacations: vacations.map(function(vacation){
                return {
                    sku: vacation.sku,
                    name: vacation.name,
                    description: vacation.description,
                    price: vacation.getDisplayPrice(),
                    inSeason: vacation.inSeason,
                }
            })
        };
        res.render('vacations', context);
    });
});
```

Realizamos la búsqueda y con el objeto resultante mapeamos la información que necesitamos a un string. 

Para **añadir información a la BBDD** primero creamos un nuevo esquema (*models/vacationInSeasonListener.js*):
```
var mongoose = require('mongoose');

var vacationInSeasonListenerSchema = mongoose.Schema({
    email: String,
    skus: [String],
});
var VacationInSeasonListener = mongoose.model('VacationInSeasonListener',
    vacationInSeasonListenerSchema);

module.exports = VacationInSeasonListener;
```

Una vista que lo vaya a utilizar (*views/notify-me-when-in-season.handlebars*):
```
<div class="formContainer">
    <form class="form-horizontal newsletterForm" role="form"
            action="/notify-me-when-in-season" method="POST">
        <input type="hidden" name="sku" value="{{sku}}">
        <div class="form-group">
            <label for="fieldEmail" class="col-sm-2 control-label">Email</label>
            <div class="col-sm-4">
                <input type="email" class="form-control" required
                    id="fieldName" name="email">
            </div>
        </div>
        <div class="form-group">
            <div class="col-sm-offset-2 col-sm-4">
                <button type="submit" class="btn btn-default">Submit</button>
            </div>
        </div>
    </form>
</div>
```

Y por último la ruta que lo maneje:
```
var VacationInSeasonListener = require('./models/vacationInSeasonListener.js');

app.get('/notify-me-when-in-season', function(req, res){
    res.render('notify-me-when-in-season', { sku: req.query.sku });
});

app.post('/notify-me-when-in-season', function(req, res){
    VacationInSeasonListener.update(
        { email: req.body.email },
        { $push: { skus: req.body.sku } },
        { upsert: true },
        function(err){
            if(err) {
                console.error(err.stack);
                req.session.flash = {
                    type: 'danger',
                    intro: 'Ooops!',
                    message: 'There was an error processing your request.',
                };
                return res.redirect(303, '/vacations');
            }
            req.session.flash = {
                type: 'success',
                intro: 'Thank you!',
                message: 'You will be notified when this vacation is in season.',
            };
            return res.redirect(303, '/vacations');
        }
    );
});
```

Con este código estaremos creando un nuevo documento si es que no existía ninguno antes y si no estaremos rellenando uno ya existente. Para eso usamos **upsert** que es una combinación de update e insert. Con $push insertamos los valores en los campos deseados. 


##### Utilizar mongodb para almacenar la sesión
MongoDB no es la mejor opción para almacenar sesiones, simplemente porque se puede hacer mucho más, y por ello si simplemente se va a hacer eso es mejor utilizar **redis**. Es muy sencillo montar las sesiones de esta manera. Instalamos el paquete **session-mongoose** (npm install --save session-mongoose). Lo linkamos en el fichero de aplicación principal:
```
var MongoSessionStore = require('session-mongoose')(require('connect'));
var sessionStore = new MongoSessionStore({ url:
    credentials.mongo.connectionString });

app.use(require('cookie-parser')(credentials.cookieSecret));
app.use(require('express-session')({ store: sessionStore }));
```

Estas sesiones sólo se mantendrán mientras el usuario no borre las cookies. Por ejemplo podemos utilizarlo para que recuerde que moneda prefiere el usuario. Al final de nuestra página de vacaciones:
```
<hr>
<p>Currency:
    <a href="/set-currency/USD" class="currency {{currencyUSD}}">USD</a> |
    <a href="/set-currency/GBP" class="currency {{currencyGBP}}">GBP</a> |
    <a href="/set-currency/BTC" class="currency {{currencyBTC}}">BTC</a>
</p>
```

Un poco de css:
```
a.currency {
    text-decoration: none;
}
.currency.selected {
    font-weight: bold;
    font-size: 150%;
}
```

Y el manejo de rutas:
```
app.get('/set-currency/:currency', function(req,res){
    req.session.currency = req.params.currency;
    return res.redirect(303, '/vacations');
});

function convertFromUSD(value, currency){
    switch(currency){
        case 'USD': return value * 1;
        case 'GBP': return value * 0.6;
        case 'BTC': return value * 0.0023707918444761;
        default: return NaN;
    }
}

app.get('/vacations', function(req, res){
    Vacation.find({ available: true }, function(err, vacations){
        var currency = req.session.currency || 'USD';
        var context = {
            currency: currency,
            vacations: vacations.map(function(vacation){
                return {
                    sku: vacation.sku,
                    name: vacation.name,
                    description: vacation.description,
                    inSeason: vacation.inSeason,
                    price: convertFromUSD(vacation.priceInCents/100, currency),
                    qty: vacation.qty,
                }
            })
        };
        switch(currency){
            case 'USD': context.currencyUSD = 'selected'; break;
            case 'GBP': context.currencyGBP = 'selected'; break;
            case 'BTC': context.currencyBTC = 'selected'; break;
        }
        res.render('vacations', context);
    });
});
```



### CAP 14 - Routing

**Routing** es el mecanismo por el cual las peticiones son encaminadas al código que las maneja a través de lo que especifique su URL y método. 

**IA**: Information Architecture. Se trata del orden conceptual de tus contenidos. Es recomendable darle una vuelta antes de ponerse a tirar código. [Ensayo de 1998](http://www.w3.org/Provider/Style/URI.html) sobre el tema por uno de los *creadores* de Internet. Uno de los puntos que viene a decir es que nuestras URLs no pueden cambiar a lo largo del tiempo porque si no las referencias que puedan haberse hecho se pierden. Intenta categorizar las cosas de manera lógica. 

Recomendaciones para conseguir una IA duradera:

* **Nunca expongas detalles técnicos en tu URL**: Puede mostrar algo que actualmente esté pasado de moda.
* **Evita información insignificante en tus URL**: Que cada palabra que pongas tenga sentido. ¿Para que queremos /home si ya tenemos el root donde debería colgar tu página principal?
* **Evita URLs demasiado largas**: Pero no a cualquier precio. Debe ser legible.
* **Consistente con los separadores de palabra**: Mejor "-" que "_".
* **Nunca usar blancos ni caracteres no imprimibles**
* **Usar minúsculas**


##### SEO
Si quieres que tu sitio sea encontrado debes preocuparte del SEO y de como tus URL le afectan. Hay que tratar de que la URL que indiques tenga información de lo que va a mostrar pero no podemos volvernos locos y poner todo tipo de información sólo para que nuestro sitio esté mejor posicionado.

##### Subdominios
Otra manera de enrutar peticiones. Están reservados para partes significativamente diferentes de la aplicación (REST API - api.dominio.com, Administración - admin.dominio.com). Este tipo de contenido quedará fuera del SEO por lo que es importante discernir si nos importa que esté bien posicionado o si da igual (como los ejemplos que se han puesto). 

Para poder manejar los subdominios necesitamos del paquete **vhost** (npm install --save vhost) que permite enrutar para unos casos y otros, tal y como hacemos con app:
```
// create "admin" subdomain...this should appear
// before all your other routes
var admin = express.Router();
app.use(vhost('admin.*', admin));

// create admin routes; these can be defined anywhere
admin.get('/', function(req, res){
        res.render('admin/home');
});
admin.get('/users', function(req, res){
        res.render('admin/users');
});
``` 

Podemos ir complicando el enrutado. Dividir el tráfico entre dos app.get:
``` 
app.get('/foo', function(req,res,next){
        if(Math.random() < 0.5) return next();
        res.send('sometimes this');
});
app.get('/foo', function(req,res){
        res.send('and sometimes that');
});
```

Mostrar ofertas especiales (distintas en cada caso) en función de lo recuperado de la BBDD utilizando la propiedad **res.locals**:
```
function specials(req, res, next){
        res.locals.specials = getSpecialsFromDatabase();
        next();
}

app.get('/page-with-specials', specials, function(req,res){
        res.render('page-with-specials');
});
```

Y se puede implementar un **mecanismo de autorización** a través de lo siguiente:
```
function authorize(req, res, next){
        if(req.session.authorized) return next();
        res.render('not-authorized');
}

app.get('/secret', authorize, function(){
        res.render('secret');
})

app.get('/sub-rosa', authorize, function(){
        res.render('sub-rosa');
});
```

A través de **expresiones regulares** podemos hacer que varias rutas entren dentro de un mismo app.get. Los caracteres son +, ?, \*, ( y ). Para hacer que un ruta responda a *user* y *username*:
```
app.get('/user(name)?', function(req,res){
        res.render('user');
});
```

O un sitio que permite que pongas tantas "a" como quieras:
```
app.get('/khaa+n', function(req,res){
        res.render('khaaan');
});
```

**Parámetros de enrutado**: A través de variables indicadas en la url podemos modificar lo que se muestre a continuación. Esto lo vamos a utilizar mucho, especialmente para crear REST APIs. Como ejemplo:
```
var staff = {
        portland: {
                mitch: { bio: 'Mitch is the man to have at your back.' },
                madeline: { bio: 'Madeline is our Oregon expert.' },
        },
        bend: {
                walt: { bio: 'Walt is our Oregon Coast expert.' },
        },
};

app.get('/staff/:city/:name', function(req, res){
        var info = staff[req.params.city][req.params.name];
        if(!info) return next();        // will eventually fall through to 404
        res.render('staffer', info);
});
```


##### Organizando las rutas
No es factible definir todas nuestras rutas en el archivo principal de la aplicación. Un sitio sencillo puede tener unas cuantas rutas pero uno complejo podría tener cientos de rutas. 

Principios para saber como organizar las rutas:

* **Utilizar funciones con nombre para manejar las rutas**
* **Las rutas no deben ser un misterio**: Para un sitio grande con muchas rutas, lo recomendable es dividirlas por funcionalidad pero tienes que tener claro donde pertenecen y poder localizarlas rápido.
* **La organización de rutas debe ser extensible**: El método que elijas tiene que tener contemplado crecer.
* **Utiliza las vistas automáticas para manejar rutas**: Muy útil si el sitio contiene muchas páginas estáticas con URLs fijas.

**Declarar rutas en un módulo**
El primer paso para ordenar las rutas es meterlas en un módulo. Podemos hacerlo de dos maneras.

* **Opción 1:** Hacer del módulo una función que devuelva un array de objetos que contenga las propiedades "method" y "handler". Este método por ejemplo está bien para almacenar nuestras rutas dinámicamente, como puede ser una BBDD o un JSON. 

```
var routes = require('./routes.js')();

routes.forEach(function(route){
        app[route.method](route.handler);
})
```

* **Opción 2:** Más sencilla. Que el módulo sea el que añade las rutas:

```
module.exports = function(app){

        app.get('/', function(req,res){
                app.render('home');
        }))

        //...

};
```

Y sólo nos quedará llamarlo desde el archivo principal de nuestra aplicación:
```
require('./routes.js')(app);
```

Para esta segunda opción, que es la que elegiremos en el libro, separaremos las funciones con nombre a las que llamaremos en las rutas en distintos ficheros en función de su funcionalidad. Crearemos un directorio *handlers* y dentro tendremos distintos ficheros (*main.js* para el home, el about y alguna otra que consideremos, y ya después iremos creando en base a las funcionalidades que vayamos incorporando). Por ejemplo *main.js*:
```
var fortune = require('../lib/fortune.js');

exports.home = function(req, res){
        res.render('home');
};

exports.about = function(req, res){
        res.render('about', {
                fortune: fortune.getFortune(),
                pageTestScript: '/qa/tests-about.js'
        } );
};
```

Y modificamos *routes.js* para hacer uso de ello:
```
var main = require('./handlers/main.js');

module.exports = function(app){

        app.get('/', main.home);
        app.get('/about', main.about);
        //...

};
```

Y así iremos haciendo para ir cargando las rutas de las distintas funcionalidades de la aplicación. 


**Vistas que se renderizan solas**
Si tenemos mucho contenido en la aplicación pero no tiene mucha funcionalidad podemos evitar tener que estar indicando cada una de las rutas. Para hacerlo podemos hacer lo siguiente, situando el código antes de del handler del 404:
```
var autoViews = {};
var fs = require('fs');

app.use(function(req,res,next){
    var path = req.path.toLowerCase();
    // check cache; if it's there, render the view
    if(autoViews[path]) return res.render(autoViews[path]);
    // if it's not in the cache, see if there's
    // a .handlebars file that matches
    if(fs.existsSync(__dirname + '/views' + path + '.handlebars')){
        autoViews[path] = path.replace(/^\//, '');
        return res.render(autoViews[path]);
    }
    // no view found; pass on to 404 handler
    next();
});
```



### CAP 15 - REST APIs and JSON

En este capítulo no tenemos el navegador como el consumidor de nuestros servicios sino que van a ser otros programas, otras webs existentes en Internet. Vamos a añadir webservices a nuestra aplicación, que no hay problema de coexistencia con los servicios normales que hemos añadido hasta ahora. Hasta hace no mucho, con la aparición de los servicios RESTful, no ha sido una tecnología realmente accesible.

**REST**: Representational State Transfer. RESTful es un adjetivo para describir un webservice que satisface los principios REST. El concepto básico es que REST representa una conexión sin estado entre cliente y servidor. Además el servicio puede estar cacheado y llamar a otros servicios tras la llamada inicial. 

Primero hay que tener un plan de que queremos hacer. Por ejemplo, para la aplicación del libro, quiere cubrir los siguientes servicios:

*GET /api/attractions*   
Retrieves attractions. Takes lat, lng, and radius as querystring parameters and returns a list of attractions.

*GET /api/attraction/:id*   
Returns an attraction by ID.

*POST /api/attraction*   
Takes lat, lng, name, description, and email in the request body. The newly added attraction goes into an approval queue.

*PUT /api/attraction/:id*   
Updates an existing attraction. Takes an attraction ID, lat, lng, name, description, and email. Update goes into approval queue.

*DEL /api/attraction/:id*   
Deletes an attraction. Takes an attraction ID, email, and reason. Delete goes into approval queue.

Como se puede ver se distinguirá entre servicios mediante los paths y los métodos HTTP utilizados. POST se suele utilizar para añadir un nuevo contenido y PUT para actualizar uno existente. 

##### Reporte de errores en API
Ante los distintos casos de respuesta se utilizan los siguientes códigos HTTP:

* *Respuesta OK*: **Código 200**. 
* *Error catastrófico*: Supone un estado inestable o desconocido en el servidor. Generalmente una excepción no controlada. Normalmente sólo se puede salir de este estado reiniciando el servidor. Puede ser un **código 500** pero si el fallo es general lo normal es que ni responda y de **TIMEOUT**. 
* *Error recuperable*: Error controlado por el servidor, cuyo problema puede ser transitorio o bien permanente. Se responde a través de un **código 500**. 
* *Error de cliente*: Generalmente porque el cliente ha omitido o introducido incorrectamente ciertos parámetros. Lo mejor es utilizar códigos 40X para responder estos errores: **404** (Not found), **400** (Bad Request), **401** (Unauthorized). Además el cuerpo de la respuesta deberá indicar por qué se está produciendo el error. Si el usuario estuviese solicitando una lista de cosas y no hay nada que devolver, no es un error, sino que se devolvería una lista vacía.


**CORS**: Cross-Origin Resource Sharing. Permite recibir llamadas indicando que dominios están permitidos para ejecutar los scripts que indicamos. Instalamos el paquete (npm install --save cors). Para habilitarlo:
```
app.use(require('cors')());
```

Sólo lo aplicaremos donde sea necesario para no exponer partes que no deben. Para exponer el path */api*:
```
app.use('/api', require('cors')());
```


**PRUEBAS**
Las pruebas de APIs las haremos con **restler** (npm install --save-dev restler), aunque también se pueden hacer con postman. Introducimos el test en nuestra librería de tests (qa/tests-api.js):
```
var assert = require('chai').assert;
var http = require('http');
var rest = require('restler');

suite('API tests', function(){

    var attraction = {
        lat: 45.516011,
        lng: -122.682062,
        name: 'Portland Art Museum',
        description: 'Founded in 1892, the Portland Art Museum\'s colleciton ' +
            'of native art is not to be missed.  If modern art is more to your ' +
            'liking, there are six stories of modern art for your enjoyment.',
        email: 'test@meadowlarktravel.com',
    };

    var base = 'http://localhost:3000';

    test('should be able to add an attraction', function(done){
        rest.post(base+'/api/attraction', {data:attraction}).on('success',
                function(data){
            assert.match(data.id, /\w/, 'id must be set');
            done();
        });
    });

    test('should be able to retrieve an attraction', function(done){
        rest.post(base+'/api/attraction', {data:attraction}).on('success',
                function(data){
            rest.get(base+'/api/attraction/'+data.id).on('success',
                    function(data){
                assert(data.name===attraction.name);
                assert(data.description===attraction.description);
                done();
            })
        })
    });

});
```

Los tests tienen que ser independientes de sus ejecuciones por lo que siempre crearemos un elemento aunque ya hubiese corrido un test antes en que se hubiese creado ya. Funcionan con **promesas**. 

**Crear una API** con express es tan fácil como:
```
var Attraction = require('./models/attraction.js');

app.get('/api/attractions', function(req, res){
    Attraction.find({ approved: true }, function(err, attractions){
        if(err) return res.send(500, 'Error occurred: database error.');
        res.json(attractions.map(function(a){
            return {
                name: a.name,
                id: a._id,
                description: a.description,
                location: a.location,
            }
        }));
    });
});

app.post('/api/attraction', function(req, res){
    var a = new Attraction({
        name: req.body.name,
        description: req.body.description,
        location: { lat: req.body.lat, lng: req.body.lng },
        history: {
            event: 'created',
            email: req.body.email,
            date: new Date(),
        },
        approved: false,
    });
    a.save(function(err, a){
        if(err) return res.send(500, 'Error occurred: database error.');
        res.json({ id: a._id });
    });
});

app.get('/api/attraction/:id', function(req,res){
    Attraction.findById(req.params.id, function(err, a){
        if(err) return res.send(500, 'Error occurred: database error.');
        res.json({
            name: a.name,
            id: a._id,
            description: a.description,
            location: a.location,
        });
    });
});
```

Que ataca el modelo generado (*models/attraction.js*):
```
var mongoose = require('mongoose');

var attractionSchema = mongoose.Schema({
    name: String,
    description: String,
    location: { lat: Number, lng: Number },
    history: {
        event: String,
        notes: String,
        email: String,
        date: Date,
    },
    updateId: String,
    approved: Boolean,
});
var Attraction = mongoose.model('Attraction', attractionSchema);
module.exports = Attraction;
```

##### Usar un plugin REST
Pese a que podemos crear la API utilizando sólo Express hay ventajas por hacerlo mediante un plugin de connect **connect-rest** (npm install --save connect-rest) y lo requeriremos en el archivo principal de nuestra aplicación:
```
var rest = require('connect-rest');
```

Ponemos primero las rutas normales de la aplicación y luego irían las de la API (/api) y siempre antes del handler 404. 
```
// website routes go here

// define API routes here with rest.VERB....

// API configuration
var apiOptions = {
    context: '/api',
    domain: require('domain').create(),
};

// link API into pipeline
app.use(rest.rester(apiOptions));

// 404 handler goes here
```

> Para máxima separación entre API y aplicación lo mejor es utilizar un subdominio *api.nombre_aplicacion.com*.

Los métodos de la API los añadimos entonces de la siguiente manera:
```
rest.get('/attractions', function(req, content, cb){
    Attraction.find({ approved: true }, function(err, attractions){
        if(err) return cb({ error: 'Internal error.' });
        cb(null, attractions.map(function(a){
            return {
                name: a.name,
                description: a.description,
                location: a.location,
            };
        }));
    });
});

rest.post('/attraction', function(req, content, cb){
    var a = new Attraction({
        name: req.body.name,
        description: req.body.description,
        location: { lat: req.body.lat, lng: req.body.lng },
        history: {
            event: 'created',
            email: req.body.email,
            date: new Date(),
        },
        approved: false,
    });
    a.save(function(err, a){
        if(err) return cb({ error: 'Unable to add attraction.' });
        cb(null, { id: a._id });
    });
});

rest.get('/attraction/:id', function(req, content, cb){
    Attraction.findById(req.params.id, function(err, a){
        if(err) return cb({ error: 'Unable to retrieve attraction.' });
        cb(null, {
            name: attraction.name,
            description: attraction.description,
            location: attraction.location,
        });
    });
});
```

Las funciones REST toman tres parámetros: **request** como es normal, **content** que es el cuerpo parseado de la petición y **cb** que es una callback function que se utiliza para llamadas asíncronas de la API (por ejemplo llamar a la BBDD).

Para reiniciar el servidor en caso de que estemos dando errores:
```
apiOptions.domain.on('error', function(err){
    console.log('API domain error.\n', err.stack);
    setTimeout(function(){
        console.log('Server shutting down after API domain error.');
        process.exit(1);
    }, 5000);
    server.close();
    var worker = require('cluster').worker;
    if(worker) worker.disconnect();
});
```

##### Usar un subdominio
Al ser una parte separada de la aplicación es muy típico utilizar un subdominio para ofrecer una API (*api.aplicacion.com*). El habla de como habilitarlo a partir del fichero hosts del PC. ¿Pero como se sabe como llegar a tu PC cuando no estás en un entorno de Desarrollo? Mirar en internet como se hace de verdad.



### CAP 16 - Static Content

Objetos que no van a cambiar entre petición y petición. Suelen ser objetos multimedia, CSS, javascript que ejecuta el cliente y archivos binarios de descarga. No metemos en este saco a HTML ya que si lo hiciésemos esas URLs se verían terminadas en .html y no queremos hacer eso, así que los implementaremos mediante vistas.

Como manejas el contenido estático de tu web afecta mucho al **rendimiento** de la misma. Las dos principales premisas son reducir el número de peticiones y reducir el tamaño del contenido. 

* **Número de peticiones**: Lo podemos reducir bien cacheando en el navegador o combinando recursos. Las imágenes pequeñas han de combinarse con el resto en un único sprite. Luego a través de CSS cogemos la parte que nos interesa de la imagen combinada (lo puedes crear a través de [SpritePad](https://wearekiss.com/spritepad)). 
* **Tamaño contenido**: Se puede reducir sin pérdida (minificar CSS/JS y optimizar imágenes) o con ella (incrementar el nivel de compresión).


##### Preparar tu sitio para el futuro
Cuando pones en producción una página web lo normal es almacenar los estáticos en otro servidor especializado y no tenerlos junto al resto de la aplicación. Se utilizarán los **CDN (Content Delivery Network) para almacenarlos** ya que son elementos especializados en su almacenamiento y distribución, acelerando la entrega de los ficheros a los clientes demandantes. Cacheo, cercanía geográfica... son características que utilizan.

A la hora de mapear los recursos lo haremos **indicando la ruta relativa**. Es decir, utilizar \<img src="/img/meadowlark\_logo.png" alt="Meadowlark Travel Logo"\> en vez de \<img src="//s3-us-west-2.amazonaws.com/meadowlark/img/meadowlark\_logo-3.png" alt="Meadowlark Travel Logo"\>. Utilizaremos **URLs de protocolo relativo**, que están con "//" en vez de "http://" o "https://".

Para hacer el **path dinámico** en función de donde estemos almacenando el contenido estático crearemos el fichero *lib/static.js*:
```
var baseUrl = '';

exports.map = function(name){
        return baseUrl + name;
}
```

Para **utilizarlo en vistas** haremos uso de los helpers:
```
// set up handlebars view engine
var handlebars = require('express3-handlebars').create({
    defaultLayout:'main',
    helpers: {
        static: function(name) {
            return require('./lib/static.js').map(name);
        }
    }
});
```

Y entonces podremos llamar a nuestros recursos de la siguiente manera:
```
<header><img src="{{static '/img/logo.jpg'}}"
    alt="Meadowlark Travel Logo"></header>
```

Para **utilizarlo en CSS** tendremos que hacer uso de un preprocesador de CSS para poder utilizar variables. Podremos utilizar LESS, SASS, Stylus... Realiza un ejemplo con LESS, Grunt y una función de ayuda en el fichero de configuración *grunt.js*. 

Para hacer uso de ello en los **JS estáticos de servidor** lo haremos a través de una función que nos realice el mapeo. Por ejemplo:
```
var static = require('./lib/static.js').map;

app.use(function(req, res, next){
        var now = new Date();
        res.locals.logoImage = now.getMonth()==11 && now.getDate()==19 ?
                static('/img/logo_bud_clark.png') :
                static('/img/logo.png');
        next();
});
```

En nuestro fichero handlebars lo llamaremos de la siguiente manera:
```
<header><img src="{{logoImage}}" alt="Meadowlark Travel Logo"></header>
``` 

Para utilizarlo en **JS estáticos de cliente** jQuery se puede encargar de cambiar los recursos si es necesario, por ejemplo :
``` 
$(document).on('meadowlark_cart_changed', function(){
        $('header img.cartIcon').attr('src', cart.isEmpty() ?
                IMG_CART_EMPTY : IMG_CART_FULL );
});
``` 

Con la variables definidas *views/layouts/main.handlebars*:
``` 
<!-- ... -->
<script>
    var IMG_CART_EMPTY = '{{static '/img/shop/cart_empty.png'}}';
    var IMG_CART_FULL = '{{static '/img/shop/cart_full.png'}}';
</script>
``` 

Las **cabeceras que utiliza el navegador** para saber cuando cachear un recurso son:

* **Expires**: Le dicen al navegador cuanto tiempo pueden llegar a cachear un recurso. El navegador respeta estos tiempos a menos que se borre la caché manualmente por el usuario, el navegador libere espacio porque se ha quedado sin recursos disponibles o algo similar. Esto mejorará el resultado, sobretodo en navegación móvil. Si no ha pasado el tiempo no se mandará ninguna petición al servidor por ese recurso.
* **Last-Modified**/**ETag**: La última vez que fue modificado un recurso y un id de versión del recurso. Se mandará una petición GET al servidor que será respondida con estas dos cabeceras. Si coinciden con el valor que tenemos no solicitaremos el recurso al servidor. 

La funcionalidad respecto al manejo de estas cabeceras está **muy limitada por Express** (el middleware static no tiene en cuenta Last-Modified ni Etag). Si no queremos hacer uso de un CDN tendremos que utilizar un **proxy-server como Nginx**.

**Cambiar el contenido estático**
Aunque el cacheo tiene ventajas también tiene algún inconveniente. Si cambias tus recursos estáticos el cliente no verá los cambios hasta que no caduque la fecha en que los tiene cacheados. Google recomienda ponerles 1 mes de vigencia, incluso 1 año. Pero como no podemos tener a los usuarios desactualizados tanto tiempo lo que recomiendan es utilizar en el nombre un indicador de la versión y así evitar estos problemas (logo.png por logo-1.png cambiando las referencias en el código).

**Bundling y Minification**
El primero sirve para juntar distintos ficheros en uno solo, así reduciendo el número de peticiones. El segundo sirve para reducir el tamaño de los ficheros quitando todo lo que es innecesario (como espacios en blanco para hacerlo legible a los humanos) y acortar nombres de variables. **Grunt** nos ayudará a realizar estas tareas. Para hacer todo eso utilizaremos los siguientes módulos:
```
npm install --save-dev grunt-contrib-uglify
npm install --save-dev grunt-contrib-cssmin
npm install --save-dev grunt-hashres
``` 

Cargamos esas tareas en el gruntfile:
```
    [
        // ...
        'grunt-contrib-less',
        'grunt-contrib-uglify',
        'grunt-contrib-cssmin',
        'grunt-hashres',
    ].forEach(function(task){
        grunt.loadNpmTasks(task);
    });
```

Y luego preparamos las tareas:
```
    grunt.initConfig({
        // ...
        uglify: {
            all: {
                files: {
                    'public/js/meadowlark.min.js': ['public/js/**/*.js']
                }
            }
        },
        cssmin: {
            combine: {
                files: {
                    'public/css/meadowlark.css': ['public/css/**/*.css',
                        '!public/css/meadowlark*.css']
                }
            },
            minify: {
                src: 'public/css/meadowlark.css',
                dest: 'public/css/meadowlark.min.css',
            }
        },
        hashres: {
            options: {
                fileNameFormat: '${name}.${hash}.${ext}'
            },
            all: {
                src: [
                    'public/js/meadowlark.min.js',
                    'public/css/meadowlark.min.css',
                ],
                dest: [
                    'views/layouts/main.handlebars',
                ]
            },
        }
    });
};
```

Estas tareas hacen:

* uglify: Combina todo el js de la web en un sólo fichero.
* cssmin: Combina todos los ficheros css en uno sólo y al final lo minimiza.
* hashres: Genera un hash que pone en las terminaciones de los ficheros y cambia las referencias que haya a los mismos en el código. 

Tras haber generado los ficheros minimizados/unidos tendremos que cambiar las referencias en nuestro layout:
```
    <!-- ... -->
    <script src="http://code.jquery.com/jquery-2.0.2.min.js"></script>
    <script src="{{static '/js/meadowlark.min.js'}}"></script>
    <link rel="stylesheet" href="{{static '/css/meadowlark.min.css'}}">
</head>
```

Estas tareas tienen dependencias así que es importante hacerlas en orden:
```
grunt less
grunt cssmin
grunt uglify
grunt hashres
```

Para no tener que recordar y hacer todo ese trabajo nos configuramos una nueva tarea (grunt static):
```
grunt.registerTask('default', ['cafemocha', 'jshint', 'exec']);
grunt.registerTask('static', ['less', 'cssmin', 'uglify', 'hashres']);
```

Todo esto es muy ventajoso en Producción pero puede hacer que el trabajo de depuración sea muy complicado. Para solucionarlo lo ideal sería desactivar estos ajustes cuando se están realizando tareas de desarrollo.

Primero crea un fichero de configuración (*config.js*):
```
module.exports = {
    bundles: {

        clientJavaScript: {
            main: {
                file: '/js/meadowlark.min.js',
                location: 'head',
                contents: [
                    '/js/contact.js',
                    '/js/cart.js',
                ]
            }
        },

        clientCss: {
            main: {
                file: '/css/meadowlark.min.css',
                contents: [
                    '/css/main.css',
                    '/css/cart.css',
                ]
            }
        }
    }
}
```

Para los JS especificamos el sitio donde queremos que se inserte. Ahora modificamos el fichero *views/layouts/main.handlebars*:
```
    <!-- ... -->
    {{#each _bundles.css}}
        <link rel="stylesheet" href="{{static .}}">
    {{/each}}
    {{#each _bundles.js.head}}
        <script src="{{static .}}"></script>
    {{/each}}
</head>
```

Las **librerías de terceros** normalmente no se suele meter en los paquetes anteriormente mencionados. Es debido a que seguramente sean utilizadas por otras webs accedidas. A menos que llamemos a multitud de librerías externas que ya puede empezar a interesarnos.



### CAP 17 - Implementing MVC in Express

MVC (modelo vista-controlador) es un paradigma de la programación introducido en 1970 y que ha visto su resurgir gracias al desarrollo web. Este paradigma facilita el arranque de los proyectos y lo hace más independiente del lenguaje utilizado. Desglosa la funcionalidad en entornos claramente diferenciados, dando un framework común sobre el que construir software. Los tres elementos son:

* **Modelo:** La lógica y como están organizados los datos. 
* **Vista:** La parte que ve el usuario y con la que interactúa. 
* **Controlador:** A través del cuál manipula el modelo y elige que vista visualizar.

**Modelo**
Tus modelos son el corazón del proyecto. Es muy importante definirlos correctamente para que el resto vaya rodado. En un mundo ideal la capa de persistencia estaría totalmente separada del modelo, pero como es algo que requiere mucho trabajo no suelen estar tan separadas. En este libro se utilizado Mongoose para definir los modelos que utiliza la aplicación. La lógica y datos del proyecto deberán ir dentro de la carpeta *models/*. Por ejemplo las relaciones de cliente (*models/costumer.js*):
``` 
var mongoose = require('mongoose');
var Orders = require('./orders.js');

var customerSchema = mongoose.Schema({
        firstName: String,
        lastName: String,
        email: String,
        address1: String,
        address2: String,
        city: String,
        state: String,
        zip: String,
        phone: String,
        salesNotes: [{
                date: Date,
                salespersonId: Number,
                notes: String,
        }],
});

customerSchema.methods.getOrders = function(){
        return Orders.find({ customerId: this._id });
};

var Customer = mongoose.model('Customer', customerSchema);
modules.export = Customer;
```


**Vista**
Si hay que mostrar algo en una vista lo que hay que hacer es crear una vista y no modificar el modelo. En esa vista elegiremos la información que queremos mostrar del modelo y cómo. Para mostrar información sobre los clientes y las órdenes que tienen (*viewModels/customer.js*):
```
var Customer = require('../model/customer.js');

// convenience function for joining fields
function smartJoin(arr, separator){
        if(!separator) separator = ' ';
        return arr.filter(function(elt){
                return elt!==undefined &&
                        elt!==null &&
                        elt.toString().trim() !== '';
        }).join(separator);
}

module.exports = function(customerId){
        var customer = Customer.findById(customerId);
        if(!customer) return { error: 'Unknown customer ID: ' +
                req.params.customerId };
        var orders = customer.getOrders().map(function(order){
                return {
                        orderNumber: order.orderNumber,
                        date: order.date,
                        status: order.status,
                        url: '/orders/' + order.orderNumber,
                }
        });
        return {
                firstName: customer.firstName,
                lastName: customer.lastName,
                name: smartJoin([customer.firstName, customer.lastName]),
                email: customer.email,
                address1: customer.address1,
                address2: customer.address2,
                city: customer.city,
                state: customer.state,
                zip: customer.zip,
                fullAddress: smartJoin([
                        customer.address1,
                        customer.address2,
                        customer.city + ', ' +
                                customer.state + ' ' +
                                customer.zip,
                ], '<br>'),
                phone: customer.phone,
                orders: customer.getOrders().map(function(order){
                        return {
                                orderNumber: order.orderNumber,
                                date: order.date,
                                status: order.status,
                                url: '/orders/' + order.orderNumber,
                        }
                }),
        }
}
``` 

Para seleccionar (o descartar) que información queremos mostrar también podemos utilizar la librería **underscore** que facilita la tarea. 


**Controlador**
El responsable de manejar la interacción del usuario y elegir las vistas apropiadas en función de las elecciones del usuario. Es **como el enrutado pero agrupando por funcionalidad**. El controlador para ver y editar la información de usuario (*controllers/customer.js*):
``` 
var Customer = require('../models/customer.js');
var customerViewModel = require('../viewModels/customer.js');

exports = {

        registerRoutes: function(app) {
                app.get('/customer/:id', this.home);
                app.get('/customer/:id/preferences', this.preferences);
                app.get('/orders/:id', this.orders);

                app.post('/customer/:id/update', this.ajaxUpdate);
        }

        home: function(req, res, next) {
                var customer = Customer.findById(req.params.id);
                if(!customer) return next();    // pass this on to 404 handler
                res.render('customer/home', customerViewModel(customer));
        }

        preferences: function(req, res, next) {
                var customer = Customer.findById(req.params.id);
                if(!customer) return next();    // pass this on to 404 handler
                res.render('customer/preferences', customerViewModel(customer));
        }

        orders: function(req, res, next) {
                var customer = Customer.findById(req.params.id);
                if(!customer) return next();    // pass this on to 404 handler
                res.render('customer/preferences', customerViewModel(customer));
        }

        ajaxUpdate: function(req, res) {
                var customer = Customer.findById(req.params.id);
                if(!customer) return res.json({ error: 'Invalid ID.'});
                if(req.body.firstName){
                        if(typeof req.body.firstName !== 'string' ||
                                req.body.firstName.trim() === '')
                                return res.json({ error: 'Invalid name.'});
                        customer.firstName = req.body.firstName;
                }
                // and so on....
                customer.save();
                return res.json({ success: true });
        }
}
```

Con la actualización AJAX, y siempre que modifiquemos información en el servidor a través de formularios, debemos validar los campos para que no nos puedan hacer posibles ataques  




### CAP 18 - Security

Mantener seguridad de la aplicación o bien autenticar al usuario para mantener información personal.

##### HTTPS
Encriptar los paquetes que viajan por la red. Es la base sobre la que montar la autenticación en tu página web. Se basa en los certificados SSL, los cuales puedes conseguirlos de un CA gratuito, de un CA de pago o bien hacerlo tu mismo. 

* **Propio:** Desarrollo, testeo o bien una intranet. Al no darlo como seguro el navegador la gente iba a ver el mensaje de alerta y no va a confiar en el sitio. Para poder generarlo necesitas **openssl**. También puedes conseguir certificados en [http://www.selfsignedcertificate.com](SelfSignedCertificate).
* **CA gratuito:** Hay opciones pero no son realmente útiles ya que sólo funcionando con Firefox. [http://www.cacert.org/](CACert).
* **CA pago:** Prácticamente todos pertenecen a 4 grandes empresas: Symantec, Comodo, Go Daddy y GlobalSign. Una forma de abaratarlo es comprarlo a un revendedor (~10$). Se pueden comprar al proveedor de hosting. 

Al final tendremos una **clave privada** y un **certificado**. Los almacenaremos en el subdirectorio *ssl/* y será tan fácil como arrancar el servidor de la siguiente manera:
```
var https = require('https');   // usually at top of file

var options = {
        key: fs.readFileSync(__dirname + '/ssl/meadowlark.pem');
        cert: fs.readFileSync(__dirname + '/ssl/meadowlark.crt');
};

https.createServer(options, app).listen(app.get('port'), function(){
        console.log('Express started in ' + app.get('env') +
                ' mode on port ' + app.get('port') + '.');
});
```

El puerto por defecto para HTTP es el 80 y para HTTPS es el 443. Para levantar cualquier puerto en Linux entre el 0-1024 se requiere hacerlo con SUDO ya que necesita de privilegios. Una forma de evitarlo es poniendo un proxy que hable con el servidor por el puerto que tenga levantado. 

Si tuviesemos un proxy, la comunicación HTTPS sería entre cliente y proxy. De proxy a servidor web ya está en un entorno seguro por lo que será HTTP. Si se utiliza un proxy no hay que olvidarse de indicarlo a Express (app.enable('trust proxy')).

##### CSRF - Cross-Site Request Forgery
Utilizar sesiones logadas en un navegador desde otra aplicación. Para evita esto hay que indicar de alguna manera que la petición viene de tu propia aplicación y eso lo hacen mediante el uso de un **token**. Para ello utilizamos el middleware **surf** (npm install --save csurf) y lo linkamos:
```
// this must come after we link in cookie-parser and connect-session
app.use(require('csurf')());
app.use(function(req, res, next){
        res.locals._csrfToken = req.csrfToken();
        next();
});
```

Y habrá que incluirlo en todos los formularios o llamadas AJAX:
```
<form action="/newsletter" method="POST">
        <input type="hidden" name="_csrf" value="{{_csrfToken}}">
        Name: <input type="text" name="name"><br>
        Email: <input type="email" name="email"><br>
        <button type="submit">Submit</button>
</form>
``` 

Para las APIs utilizaremos **API key** de connect-rest.


##### Authentication
Recomienda no hacer nada por uno mismo a menos que se sea un experto, ya que se tiene mucho que perder. Distinguir entre **autenticación** (comprobar que alguien es quién dice) y **autorización** (que es lo que puede hacer un usuario). Va a enseñar como hacer un sistema de autorización muy básico en el que sólo hay cliente/empleado. Hay varios tipos:

* **Autenticación a partir de terceros** Utilizar la cuenta que ya tengan de otro servicio. Dos tipos: federated authentication (SAML / OpenID) y delegated authentication (OAuth). Es amigable hacia los usuarios. Hay sitios donde se permite esto o si el usuario lo desea generar un usuario en la web.  
* **Guardar usuarios en la BBDD** Guardar toda la información de los usuarios en la BBDD o sino al menos parte de ella si es que se logan con terceros. Podemos utilizar el siguiente modelo para usuarios:

```
var mongoose = require('mongoose');

var userSchema = mongoose.Schema({
        authId: String,
        name: String,
        email: String,
        role: String,
        created: Date,
});

var User = mongoose.model('User', userSchema);
module.exports = User;
```

**Passport**
Módulo popular y robusto de autenticación para Express. No está atado a ningún tipo de autenticación en especial.

Para el caso de autenticación de terceros la password nunca está en nuestra aplicación sino que todo recae en redirecciones. 

*IMAGEN WDNE-passport-authoritation* 

*IMAGEN WDNE-passport-interactions*

Los distintos pasos que importan son:

* **Página de login:** Es donde el usuario escoge la manera de autenticarse. Si es directamente en la aplicación tendrá un cajetín y si es a base de terceros tendrá un link para ir a su plataforma. Es a donde se llevará cuando se intente acceder a una página sin permisos.
* **Construcción de la petición de autenticación:** Donde se crea la petición de redirección para ir a la plataforma de terceros. Aquí es donde indicamos que es lo que queremos obtener de la información del usuario en ese sistema.
* **Comprobar respuesta de autenticación**: Si el usuario autoriza tu aplicación recibirás una respuesta de autenticación válida de la plataforma de terceros. Para el siguiente paso tendremos que tener un registro de que el usuario está autenticado (normalmente se guarda en una sesión con el ID del usuario).
* **Comprobar autenticación:** Ahora ya validaremos contra el objeto de nuestra BBDD *User* para saber qué le está permitido hacer al usuario.


##### Ejemplo de como hacer login con Facebook
Lo primero que tienes que hacer es asociar tu web con la app de Facebook para dar permisos sobre la cuenta que se utilizará como origen de las peticiones de autenticación. Lo que habría que instalar para este caso serían los módulos **passport** y **passport-facebook** (npm install --save passport passport-facebook). 

Todo el código de autenticación lo pondremos en un fichero aparte para no mezclar cosas (*lib/auth.js*). 

Importante hacer volver al usuario donde estaba antes de logarse con la plataforma externa. Mejorará la experiencia de usuario.


##### Autorización basada en roles
Para permitir la visualización de ciertas páginas exclusivamente a determinados roles. 

Para lo que queremos que se vea como cliente:
``` 
function customerOnly(req, res){
        var user = req.session.passport.user;
        if(user && req.role==='customer') return next();
        res.redirect(303, '/unauthorized');
}
``` 

Y como empleado (tenemos que evitar dar pistas sobre páginas no autorizadas y que pueden tener información sensible, por eso el next('route')):
``` 
function employeeOnly(req, res, next){
        var user = req.session.passport.user;
        if(user && req.role==='employee') return next();
        next('route');
}
``` 

Y ponemos las rutas de cada uno:
``` 
// customer routes
app.get('/account', customerOnly, function(req, res){
        res.render('account');
});
app.get('/account/order-history', customerOnly, function(req, res){
        res.render('account/order-history');
});
app.get('/account/email-prefs', customerOnly, function(req, res){
        res.render('account/email-prefs');
});

// employer routes
app.get('/sales', employeeOnly, function(req, res){
        res.render('sales');
});
``` 

La autorización se puede hacer de muchas maneras. Por ejemplo:
``` 
function allow(roles) {
        var user = req.session.passport.user;
        if(user && roles.split(',').indexOf(user.role)!==-1) return next();
        res.redirect(303, '/unauthorized');
}

app.get('/account', allow('customer,employee'), function(req, res){
        res.render('account');
});
```



### CAP 19 - Integrating with Third-Party APIs

Las webs ya no suelen ser completas por si mismas, sino que utilizan funcionalidades de webs de terceros. Por ejemplo las redes sociales o las aplicaciones de geolocalización.

Muy importante cuidarse de cuantas peticiones hacemos desde nuestra página. Podemos hacer que aumente el tiempo de carga o el consumo de datos en redes móviles.

Tendremos que utilizar las APIs que proporcionan para llamar a funciones como publicar un tweet o conseguir las ultimas historias de un usuario. Es posible que esas APIs ya estén cacheadas en el navegador y que no haya que descargarlas.    


##### Redes sociales
Aunque las peticiones a la API se pueden hacer desde el FrontEnd, lo recomendable es hacerlo desde el Backend para cachear información o bien añadir funcionalidades como descartar elementos indeseados. Para poder utilizar esas APIs tendremos que crear un token en las páginas de desarrollo de los servicios que queremos utilizar. Nunca hay que pasar la clave hacia el usuario o podría haber gente que la utilizase para hacer llamadas maliciosas. 

El código que generemos lo pondremos en un fichero aparte (por ejemplo */lib/twitter.js*) y siempre haremos algo similar a lo siguiente (lo que no devolvamos sólo se ejecutará la primera vez, como por ejemplo conseguir el token):
``` 
var https = require('https');

module.exports = function(twitterOptions){

        // this variable will be invisible outside of this module
        var accessToken;

        // this function will be invisible outside of this module
        function getAccessToken(cb){
                if(accessToken) return cb(accessToken);
                // TODO: get access token
        }

        return {
                search: function(query, count, cb){
                        // TODO
                },
        };
};
```

Y luego consumiremos los servicios de la siguiente manera (con las credenciales en *credentials.js*):
```
var twitter = require('./lib/twitter')({
        consumerKey: credentials.twitter.consumerKey,
        consumerSecret: credentials.twitter.consumerSecret,
});

twitter.search('#meadowlarktravel', 10, function(result){
        // tweets will be in result.statuses
});
```

Conseguir por tanto el token lo haríamos de la siguiente manera en función de lo que indica la documentación de Twitter:
```
function getAccessToken(cb){
    if(accessToken) return cb(accessToken);

    var bearerToken = Buffer(
        encodeURIComponent(twitterOptions.consumerKey) + ':' +
        encodeURIComponent(twitterOptions.consumerSecret)
    ).toString('base64');

    var options = {
        hostname: 'api.twitter.com',
        port: 443,
        method: 'POST',
        path: '/oauth2/token?grant_type=client_credentials',
        headers: {
            'Authorization': 'Basic ' + bearerToken,
        },
    };

    https.request(options, function(res){
        var data = '';
        res.on('data', function(chunk){
            data += chunk;
        });
        res.on('end', function(){
            var auth = JSON.parse(data);
            if(auth.token_type!=='bearer') {
                console.log('Twitter auth failed.');
                return;
            }
            accessToken = auth.access_token;
            cb(accessToken);
        });
    }).end();
}
```

Y realizar búsquedas lo haríamos de la siguiente manera:
```
search: function(query, count, cb){
        getAccessToken(function(accessToken){
                var options = {
                        hostname: 'api.twitter.com',
                        port: 443,
                        method: 'GET',
                        path: '/1.1/search/tweets.json?q=' +
                                encodeURIComponent(query) +
                                '&count=' + (count || 10),
                        headers: {
                                'Authorization': 'Bearer ' + accessToken,
                        },
                };
                https.request(options, function(res){
                        var data = '';
                        res.on('data', function(chunk){
                                data += chunk;
                        });
                        res.on('end', function(){
                                cb(JSON.parse(data));
                        });
                }).end();
        });
},
```

Habrá situaciones en las que tendremos que esperar a que finalicen llamadas asíncronas. Para ello utilizaremos **Promesas** (**Q** antes de que existiese en JS).

*Introducir WDNE-api-promises*


##### Geolocalización
Se refiere al hecho de cambiar direcciones de calles o nombres de sitios por las coordenadas geográficas que les representen. Si extraes información de la API de un proveedor (google, bing...) debes mostrar la información en sus mapas.



### CAP 20 - Debugging

Podría llamarse Explorar en vez de Depurar ya que es algo que estaremos utilizando todo el rato no sólo para arreglar cosas sino para descubrir como funciona algo, implementar nuevas funciones... 

El primer paso para depurar parte de la premisa de que se puede eliminar como posible causa del problema que ese está teniendo. Eso hará que sea menor el área que tengamos que observar para encontrar la causa origen. La eliminación puede hacerse de varias maneras: comentando código, test unitarios, analizando tráfico para saber si es problema de servidor o de cliente, ir cambiando la entrada al programa hasta que se da con una que falla, utilizar el Control de Versiones hacia atrás hasta que el problema desaparece o hacer bocetos de la funcionalidad para eliminar subsistemas complejos. 

##### REPL
Se trata de la consola, bien node o bien en el navegador, para poder probar trocitos de código. Utilizar console.log para ver el contenido de objetos, puntos alcanzados...

##### Depurador integrado en Node
Lo lanzamos ejecutando:
```
node debug nombre_app.js
```

En la consola podremos ver mensajes del depurador y podremos escribir *help* para que nos de una lista de comandos a utilizar. Ese depurador se levantará por defecto contra el **puerto 5858** al que podremos acceder con aplicaciones como **Node Inspector**.

```
sudo npm install -g node-inspector
```

Y luego lo lanzamos en background sin parámetros. A partir de entonces podemos lanzar la app en debug (si lo hacemos con --debug no pintará nada por consola). Para acceder al depurador tendrás que dirigirte a la web que indica la herramienta *http://localhost:8080/debug?port=5858*. Con esto podremos ir poniendo y quitando breakpoints, viendo por donde se encuentra en el código, acceder a un terminal en tiempo real con la aplicación, cambiar valores de variables... Podemos vigilar cosas en concreto con **watch expressions**, con **call stack** ver de como hemos llegado hasta donde estamos, **scope variables** que te dice lo que contiene actualmente el scope. 

Si queremos ver como arranca la aplicación tendremos que parar la carga desde el principio mediante:
```
node --debug-brk meadowlark.js
```



### CAP 21 - Going live

Y de repente un día tu sitio se encuentra corriendo en Internet.

##### Registro de Dominio y Hosting
Un nombre de dominio es como si fuera el nombre de la empresa y la dirección IP es como si fuera la dirección física. Los **dominios** crean la relación entre esos dos conceptos. Esta relación se hace a través de los **DNS**. Que te roben el dominio puede hacer un destrozo serio en tus planes por lo que hay que buscar un dominio de calidad (y no suelen ser caros). El **recomienda** *name* y *namecheap.com*.

**Hosting** por otro lado indica el servidor donde se aloja tu sitio web. Podríamos decir que son los edificios físicos que verías cuando llegases a la dirección física.

Los dominios se dividen en:

* General TLDs: .com, .net, .gov, .edu,.fed, .mil. De ahí sólo los dos primeros se pueden adquirir libremente. 

* Country Code TLDs: .us, .es, .uk. Con estos se pueden hacer dominios muy interesantes (por ejemplo goo.gl). 

**Subdominios** se trata del www de antes del dominio. El autor recomienda no poner nada, es decir, mejor *http://meadowlarktravel.com/* que *http://www.meadowlarktravel.com/*. También se puede utilizar por razones técnicas (usas un servidor diferente para esa parte) para acceder a sitio móvil *m.tu_sitio.com*, las apis *api.tu_sitio.com*... Otro útil es para tener la administración separada *admin.tu_sitio.com*. **El registro de Dominio**, a menos que especifiques lo contrario, **redireccionará todo el tráfico a tu servidor sin importar el subdominio**. Ya ahí serás tú quién vea como actuar.

**Nameservers** son a quienes van los registros de Dominio a preguntar por donde está el servidor que da servicio a la web. Esto lo tendremos que utilizar si nuestro hosting nos dá una *ip dinámica*. Si fuese *estática* podríamos ir directamente al servidor donde está alojado saltando un paso intermedio.

**Para implantar los ficheros** utilizar **GIT** de cualquiera de los que dan servicio (github, bitbucket...). Lo mejor es tener una rama que sea **production**, otra que sea **master** y las que se necesiten de **stagging** para ir desarrollando nuevas partes de la aplicación o corrigiendo problemas. De master iremos a los distintos stagging, de los cuales haremos merge con el master una vez estén terminados. Cuando estemos listos para llevarlo a producción introduciremos esos cambios en la rama production. Como production será una rama de sólo lectura podremos volver atrás en el tiempo si alguna vez tenemos problemas sin miedo a perder cambios realizados. 

Si estuviésemos trabajando con **Azure** es capaz de detectar cuando ha habido cambios en production y actualizar automáticamente el código, o si han cambiado las dependencias de package.json instalará automáticamente los nuevos paquetes. Esto lo puedes **hacer manualmente** con un script que haga **git pull --ff-only** periódicamente y si llega a traer nuevo código se encargue de instalar los nuevos paquetes y reiniciar el servidor.



### CAP 22 - Maintenance

Una vez la aplicación está online viene el trabajo no reconocido.

* Ten un plan de longevidad: El sitio tendrá una vida determinada (dice que como máximo 7 años para seguir siendo usable). Hay que hacer algo que se mantenga bien en el tiempo. Si el presupuesto es bajo habrá que hacer algo reutilizable con otros proyectos. Si el presupuesto es alto habrá que hacer algo excepcional. 
* Utilizar control de versiones: Saber quién y cuando hizo algo. Poder preguntar por qué.
* Utilizar un issue tracker: Registra todos los problemas que se han tenido, las dudas, quién lo arregló, el commit que se realizó para quitarlo... Y utiliza una herramienta que conlleve el mínimo trabajo.
* Mantener buena higiene: Utilizar todo lo anterior e ir haciendo revisiones periódicas para ver que mejoras se pueden ir sacando por problemas recurrentes. 
* No procastinar: Tener claro lo que se hace y si se repite varias veces en el tiempo ver la manera de automatizarlo. Cuestionarse las tareas.
* Chequear QA periódicamente: Tenerlos documentados para que puedan realizarse por una persona no técnica o incluso el cliente pueda leerlos. También es útil *https://www.google.com/webmasters* y *http://www.bing.com/toolbox/webmaster* para saber como ven el sitio los buscadores.
* Monitor de analíticas: Google Analitics es gratuito y da una idea de como están utilizando el sitio los usuarios. 
* Mejoras de rendimiento: Acelerar el sitio. Google Analitics también nos dará información sobre ello.
* A la hora de recoger datos: En los formularios mejor utilizar AJAX que JS ya que funcionará igualmente en caso de que deshabiliten JS (ten siempre un parámetro *action* en el form). Utilizar la URL del parámetro action: *$('form').on('submit', function(evt){ evt.preventDefault(); var action = $(this).attr(\'action'); /\* perform AJAX submission \*/ });.*. En caso de fallos está bien tener un nivel de redundancia bien en un fichero, en otra BBDD, en email... En caso de fallo total habría que hablar con el usuario para que bien lo intente luego o avise al staff. En vez de comprobar por un error busca una confirmación positiva (añade una propiedad *success*).
* Logging: Ocúpate de escribir lo que es útil, revisa lo logs que escriben lo que piensas. Ten costumbre de mirarlos e instruye a la gente a hacerlo. 

##### Reutilización de código
Bien a través de los **módulos** o de un **registro privado npm** en Node o del **middleware** en Express podemos hacer código reusable que sea realmente fácil de reutilizar. 

Como no se puede tener un package.json que tire de dos fuentes diferentes lo mejor que podemos hacer es utilizar **sinopia** que es un proxy que se pone entre medias y que detecta aquellos paquetes que son privados y que no tiene que ir al registro central de npm.

Para **crear un middleware** es tan fácil como (*meadowlark-stuff*):
```
module.exports = function(config){
        // it's common to create the config object
        // if it wasn't passed in:
        if(!config) config = {};

        return function(req, res, next){
            // your middleware goes here...remember to call next()
            // or next('route') unless this middleware is expected
            // to be an endpoint
            next();
        }
}
```

Y lo llamamos con:
```
var stuff = require('meadowlark-stuff')({ option: 'my choice' });

app.use(stuff);
```



### CAP 23 - Additional resources

Hemos tocado muchos palos pero aunque parezca bastante esto es solo la superficie. Para profundizar:

##### Documentación Online

* [MDN](https://developer.mozilla.org/en-US/#): Para documentación de JS, HTML y CSS. JS sin ninguna duda ahí.
* [Living Standard HTML5 Specification](http://developers.whatwg.org/): Para temas complicados de HTML.
* [Tabla compatibilidades](http://kangax.github.io/compat-table/es6/): Para saber que características de cada versión JS se soporta en los navegadores.

##### Suscripciones

* [JavaScript Weekly](javascriptweekly)
* [Node Weekly](https://nodeweekly.com/)
* [HTML5 Weekly](https://frontendfoc.us/)

##### Stack Overflow

Sitio de preguntas/respuestas técnicas basado en la reputación. Muy útil para resolver dudas (tanto lo que ya se ha preguntado como lo que puedes preguntar tu mismo) y para ganar reputación a base de responder preguntas.

##### Publicar/Contribuir 

Puedes contribuir en los proyectos de las herramientas que utilices a través de github. También puedes aportar middleware de Express/Connect a la comunidad a través de NPM (pero no significa que publiques cualquier cosa). 




### DUDAS: 

* ¿Por qué con node no se utiliza un servidor web por encima y con otros si que se utiliza un apache o lo que se necesite? ¿O estoy equivocado?
* Como se monta un subdominio para publicar APIs.