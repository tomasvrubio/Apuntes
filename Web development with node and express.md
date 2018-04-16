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



### CAP 11 -


### CAP 12 -


### DUDAS: 

* ¿Por qué con node no se utiliza un servidor web por encima y con otros si que se utiliza un apache o lo que se necesite? ¿O estoy equivocado?
