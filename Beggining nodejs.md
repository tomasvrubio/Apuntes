# BEGINNING node.js


## CAP 1: Setting Up for node.js Development

Nada reseñable. Es la instalación de node.


## CAP 2: Understanding node.js

Because of the importance of the Web and the pivotal role that JavaScript plays in web development, you can find a solution for most technical problems in some open source JavaScript project. Node.js allows you to use all these innovative JavaScript projects on the server the same as on the client browser. Using JavaScript on the server also reduces the context switching that needs to happen in your brain as you change programming language and associated code conventions. This is the emotional side of why you should use Node.js.

Una de las filosofías de node es que debe ser intuitivo para los desarrolladores front-end. *Es común llamar app.js al archivo principal* del proyecto para saber que script es el que debe correr para levantar la aplicación.

JS tiene mala reputación por:

* **DOM:** Inconsistente entre navegadores.
* **Errores:** Hay que tener cuidado con cómo maneja los errores ya que por si sólo trata de hacer que el código inválido funcione.

All functions return a value in JavaScript. In the absence of an explicit return statement, a function returns undefined.

A programming language is said to have **first-class functions** if a function can be treated the same way as any other variable in the language. JavaScript has first-class functions.


Nginx funciona mucho mejor con conexiones concurrentes que apache. *¿Y por qué es mucho más frecuente ver Apache?* Nginx utiliza un sólo hilo para ir despachando conexiones mientras que Apache tiene un pool de conexiones que va reutilizando.

**Callback:** Es una función que pasas a otra función para que la ejecute cuando termine. Es la manera que tiene JS para poder ejecutarse en un sólo hilo.

Node es muy eficiente y utiliza JS porque soporta “first-class” functions y closures.

El corazón de node es un event loop. Starvation es cuando nos quedamos sin recursos por estar atendiendo otros eventos que ha ejecutado el usuario. **Node no es la mejor opción si vas a tener que ejecutar tareas de alto consumo de CPU** por peticiones de usuario en un entorno de servidor multicliente. Lo que pasa es que muchas de estas tareas se hacen en otros servidores (por ejemplo servidor de BBDD) y lo que tiene que manejar node son eventos de network, que es donde brilla.

JS está hecho para consumir los recursos mínimos. Por ejemplo cuando copiamos variables realmente estamos referenciando unas entre otras.

Hay que tratar de utilizar siempre “===” en vez de “==”, aunque no hará falta si nos preocupamos de nunca comparar tipos de datos distintos para no vernos en una situación complicada.


**Prototipo:** Es compartido por las instancias creadas a partir de él. Con tocar ahí se cambia para todas las instancias. Además al tener las propiedades en un único lugar estamos ahorrando memoria. Aunque utilizar propiedades de prototipos no es bueno si se tiene pensado escribir sobre ellas, ya que verás directamente la propiedad del objeto y no del prototipo. 

JavaScript has a great **exception handling mechanism** that you might already be familiar with from other programming languages. To throw an exception, you simply use the throw JavaScript keyword. To catch an exception, you can use the catch keyword. For code you want to run regardless of whether an exception was caught or not, you can use the finally keyword. The catch section executes only if an error is thrown. The finally section executes despite any errors thrown within the try section. This method of exception handling is great for synchronous JavaScript. However, it will not work under an async workflow. 

Para poder manejar las excepciones en node y no tener crashes debemos manejar el error dentro de las funciones de callback.

```javascript
function getConnection(callback) {
    var connection;
    try {
        // Lets assume that the connection failed
        throw new Error('connection failed');

        // Notify callback that connection succeeded?
        callback(null, connection);
    }
    catch (error) {
        // Notify callback about error?
        callback(error, null);
    }
}

// Usage
getConnection(function (error, connection) {
    if (error) {
        console.log('Error:', error.message);
    }
    else {
        console.log('Connection succeeded:', connection);
    }
});
```

Having the **error as the first argument ensures consistency in error checking**. This is the convention followed by all Node.js functions that have an error condition.  


## CAP 3: Core node

Hay dos patrones diferentes en JS ya que **no hay las mismas preocupaciones en el navegador que en el servidor**, en uno la mayor preocupación será la **latencia de red** y en el otro la del **filesystem**.

El archivo principal en nodejs se llama *app.js*.

**Module system:**

* Each file is its own module.
* Each file has access to the current module definition using the module variable.
* The export of the current module is determined by the module.exports variable.
* To import a module, use the globally available require function.

**Función *require*:** Es la manera de importar un módulo en el fichero en el que se esté trabajando. Hay tres tipos de módulos: core modules, file modules y external node_modules.

Cuando ejecutamos *require* con un path relativo, nodejs ejecuta el js contenido en ese fichero y devuelve el valor final de module.exports que haya en ese fichero. **Características** de los módulos:

* **nodejs is safe:** Al usar require sólo cargamos las variables con module.exports y hay que asignar el resultado a una variable local para poder utilizarlo.
* **Conditionaly load a module**: Se pueden cargar los módulos bajo condiciones, ya que require funciona como cualquier otra función en JS.
* **Blocking:** Hasta que no se termina de cargar el módulo no continua con la ejecución.
* **Cached:** Cada vez que se llama a require, la parte obtenida con module.exports es cacheada por lo que ganamos velocidad en las siguientes ejecuciones.
* **Shared state:** Se puede compartir objetos en memoria entre módulos, lo que lo hace muy interesante para compartir información o parámetros de configuración.
* **Object Factories:** Un modulo se puede utilizar para generar objetos al llamar a require.

*Module.exports* se puede sustituir por *exports* a secas (alias). Solamente se puede utilizar este alias para que apunte a información y no asignarla ya que estaríamos pisando el enlace module.exports = exports.


**Mejores prácticas** para los módulos:

* No usar la extensión .js, es decir, poner sólo require(‘./foo’); Es para dar consistencia entre navegador y servidor.
* Usar los paths relativos cuando llamamos a los módulos basados en fichero.
* Usar el alias de exports. Es bueno también tener una variable local de aquello que exportamos:

```
var foo = exports.foo = //exportar lo que sea
```
	
* Para exportar una carpeta llena de módulos lo mejor que se puede hacer es generar un index.js en la carpeta que llame con su ruta relativa a todos los otros módulos y así ahorrar las múltiples llamadas con ruta más larga desde app.js.

```
Index.js:
	exports.foo = require('./foo');
	exports.bar = require('./bar');
	exports.bas = require('./bas');
	exports.qux = require('./qux');
```

Y ya sólo queda importar index: 
```
var something = require('../something/index');
```

**Variables globales:** *console* (Para mostrar lo que estamos haciendo a través de la terminal. Entrará mucho más en detalle en cap11), *timers* (tenemos setTimeout y setInterval para hacer llamadas pasado un tiempo). *\__filename* y *\__dirname*, *process* (En profundidad en cap5). Process.argv devuelve como se le ha llamado, el primero es node, el segundo el nombre del archivo js y después ya los argumentos. Hay librerías que permiten trabajar con los argumentos más fácilmente, y utilizaremos una en el siguiente capítulo. Process.nextTick lo que hace es ejecutar una función de callback en la siguiente vuelta del bucle de eventos. *buffer* (muy útil para manejar información a través de TCP o el filesystem).

La filosofía de diseño de node es dar un mínimo de módulos superprobados (core modules) y dejar a la comunidad construir sobre estos el resto de módulos con funcionalidades más avanzadas. Para llamarles se hace también con require pero sin tener que hacer uso del path:

```
var path = require(‘path’);  
```
	
**core modules** (vienen bien explicados en la documentación oficial de node):

* **path** – Permite trabajar con rutas abstrayendo de problemas con el SO. Útiles las funciones normalize, join, dirname, basename, extname.
* **fs** – Permite trabajar con filesystems. Tiene funciones para cambiar nombres, borrar, leer y escribir en ficheros. Tiene versión síncrona/asíncrona. Si utilizamos la versión síncrona estamos bloqueando el hilo de js hasta que se haya hecho la operación, por lo que siempre que sea posible es mejor utilizar la versión asíncrona.
* **os** – Funciones y propiedades del Sistema Operativo. Disponibilidad de memoria, cpu, … En cap13 se detalla al hablar de escalabilidad.
* **util** – Funciones de propósito general. Por ejemplo tiene util.log para poder pintar un mensaje de log con el timestamp. También tiene funciones para saber si algo es de un determinado tipo (isArray, isDate, isError).


**Reutilizar código de nodejs en el navegador:** Hay que tener en cuenta que los módulos a los que llamemos ya no estarán en el filesystem sino que estarán en la red, con lo que añadimos retardo. Para no quedarnos bloqueados lo mejor es hacer peticiones asíncronas (AMD) y para ello lo mejor es utilizar alguna librería que nos lo facilite,  siendo la más famosa requireJS.
Las características de AMD son:

* Los módulos son cacheados como en nodejs.
* Puedes hacer peticiones asíncronas que se ejecuten sólo bajo determinadas condiciones como también se podía hacer en node.

Browserify transforma el código node en código de navegador. No vale para todos los módulos de node ya que hay algunos que no existen en el navegador como es por ejemplo el de filesystem. El siguiente comando transforma el contenido de app en el fichero amdmodule:
```	
browserify app.js -o amdmodule.js	
```

## CAP 4: node Packages

Node como tal lleva pocos módulos incorporados para no dejar una única opción hacia los desarrolladores sino confiar en la habilidad de la comunidad open source de aportar distintos tipos de ideas. En este capítulo vamos a ver los external **node\_modules**. Se les llama igual que los core\_modules, pero si no existe un core\_module con ese nombre entonces busca en el apartado de node\_modules. Lo va a buscar en el directorio “node_modules” de todos los directorios padre hasta que consiga encontrarlo:

```
/home/ryo/project/node_modules/bar.js	
/home/ryo/node_modules/bar.js
/home/node_modules/bar.js
/node_modules/bar.js
```
Para llamar a los **folder-based modules**, para requerir a varios de un mismo directorio, podemos hacerlo explícitamente llamando a la carpeta y el sólo buscará un fichero *“index.js”* que referencie al resto de módulos del directorio.

**Ventajas** de utilizar *node\_modules* en vez de *folder\_based*:

* Simplificamos paths demasiado largos.
* Facilita el reutilizar módulos al sólo tener que copiar la carpeta que contiene el módulo.
* Reduce los efectos colaterales de tener que usar versiones determinadas ya que al utilizar primero el 	node_modules de la carpeta en que nos encontramos y luego ir yendo hacia atrás a través de las carpetas padre podemos ir de lo particular a lo general.

**Npm** utiliza ficheros JSON para configurar los módulos, dependencias, … (el ultimo objeto de un JSON no puede tener coma). Cada vez que tratamos de cargar un módulo, sino existe el fichero *“file”***.js** se tratará de cargar el fichero *“file”***.json**.

Hay un objeto global en js/node que permite convertir de JSON a otros tipos de datos y viceversa.

NPM permite compartir lo creado con el resto de la comunidad y también utilizar módulos creados por otras personas. Se basa en el fichero *“package.json”* para saber los módulos a utilizar y las versiones que ha de descargar. *“npm init”* permite inicializar un proyecto.

Cuando **instalamos un módulo**, para que quede apuntado en el package.json tenemos que utilizar la opción *–save*:
```
	npm install nombre_modulo –save
```

Para **refrescar** la carpeta node_modules de tu proyecto sólo debes utilizar:
```
npm install
```

Para **eliminar** una dependencia de un módulo (y eliminarlo de la carpeta node\_modules):
```
npm rm nombre_modulo –save
```

**Versionado semántico:** X.Y.Z

	X – Major version – Nuevas funcionalidades que lo hacen incompatible con anteriores versiones.
	Y – Minor version – Nuevas funcionalidades compatibles con versiones anteriores.
	Z – Patch version - Arreglos

Para **indicar la versión** a instalar:
```
	npm install nombre_modulo@"X.Y.Z”
```

Eso sería para una versión concreta del módulo. Si queremos **permitir todas las versiones de parches** ponemos *“~X.Y.Z”*, y si queremos **permitir todas las versiones menores** ponemos *“^X.Y.Z*”. Podemos utilizar también * para marcar **si queremos permitir todo de alguno de los puntos**: *X.Y.*, X.* , ** (ultima versión disponible en todo momento). Utilizando el –save sin poner ninguna versión específica pondrá la última estable y marcará con ^ para recibir todas las actualizaciones que no rompan la compatibilidad.

Con el comando *“npm outdated”* podemos **ver las versiones** que hay para un modulo. Y para actualizarlos haríamos un “npm update –save”.

Si a los comandos anteriores le añadimos un *“-g”* instalamos los **paquetes de manera global para utilizarlos en el sistema**.

Assume you require('something'). Then the follow is the logic followed by Node.js: 

* If something is a core module, return it. 
* If something is a relative path (starts with ‘./’ , ‘../’) return that file OR folder. 
* If not, look for node_modules/filename or node_modules/foldername each level up until you find a file OR folder that matches something. When matching a file OR folder, follow these steps: 
	* If it matched a file name, return it. 
	* If it matched a folder name and it has package.json with main, return that file. 
	* If it matched a folder name and it has an index file, return it. Of course, the file can be a file.js or file.json since JSON is first class in Node.js! 
	
For JSON, we return the parsed JSON and for JavaScript files we simply execute the file and return ‘module.exports’. That’s all there is to it. Having this knowledge allows you to open up and look at thousands of open source Node.js packages available on npmjs.org and Github.

Módulos famosos y bastante útiles:

* **Underscore:** Para poder utilizar métodos existentes en otros lenguajes. Muchas de las utilidades ya están incluidas en las últimas versiones de javascript. ¿Me merece la pena utilizarlo?
* **Optimist:** Te devuelve un objeto con los argumentos con los que se llamó al programa. En \_ vienen todos los argumentos que no empiezan con – (los flags). Los flags vienen como propiedades de argv con el valor marcado a true y si aceptan valor muestran su valor.

```
		var argv = require('optimist').argv;
		console.log(argv);
		$ node app.js foo bar bas -rfs -a 100
		{ _: [ 'foo', 'bar', 'bas'],
		  '$0': 'node /path/to/your/app.js', r: true, f: true, s: true, a: 100 }
```
* **Moment:** Permite trabajar con fechas más comodamente, formatearlas con el estilo necesario o poder ver la diferencia de tiempo entre varias fechas (mañana a tal hora, dentro de 6 horas, hace un día, …).

La manera de mandar fechas a través de JSON es mediante un string y se utiliza el ISO8601 como estándar entre países. Se hace relativo al UTC y con el formato: 2014-05-08T17:35:16Z. Para conseguir ese string utilizamos “.toJSON”, que vale tanto con fechas JS como con las de moment.

* **Colors:** Para cambiar el color con el que se pinta la información en la terminal.

```
console.log('hello'.green);	// outputs green text
console.log('world!'.red);	// outputs red text
```

## CAP 5: Events and Streams

Node está **centrado en tener un rendimiento excelente** a través de una manera sencilla de crear servidores de aplicaciones. Para ello los eventos y los streams tienen un rol importante. 

Para preparar la cadena de los **prototipos**:

Para forzar **this** lo hacemos con **“call”**:

```
var foo = {};
var bar = {};
// A function that uses `this`
function func(val) {
    this.val = val;
}
// Force this in func to be foo
func.call(foo, 123);
// Force this in func to be bar
func.call(bar, 456);
// Verify:
console.log(foo.val); // 123
console.log(bar.val); // 456
```

Cuando vayas a llamar al constructor del padre utilizas el patrón “Parent.call(this, /* additional args */)”:

```
function Bird(name){
    Animal.call(this,name);
    // Any additional initialization code you want
}
// Copy all Animal prototype members to Bird
```

Para acceder a los valores del padre sobre el que hemos construido el objeto utilizamos “constructor”:

```
function Foo() { }
var foo = new Foo();
console.log(foo.constructor.name); // Foo
console.log(foo.constructor === Foo); // true
```
	
Realmente para trabajar con la cadena de prototipos en node lo podremos hacer con “utils” que facilita la tarea.

```
// util function
var inherits = require('util').inherits;

// Bird Child class
function Bird(name) {
    // Call parent constructor
    Animal.call(this, name);

    // Additional construction code
}
inherits(Bird, Animal);
```
	
Para sobreescribir funciones en los hijos pero hacerlo reutilizando parte de la funcionalidad original debemos reutilizar el nombre de una de las funciones del prototipo y llamarlo de la siguiente manera:

```
// Overide parent behaviour in child
Child.prototype.foo = function () {
    // Call base implementation + customize
    return Base.prototype.foo.call(this) + " child foo";
}
```

Para comprobar que está en la cadena de prototipos usamos “instanceOf”:
```
	console.log(b instanceof B); // true because b.__proto__ == B.prototype
```

---

**Eventos**   
Ya hemos implementado eventos a través de callbacks pero en node es mucho más útil, al no saber nada de nuestros clientes, hacerlo a través de eventos en si mismo. Para ello tenemos el módulo **events**. 

```
var EventEmitter = require('events').EventEmitter;
var emitter = new EventEmitter();
// Subscribe
emitter.on('foo', function (arg1, arg2) {
    console.log('Foo raised, Args:', arg1, arg2);
});
// Emit
emitter.emit('foo', { a: 123 }, { b: 456 });
```
	
Una de las **ventajas de usar events es que puedes tener más de uno suscrito a un evento en sí**. Los que se suscriben se llaman listeners. Se irán ejecutando en el orden en el que fueron registrados. Para dejar de estar suscritos llamamos a ***“removeListener”*** (hay que pasarle el nombre del evento a escuchar y la función a la que llama). 

Con ***“emitter.once(‘evento’, funcion())”*** podemos ver lo que pasa cuando se genera el evento pero sólo la primera vez. 

Con ***“emitter.listeners(‘evento’)”*** vemos todos los listeners suscritos a un evento. 

**Una manera común de tener fugas de memoria es suscribirse a eventos en callbacks pero olvidarse de desuscribirse**. Por defecto se aceptarán 10 listeners por cada tipo de evento y luego se empezará a avisar con un mensaje de warning de que se está superando el límite. Aún así el código seguirá funcionando y nos podremos seguir suscribiendo a eventos. Para quitar ese mensaje habrá que modificar el parámetro
***“emitter.setMaxListeners(num_max_deseado)”***, 0 para ser ilimitado. Otra forma de evitar estas fugas de memoria es **creando los eventos en la función de callback con un nuevo emitter y así será eliminado cuando finalice la función**. 

**Error event:** Si no hay un listener para el mismo lo que se hace es pintarlo y salir del programa. Por lo tanto sólo hay que levantar eventos de error en situaciones excepcionales y debe ser manejado. 

**Eventos propios:** Se crean con emit y luego ya los capturamos con on. 

**Eventos de Process:**

* **uncaughtException:** Cuando el servidor ha entrado en un estado inconsistente. Lo mejor es dejar una traza en el log y salir.
process.on('uncaughtException', function (err) {
    		console.log('Caught exception: ', err);
    		console.log('Stack:', err.stack);
		});
* **exit:** No se puede abortar el salir del programa tras haber recibido este evento ni realizar operaciones asíncronas. Su funcionalidad es sobretodo para escritura de log. 
* **signals:** Para utilizar las señales de sistema. 

---

**Streams**   
Juega un papel muy importante en conseguir hacer **servidores de aplicación de alto desempeño**. Los hay de **4 tipos** (están contenidos en el módulo stream que viene de serie):

* **Readable:** Por ejemplo la entrada del usuario (process.stdin).
* **Writable:** Por ejemplo la salida process.stdout. 
* **Duplex:** El socket de la red, en el que puedes escribir datos y leerlos. 
* **Transform:** Un caso especial de un stream duplex en el que la salida está tratada respecto a la entrada. Un ejemplo son los streams de compresión o encriptado.

Están basados en eventos. Antes de ver como creamos nuestros streams vemos algunos **de la propia librería de node**:

* **pipe:** Permite mandar el contenido de un stream a otro. Se llama así porque imita la “|” de linux. Están basados en eventos pero de momento no hace falta entrar más en detalle.
* **Consumir los streams de lectura:** A través del evento readable hasta que recibe un null.
* **Escribir en streams de escritura:** A través de write y cuando ya no queda nada que enviar llamamos a end. 

**Crear tu propio stream:** Muy similar a crear tu propio eventEmitter. Se hace desde la clase stream base (Readable, Writable, Duplex o Transform) y se le implementan métodos base en función del tipo que sea (_read, _write, _transform, _flush). 



## CAP 6: Getting started with HTTP

Node fue **especialmente creado para hacer aplicaciones de red escalables** para la parte de servidor. El core de node contiene lo suficiente para que se pueda construir sobre él potentes aplicaciones. Los **módulos principales** que trae son:

* **net:** Para crear servidores TCP y clientes.
* **dgram:** Para crear UDP y sockets.
* **http:** Para trabajar con la capa http.
* **https:** Para crear conexiones SSL/TLS.

**http**   
Empezamos con http para crear un servidor estático. Para ello utilizamos la funcion *createServer*, que para cada petición de un cliente utiliza la petición entrante y la salida que vuelve hacia él. Para arrancar simplemente hay que escuchar en el puerto deseado.

```
var http = require('http');
var server = http.createServer(function (request, response) {
    console.log('request starting...');
    // respond
    response.write('hello client!');
    response.end();
});
server.listen(3000);
console.log('Server running at http://127.0.0.1:3000/');
```

Para probarlo:

```
$ curl http://127.0.0.1:3000
hello client!
```

Si hacemos log de la cabecera de la petición (request.header):

```
$ curl http://127.0.0.1:3000 -i
HTTP/1.1 200 OK
Date: Thu, 22 May 2014 11:57:28 GMT
Connection: keep-alive
Transfer-Encoding: chunked
hello client!
```

**Debugging proxy:** Permanece entre el cliente y el servidor de node. Uno famoso es *fiddler*, aunque en internet veo que se podria utilizar *wireshark*. Para pasar por el proxy tenemos que utilizar
```
$ curl http://127.0.0.1:3000 -x 127.0.0.1:8888
```

Si vemos con ello la respuesta veremos:

* **d:** Avisando de que viene un chunk de datos de 13 bytes (d hexadecimal = 13) "hello client!"
* **0:** Avisando de que viene un chunk de 0 bytes, es decir, que ha terminado.

**Miembros principales de la respuesta:** *Headers* y *Body*. Las cabeceras han de indicar como se va a enviar la información en la respuesta y llegar antes que el resto de datos al cliente (que sería un stream de escritura) para que sepa como debe interpretarlos. En cuanto llamamos *response.write* o *.end* se envían las cabeceras y podemos comprobar que se han enviado utilizando *response.headerSent* a true. Con *statusCode* podemos indicar el código de estado que se enviará. Puedes indicar cualquier punto de la cabecera con *response.setHeader(name, value)*, como por ejemplo “Content-Type” que indica como debe interpretar lo que se le envía al cliente (HTML, CSS, JS, JSON, JPEG, PNG…). Para saber el valor que necesita un fichero puedes utilizar el módulo *mime* de npm. Puedes ver el contenido de una cabecera con *getHeader* y quitar una cabecera con *removeHeader*.

También puedes enviar directamente la cabecera para luego encargarte del cuerpo a continuación. Se hace mediante (le podríamos añadir los parámetros que deseemos):

```
response.writeHead(200, { 'Content-Type': 'text/html' });
```

**Miembros principales de la petición:** La petición también es un stream (de lectura), que es interesante cuando el cliente quiere mandar más información al servidor como pueder ser el caso de subir un fichero. También está dividida en una cabecera y un cuerpo. Podemos ver cualquier parte de la cabecera con *request.headers[‘propiedad’]*. Una información clave a extraer de las cabeceras es el método (*request.method*) y la url (*request.url*) que ha usado el cliente.

**Connect**   
Uno de los módulos que construye sobre estos módulos básicos es connect. Es un **framework que se sitúa entre nuestra aplicación y las apis de bajo nivel http**. El corazón de este módulo es la función *connect()*, que crea un dispatcher que toma como argumentos request y response, por lo que pasará a ser el argumento de createServer.

```
var connect = require('connect')
    , http = require('http');
// Create a connect dispatcher
var app = connect();
// Register with http
http.createServer(app)
    .listen(3000);
console.log('server running on port 3000');
```

Eso hará que por defecto connect devuelva un 404 ante cualquier petición. Al ir añadiendo middleware que contemple las distintas peticiones irá devolviendo lo que corresponda. Realmente el código necesario para arrancar el servidor (enmascara la creación) es:

```
var connect = require('connect');
// Create a connect dispatcher and register with http
var app = connect()
		.listen(3000);
console.log('server running on port 3000');
```

Para crear un middleware lo haremos con *use*, situándolo antes de *“.listen”*. De la forma (podemos ir encadenando use uno tras otro):
```
.use(function (req, res, next) { ... })
```

Si queremos que se ejecute un middleware sólo para un determinado path:
```
.use(‘/path_deseado’, function…)
```

El **poder de encadenar** (con la función “next()” pasamos al siguiente middleware):

* **Compartir información petición/respuesta:** Si por ejemplo detectamos que la petición que ha entrado al servidor es un json (por la cabecera) podemos parsear la información entrante y volver a poner en el req.body para que se trate con next por el siguiente middleware.
* **Verificar acceso:** Desde un use podemos llamar a la siguiente función que sólo dejará paso al siguiente middleware en caso de que te autentiques (devolviendo una cabecera en la respuesta indicando autenticación Basic en el parámetro WWW-Authenticate):

```
function auth(req, res, next) {
		function send401(){
			res.writeHead(401 , {'WWW-Authenticate': 'Basic'});
			res.end();
		}
		var authHeader = req.headers.authorization;
		if (!authHeader) {
			send401();
			return;
		}
		var auth = new Buffer(authHeader.split(' ')[1], 'base64').toString().split(':');
		var user = auth[0];
		var pass = auth[1];
		if (user == 'foo' && pass == 'bar') {
			next(); // all good
		}
		else {
			send401();
		}
}
```
Si esto sólo queremos que sea para una ruta de la web podríamos hacerlo encadenando de la siguiente manera:
```
connect()
	.use('/admin', auth)
	.use('/admin', function (req, res) { res.end('Authorized!'); })
	.use(function (req, res) { res.end('Public') })
	.listen(3000);
```

* **Levantar un error:** Si ponemos *next(new Error(‘El texto del error’))* no va a entrar en el siguiente middleware sino que se enviará hacia el cliente con un código HTTP 500 INTERNAL SERVER ERROR. Si quieres un middleware que actúe en función de los errores levantados por otros lo haríamos con (el objeto err tiene todo el contenido del error en message, stack, …):

```
	.use(function (err, req, res, next) { … })
```

**https**   
Mientras que para otros frameworks ha sido difícil de implementar, node no tiene ningún problema y está diseñado para soportarlo. *Se basa en la existencia de una clave pública (que conoce todo el mundo) y una clave privada (que solamente tienes tú)*. La pública se utiliza para encriptar y la privada para desencriptar.

Con un programa como **openssl** generamos la clave privada, en el que tendremos que seleccionar un algoritmo de encriptado y su complejidad. En openssl también podremos crear la clave pública.

Una vez tengamos las claves hay un módulo en node similar al que ya utilizábamos antes con http, ‘https’, que toma un primer parámetro “options” que podemos utilizar para pasar las claves:

```
var https = require('https');
var fs = require('fs');
var options = {
    key: fs.readFileSync('./key.pem'),
    cert: fs.readFileSync('./cert.pem')
};
https.createServer(options, function (req, res) {
    res.end('hello client!');
}).listen(3000);
```

Como el uso del módulo es similar a http, lo podemos utilizar sin problemas con connect:

```
// Create a connect dispatcher
var app = connect();
// Register with https
https.createServer(options, app)
    .listen(3000);
```
    
**Si es un sitio público tendremos que utilizar un certificado SSL de una autoridad certificadora** (verisign, thawte). Sin su existencia podría haber un *ManInTheMiddle* entre cliente y servidor.

**Siempre que sea posible hay que utilizar https** y redirigir todo el tráfico que nos venga por http a https. **Cuando no especificas un puerto al lanzar una url, con http va al puerto 80 y con https va al puerto 443.** Se trata de tener nuestra aplicación levantada en el 443 con https y en el 80 tener un servidor http que redirija el tráfico al servidor https.

```
var https = require('https');
var fs = require('fs');
var options = {
    key: fs.readFileSync('./key.pem'),
    cert: fs.readFileSync('./cert.pem')
};
https.createServer(options, function (req, res) {
    res.end('secure!');
}).listen(443);
// Redirect from http port 80 to https
var http = require('http');
http.createServer(function (req, res) {
    res.writeHead(301, { "Location": "https://" + req.headers['host'] + req.url });
    res.end();
}).listen(80);
```

Con la respuesta 301 estamos haciendo que el navegador del cliente se redirija a la URL que le indicamos en la respuesta. Este servidor solamente se puede levantar con “sudo node”. Eso es debido a que **sólo el superusuario puede levantar servidores contra el puerto 80 y 443**.

Si toda la comunicación es a través de HTTPS podemos utilizar formularios HTTP para enviar información desde el cliente hacia el servidor sin vulnerar los datos.

***Connect* está más relacionado con aplicaciones HTTP** y ***ExpressJS* está más enfocado a la programación de sitios web** (es del mismo equipo que connect). Realmente **todo el middleware de connect es reutilizable por express** ya que utiliza también “use”.


## CAP 7: Introducing Express

**El uso es similar a connect.** Todo lo que hemos hablado en el anterior capítulo de middleware, conexiones http/https, gestión de errores… es similar en express.

```
var express = require('express'),
     http = require('http');
// Create an express application
var app = express()
            // register a middleware
            .use(function (req, res, next) {
                res.end('hello express!');
            });
// Register with http
http.createServer(app)
    .listen(3000);
```
    
**Todo el middleware de connect funciona en express pero no al contrario.**

**Middleware** creado por el equipo:

* **Páginas estáticas:** Podemos servir páginas de un directorio sin tener que contemplar la autorización de directorios, manejo de errores, redirecciones. Muestra el index.html de un directorio si el path resuelve un directorio.

```
npm install serve-static
```

```
var express = require('express');
var serveStatic = require('serve-static');
var app = express()
    .use(serveStatic(__dirname + '/public'))
    .listen(3000);
	Realmente no hace falta instalar el módulo ya que static es parte del core de express:

var express = require('express');
var app = express()
    .use(express.static(__dirname + '/public'))
    .listen(3000);
```
    
* **Listar directorios:** Con serve-index. Es habitual utilizarlo con static ya que permite al usuario ver el fichero seleccionado. Hay que llamar antes a static que a serveIndex para que si se ha solicitado un fichero index se muestre en vez de sacar el listado.

```
npm install serve-index
```

```
var express = require('express');
var serveIndex = require('serve-index');
var app = express()
    .use(express.static(__dirname + '/public'))
    .use(serveIndex(__dirname + '/public'))
    .listen(3000);
```

* **Peticiones JSON / Entradas de formulario HTTP:** Es muy típico parsear la información que nos envía el cliente para convertirlo en un objeto JS y por tanto es muy frecuente utilizar el módulo “body-parser”.
	
```
npm install body-parser
```

El módulo hace lo siguiente:

1. Parsea a un objeto JS si el content-type es un JSON (application/JSON) o si es un formulario HTML (application/x-www-form-urlencoded).
2. Pone ese objeto en el req.body para poderlo pasar al siguiente middleware.

Como ejemplo:

```
var express = require('express');
var bodyParser = require('body-parser');

var app = express()
    .use(bodyParser())
    .use(function (req, res) {
        if (req.body.foo) {
            res.end('Body parsed! Value of foo: ' + req.body.foo);
        }
        else {
            res.end('Body does not have foo!');
        }
    })
    .use(function (err, req, res, next) {
        res.end('Invalid body!');
    })
    .listen(3000);
```

Si no trae ninguno de esos datos construye un JSON vacío y continua. En cambio, si el JSON que trae la petición no es válido devuelve un error. Por ejemplo también te securiza la cantidad de información que puede mandar el usuario (por defecto 100KB). Para cambiar esa cantidad máxima llamaríamos de la forma bodyParser({limit:’1mb’}).

* **Cookies:** Las cookies se utilizan para crear sesiones de usuario. Son datos que son enviados por el servidor y son almacenados en el navegador del usuario. Cada vez que el usuario hace una petición al servidor, el navegador web envía esa cookie hacia el servidor.
Para indicar el valor de una cookie con express utilizamos:

```
res.cookie(cookieName, value, [options]) → res.cookie(‘name’,’foo’);
```

Es más útil utilizar el módulo cookie-parser:

```
npm install cookie-parser
```

```
var express = require('express');
var cookieParser = require('cookie-parser');
var app = express()
    .use(cookieParser())
    .use(function (req, res) {
        if (req.cookies.name) {
            console.log('User name:', req.cookies.name);
        }
        else {
            res.cookie('name', 'foo');
        }
        res.end('Hello!');
    })
    .listen(3000);
```
   
Para **eliminar cookies** podemos utilizar *res.clearCookie('cookieName')*.

**Para prevenir que el usuario modifique el contenido de la cookie se signa en el servidor (HMAC)**. Ese número sólo lo conoce el servidor por lo notaría cambios que se le han hecho (lo hace cookie-parser automáticamente). En caso de que **queramos que esté firmada**:

```
var express = require('express');
var cookieParser = require('cookie-parser');
var app = express()
    .use(cookieParser('my super secret sign key'))
    .use('/toggle', function (req, res) {
        if (req.signedCookies.name) {
            res.clearCookie('name');
            res.end('name cookie cleared! Was:' + req.signedCookies.name);
        }
        else {
            res.cookie('name', 'foo', { signed: true });
            res.end('name cookie set!');
        }
    })
    .listen(3000);
```
    
**Para que no pueda ser leída la cookie por un js** (y que se pueda hacer un ataque XSS) lo mejor es permitir exclusivamente que pueda ser leída por el servidor:
```
	res.cookie(name,value,{httpOnly:true})
```

Además para, hacer que **sólo se manden cookies a través de cabeceras HTTPS** y, que no sean vulnerables (es necesario estar utilizar un servidor HTTPS):
```
	res.cookie(name,value,{secure:true}
```

Para indicar **el tiempo que queremos que dure una cookie** (en milisegundos):
```
	res.cookie('foo', 'bar', { maxAge: 900000, httpOnly: true })
```

* **Sesiones basadas en cookies:** El módulo cookie-session permite usar cookies para almacenar información de la sesión del usuario.

```
var express = require('express');
var cookieSession = require('cookie-session');
var app = express()
    .use(cookieSession({
        keys: ['my super secret sign key']
    }))
    .use('/home', function (req, res) {
        if (req.session.views) {
            req.session.views++;
        }
        else{
            req.session.views = 1;
        }
        res.end('Total views for you: ' + req.session.views);
    })
    .use('/reset',function(req,res){
        delete req.session.views;
        res.end('Cleared all your views');
    })
    .listen(3000);
```
    
También se puede **borrar toda la sesión del usuario** con *req.session=null*. A cookieSession, además de keys, le podemos pasar name, maxage, path, httpOnly, signed.

**Hay que tener cuidado con las cookies porque se mandan en cada petición que hace el usuario**, por lo que si crecen mucho harán que el rendimiento sea menor. También los navegadores ponen límites (normalmente 4093bytes y max 20 cookies por sitio), por lo que no se puede tener toda la información del usuario como una cookie. **Lo más inteligente es guardar solamente un token de usuario y utilizarlo para acceder a sus datos a la BBDD** (lo veremos en el tema 8).

* **Compresión:** Se trata de comprimir las páginas antes de enviárselas al usuario. Es el módulo **compression** *(npm install compression)*. Funciona para páginas mayores a 1 kb. Para cambiar ese valor habría que modificar la opción threshold (compression({threshold: 512}) para poner 512b).

```
var express = require('express');
var compression = require('compression');
var app = express()
    .use(compression())
    .use(express.static(__dirname + '/public'))
    .listen(3000);
```

* **Manejar timeouts:** Utilizando el módulo **connect-timeout** *(npm install connect-timeout)* podemos manejar las situaciones en que un módulo falle y no se llegue a ejecutar next (por ejemplo si la BBDD está caída). Si pasa el tiempo indicado mandará un 503 Service Unavailable HTTP response.

```
var express = require('express');
var timeout = require('connect-timeout');
var app = express()
    .use('/api', timeout(5000),
                function (req, res, next) {
                    // simulate a hanging request by doing nothing
                })
    .listen(3000);
```
    
**Este módulo no debería ser usado desde la raíz (‘/’) ya que seguramente haya respuestas que tarden más tiempo en alguna parte de la aplicación.** También hay que tener cuidado con los middleware que no hayan fallado pero simplemente hayan tardado más de lo que indica el timeout. **Para los que sean susceptibles de tardar más tiempo del indicado habrá que preparar una función que se encargue de terminar el middleware.**

```
var express = require('express');
var timeout = require('connect-timeout');
var app = express()
    .use(timeout(1000))
    .use(function (req, res, next) {
        // simulate database action that takes 2s
        setTimeout(function () {
            next();
        }, 2000)
    })
    .use(haltOnTimedout)
    .use(function (req, res, next) {
        res.end('Done'); // Will never get called
    })
    .listen(3000);
function haltOnTimedout(req, res, next) {
    if (!req.timedout) next();
}
```

**Objeto de respuesta de Express**   
La respuesta de Express deriva de la respuesta estándar de node pero además le añade unas cuantas funciones que lo hace más útil. Por ejemplo *res.status(codigo)* para devolver el código que queramos además de que es encadenable, *res.set* para indicar a la vez múltiples cabeceras de la respuesta, *res-get(‘nombre-cabecera’)* para obtener el valor de una cabecera, *res.type(type)* para indicar el content-type o bien indicarle una extensión para que lo haga el sólo, *res.redirect([codigo], url)* para indicar redirecciones,

Para simplificar la respuesta debemos utilizar *res.send([codigo], [body])* ya que en una misma línea nos permite mandar código de respuesta y el contenido. Lo podemos utilizar para mandar sólo códigos de respuesta o bien para devolver JSON ya que el sólo adaptará el content-type.

**Objeto de petición de Express**   
Ocurre similar que con el de respuesta, tenemos ciertas funciones añadidas. Con *req.get(nombre-cabecera)* accedemos sin preocuparnos de minúsculas/mayúsculas a la cabecera que necesitamos, comprobar el tipo con *req.is(tipo)*, ip del cliente con *req.ip*, comprobar el protocolo con *req.secure*. Para manejar elementos que vengan en la URL podemos utilizar *req.query* para obtener los parámetros que se han introducido en la petición, *req.path* para sólo obtener la ruta de la petición, *req.url* que te da la url pero comenzando en el punto en el que empezase el middleware (si por ejemplo empiezan en ‘/home’ y la petición es ‘/home/tomas’ sólo aparecería ‘/tomas’), *req.originalUrl* daría la completa siempre.

Para acceder a estos objetos en caso de ser necesario debugar podemos hacerlo mediante:

* **req.res:** Objeto de respuesta
* **res.req:** Objeto de petición

**REST**   
REST, Representional State Transfer, es un **término para indicar un tipo de arquitectura en el que se indica como se deben conectar los componentes**. Las *apis que mantienen esto se denominan RESTful*. Hay dos tipos de URLs:

* Las que apuntan a **colecciones**.
* Las que apuntar a **elementos**.

Para estos dos tipos de URL hay que funcionar de un modo determinado al utilizar los métodos PUT, GET, POST y DELETE.
 
**Para colecciones:**

HTTP method | Behavior
----------- | ------------
GET         | Get the summarized details of the members of the collection, including their unique identifiers.
PUT         | Replace the entire collection with a new collection.
POST         | | Add a new item in the collection. It is common to return a unique identifier for the created resource.
DELETE      | Delete the entire collection

**Para elementos:**

HTTP method | Behavior
----------- | ------------
GET         | Get the details of the item.
PUT         | Replace the item.
POST        | Would treat the item as a collection and add a new sub-item in the collection. It is not generally used as you tend to simply replace properties on the item as a whole (in other words, use PUT).
DELETE      | Delete the item.

Con express se puede hacer **app.get / app.put / …** para ir registrando los distintos middlewares. Se puede utilizar **app.all** para todos los métodos.

```
var express = require('express');
var app = express();
app.all('/', function (req, res, next) {
    res.write('all\n');
    next();
});
app.get('/', function (req, res, next) {
    res.end('get');
});
app.put('/', function (req, res, next) {
    res.end('put');
});
app.post('/', function (req, res, next) {
    res.end('post');
});
app.delete('/', function (req, res, next) {
    res.end('delete');
});
app.listen(3000);
```

**Para no tener que especificar la ruta en cada una de las llamadas a app podemos hacerlo con route** y que cuelguen todos de un mismo directorio:

```
app.route('/')
    .all(function (req, res, next) {
        res.write('all\n');
        next();
    })
    .get(function (req, res, next) {
        res.end('get');
    })
```
    
Con *app.* no hay que indicar rutas exactas sino que se pueden utilizar expresiones regulares. Por ejemplo:

```
var express = require('express');
var app = express();
app.get('/', function (req, res) {
    res.send('nothing passed in!');
});
app.get(/^\/[0-9]+$/, function (req, res) {
    res.send('number!');
});
app.get('/*', function (req, res) {
    res.send('not a number!');
});
app.listen(3000);
```

**Podemos enrutar y obtener parámetros** de la url de la manera *‘/ruta/:parametro’*, donde express pondrá los parámetros en **req.params**.

```
var express = require('express');
var app = express();
app.get('/user/:userId', function (req, res) {
    res.send('userId is: ' + req.params['userId']);
});
app.listen(3000);
```

También podemos utilizar *app.param (‘nombre_parametro’, function(req, res, next, nombre_parametro){...})* para que sea **llamado cada vez que case una variable en una ruta** como el get anterior.

**Objeto Router:** Podemos crear mini-aplicaciones express con Router que nos facilita el trato con middleware + rutas. Un ejemplo:

```
var express = require('express');
var bodyParser = require('body-parser');
// An in memory collection of items
var items = [];
// Create a router
var router = express.Router();
router.use(bodyParser());
// Setup the collection routes
router.route('/')
      .get(function (req, res, next) {
          res.send({
              status: 'Items found',
              items: items
          });
      })
      .post(function (req, res, next) {
          items.push(req.body);
          res.send({
              status: 'Item added',
              itemId: items.length - 1
          });
      })
      .put(function (req, res, next) {
          items = req.body;
          res.send({ status: 'Items replaced' });
      })
      .delete(function (req, res, next) {
          items = [];
          res.send({ status: 'Items cleared' });
      });
// Setup the item routes
router.route('/:id')
    .get(function (req, res, next) {
        var id = req.params['id'];
        if (id && items[Number(id)]) {
            res.send({
                status: 'Item found',
                item: items[Number(id)]
            });
        }
        else {
            res.send(404, { status: 'Not found' });
        }
    })
    .all(function (req, res, next) {
        res.send(501, { status: 'Not implemented' });
    });
// Use the router
var app = express()
            .use('/todo', router)
            .listen(3000);
```
            
**Lo bueno de usar routers es que se pueden replicar fácilmente en otros sitios.** Es muy reusable y mantenible. Los routers funcionan con req.url por lo que comienzan en el punto de montaje (/).

[Articulo original](http://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm) donde se hacía mención a REST.



## CAP 8: Persisting Data

**NoSQL**   
*Not Only SQL.* Agrupa al movimiento de nuevas BBDD que vienen a cubrir huecos que no rellenan las bases de qdatos típicas SQL. Hay cuatro grupos:

* BBDD de documentos (mongoDB)
* BBDD clave-valor (redis)
* BBDD de columnas (Cassandra)
* BBDD gráficas (neo4j)

La principal motivación de todas ellas es la **escalabilidad**. Para casi todos los propósitos la que mejor cumple es la de documentos pero para hacer cosas sencillas las BBDD de clave-valor son las que ofrecen mejor rendimiento.

Una **BBDD de documentos** funciona basándose en el concepto de documentos, siendo este un trozo de información autocontenida para una particular entidad. Estos documentos pueden ser JSON, XML, binarios, …

Un **almacenamiento clave-valor** es un versión recortada de una BBDD de documentos. La clave es el valor para identificar el documento y el valor es el contenido del mismo. Lo que la diferencia es que la mayoría de veces en esta BBDD sólo se puede consultar por clave. En una BBDD de documentos si que podrías consultar por el contenido de los documentos.

**¿Por qué NoSQL?**

* **Escalabilidad:** Las relaciones entre tablas hace muy difícil el separar partes de una BBDD en diferentes nodos en distintos sitios. En cambio una BBDD basada en documentos facilita en gran medida esto.
* **Facilidad de desarrollo:** Para transferir datos entre la aplicación y la BBDD en una SQL hay que convertir las tablas resultantes en objetos y viceversa (ORM), mientras que en una de Documentos hay APIs para directamente recibir JSONs.

**MongoDB**   
Tiene dos ejecutables: **mongod** que se utiliza para arrancar la BBDD y **mongo** que es una consola JS para poder lanzar querys y administrarla.

**Jerarquía:** Un despliegue de mongo consiste en  múltiples BBDD. Cada BBDD puede contener múltiples colecciones. Cada colección  puede contener múltiples documentos.

**Un Documento es un documento JSON** que tiene algunos añadidos (como que soporta nativamente el campo Date). Una **agrupación de documentos recibe el nombre de Colección**, que no hace obligatorio que todos los documentos que contenga tengan la misma estructura. Finalmente un servidor mongo permite tener **varias BBDD, permitiendo una separación lógica de colecciones en el servidor**. Se utilizaría una BBDD por cada cliente (habría que ver a que se refiere con cliente aquí), teniendo dentro de ellas los mismos nombres para las colecciones.

**Cada documento debe tener un campo “\_id”**, pudiendo tener cualquier valor mientras sea único en la colección. MongoDB pone por defecto un ObjectId (como ejemplo "_id" : ObjectId("539ed1d9f7da431c00026e17")). Este identificador **no es una clave primaria natural**, como las de SQL, sino que puede variar a lo largo del tiempo  (surrogate primary key).
ObjectId: No es un número autoincremental. Tiene 12bytes y el siguiente contenido:

>Los campos que tiene son *time* segundos desde EPOCH, *machine* con un hash de la máquina, *pid* hace referencia al proceso que generó el objeto e *inc* que es un número incremental que permite que un mismo proceso en una misma máquina para un mismo segundo pueda generar 256³ ObjectId únicos. Para casi cualquier caso no te deberías preocupar por que sean únicos.

La manera de **crear un ObjectId** es a través del operador new y tiene una función para saber cuando fue creado:

```
$ mongo --nodb
MongoDB shell version: 2.6.1
> var id = new ObjectId()
> id
ObjectId("53a02d3979d8322ea34c4179")
> id.getTimestamp()
ISODate("2014-06-17T11:57:45Z")
```

**Formato del Documento**   
Es guardado internamente utilizando **BSON (Binary JSON)**. También utilizan este formato los clientes para las transferencias por la red. Una cualidad imprescindible de este formato es que **ofrece prefijos de longitud**, teniendo uno cada valor antes de aparecer. De esta manera podemos saltarnos un dato que no nos interese. Ademas también **contiene información sobre el tipo de campo del valor** (number, string, boolean) con lo que facilita el parseado. Además BSON **incluye tipos primitivos que no están en JSON como son el UTC Datetime, raw binary y ObjectId**.


**Ejemplo de uso con nodejs**   
Para utilizarlo instalamos el paquete mantenido por el mismo equipo de Mongo
```
(npm install mongodb)
```

El módulo sigue la convención típica de node y la principal conexión es a través de la clase **MongoClient** exportada del módulo.

Para crear un objeto (demoPerson), buscarlo mediante una query (buscando por name=’John’) y borrarlo (utilizando el name también) podemos ver el siguiente ejemplo:

```
var MongoClient = require('mongodb').MongoClient;
var demoPerson = { name: 'John', lastName: 'Smith' };
var findKey = { name: 'John' };
MongoClient.connect('mongodb://127.0.0.1:27017/demo', function (err, db) {
    if (err) throw err;
    console.log('Successfully connected');
    var collection = db.collection('people');
    collection.insert(demoPerson, function (err, docs) {
        console.log('Inserted', docs[0]);
        console.log('ID:', demoPerson._id);
        collection.find(findKey).toArray(function (err, results) {
            console.log('Found results:', results);
            collection.remove(findKey, function (err, results) {
                console.log('Deleted person');
                db.close();
            });
        });
    });
});
```

Para hacer un **Update** hay dos maneras:

* **Función save:** Lo que hace es sobreescribir todo el documento. Es útil pero si solo queremos modificar un campo no tiene mucho sentido reescribir todo el documento (ineficiente).
* **Función update:** Sólo modifica el campo que deseamos cambiar en el documento. Se puede cambiar un sólo campo, borrar campos, modificar un campo  si el valor que contiene cumple cierta condición, modificar arrays…

**ODM – *Object Docment Mapper*:** El más famoso es **mongoose**, con el que podemos mapear de los documentos a objetos JS que incluyan los datos y los métodos con los que contarán.

```
var mongoose = require('mongoose');
mongoose.connect('mongodb://127.0.0.1:27017/demo');
var db = mongoose.connection;
db.on('error', function (err) { throw err });
db.once('open', function callback() {
    console.log('connected!');
    db.close();
});
```

El corazón de mongoose es la clase **schema**, que define todos los campos del documento con sus tipos y su comportamiento. Sobre ellos se crean los models que son los constructores para convertir de literales de objeto a JSON. Definimos methods sobre los schemas para poder operar sobre las instancias creadas, es decir, son funciones.

```
var tankSchema = new mongoose.Schema({ name: 'string', size: 'string' });
tankSchema.methods.print = function () { console.log('I am', this.name, 'the', this.size); };
// Compile it into a model
var Tank = mongoose.model('Tank', tankSchema);
var tony = new Tank({ name: 'tony', size: 'small' });
tony.print(); // I am tony the small
```

También todas las instancias tienen funciones para operar con la BBDD. Por ejemplo:

```
tony.save(function (err) {
  if (err) throw err;
  // saved!
})
```

Para **ejecutar queries** se hace con *find* y *findOne*. Es importante saber que son encadenables para crear queries más complejas.

```
Person
.find({ city: 'LA' })
.where('name.last').equals('Ghost')
.where('age').gt(17).lt(66)
.limit(10)
.exec(callback);
```

**MongoDB para almacenar sesiones**   
En vez de utilizar cookie-session pasamos a utilizar **express-session**. En principio es igual y almacena en memoria del servidor los datos de sesión. El problema está en que **si reiniciamos el servidor perdemos la información de cada usuario**. Para poder mantener los datos de cada sesión vamos a tener que almacenar en local algo de información y aquí es donde entra mongo. **Tendremos que utilizar la opción de configuración store y el módulo connect-mongo**. Para utilizarlo conseguimos una referencia a MongoStore y creamos una conexión a la BBDD para guardar la sesión.

```
var express = require('express');
var expressSession = require('express-session');
var MongoStore = require('connect-mongo')(expressSession);
var sessionStore = new MongoStore({
    host: '127.0.0.1',
    port: '27017',
    db: 'session',
});
var app = express()
    .use(expressSession({
        secret: 'my super secret sign key',
        store: sessionStore
    }))
    .use('/home', function (req, res) {
        if (req.session.views) {
            req.session.views++;
        }
        else {
            req.session.views = 1;
        }
        res.end('Total views for you: ' + req.session.views);
    })
    .use('/reset', function (req, res) {
        delete req.session.views;
        res.end('Cleared all your views');
    })
    .listen(3000);
```    
    
Si hubiese varios servidores atendiendo las peticiones, pero yendo contra la BBDD de mongo, todas mostrarían los mismos datos y actuarían de la misma manera.

**Gestionar MongoDB**   
Recomiendan **robomongo**. Muestra todo el contenido de la BBDD y además permite acceder a la shell de mongo.


## CAP 9: FrontEnd Basics

Vamos a ver como conectar el frontend con el backend y crear aplicaciones de una sola página con **AngularJS**. Angular + Express + Mongo. En una **SPA (Single-Page Application)** el código esencial de la aplicación es cargado en la petición inicial. Luego, en las siguientes peticiones, el JS del cliente intercepta las peticiones que se hagan y sólo manda al servidor una petición XHR (XMLHttpRequest) y, cuando recibe los datos, los combina con el HTML+CSS que ya tenía de la petición inicial. Esto se puede hacer por uno mismo pero hay frameworks que te ahorran mucho trabajo.

XHR es una clase global disponible en los navegadores para realizar peticiones HTTP desde JS. Aunque contenga la palabra XML las peticiones se pueden hacer con JSON.

Para terminar de acelerar el desarrollo y no tener que hacer todo por uno mismo también va a utilizar **Bootstrap** (de Twitter) para diseñar la aplicación. Además de estilos trae ciertas funcionalidades a través de componentes JS que funcionan con jQuery. Para ello tenemos que cargar los siguientes ficheros:

* jquery.js
* bootstrap.js
* bootstrap.css

Bootstrap tiene esqueletos realizados por otras personas que podemos coger de primeras y sobre ellos comenzar a crear nuestra aplicación.

Para poder servir una SPA lo primero que hacemos es montar un servidor Express y metemos el código de estos frameworks en la carpeta “.vendor”:

```
var express = require('express');
var app = express()
    .use(express.static(__dirname + '/public'))
    .listen(3000);
```

El código de ejemplo que podría tener nuestro index.html es:

```
<html ng-app="demo">
<head>
    <title>Sample App</title>
    <!-- Add JQuery + Bootstrap JS / CSS + AngularJS-->
    <script src="./vendor/jquery/jquery.js"></script>
    <script src="./vendor/bootstrap/js/bootstrap.js"></script>
    <link rel="stylesheet" type="text/css" href="./vendor/bootstrap/css/bootstrap.css">
    <script src="./vendor/angular/angular.js"></script>
    <script src="./vendor/angular/angular-route.js"></script>
    <!-- Our Script -->
    <script>
        var demoApp = angular.module('demo', []);
        demoApp.controller('MainController', ['$scope', function ($scope) {
            $scope.vm = {
                name: "foo",
                clearName: function () {
                    this.name = ""
                }
            };
        }]);
    </script>
</head>
<body ng-controller="MainController">
    <!-- Our HTML -->
    <label>Type your name:</label>
    <input type="text" ng-model="vm.name" />
    <button class="btn btn-danger" ng-click="vm.clearName()">Clear Name</button>
</body>
</html>
```

Las distintas partes que lo conforman son:

* **Modules:** Son los que contienen los Directives y Controllers.
* **Directives:** Segmentos del código que queremos que Angular ejecute cuando encuentra que coincide un string en el HTML. Es común llamar a los directives con el prefijo “ng-” para poder diferenciar facilmente. Puedes utilizar las que vienen por defecto o crear tus propias directives.
* **Controller y $scope:**  El alma de Angular. Se llaman así por el modelo MVC (Model-View-Controller). Es el responsable de sincronizar el Modelo y la Vista. La unión es el $scope de Angular.

	
Lo realmente potente de  Angular es que permite diseñar un frontend totalmente separado del código de servidor y una vez está listo sólo queda conectar con el servidor. El pegamento es $scope.

Con “ng-repeat” repetimos la sección de HTML indicada y la clona para cada elemento en la lista:

```
<div ng-repeat="item in vm.list track by item._id" ...
</div>
```

El item luego puede ser usado para referenciar al elemento que estamos pintando. Por ejemplo para mostrar más detalles del mismo o para introducir un botón que haga referencia a él para eliminarlo.

Con ng-model enganchamos un campo del HTML con una variable del modelo. Con ng-click hacemos que una función del modelo se ejecute cuando la directiva de clickar el botón se cumpla. Desactivamos el botón con la directiva ng-disabled si no se cumplen ciertas condiciones.

Luego, para crear una aplicación con persistencia en el servidor, lo único que hay que hacer sobre esto es crear una API en Express para que se comunique con el frontend y opere en la BBDD. Luego funcionaremos a base de Promises para recibir/enviar los datos del servidor. Lo haremos creando un servicio propio que se encargue de las llamadas a través del servicio $http integrado en Angular.

Mirar como ejemplo de construcción el [siguiente proyecto](https://github.com/angular/angular-seed) creado por el equipo de Angular.
	


## CAP 10: Simplifying callbacks

Podemos tener serios problemas entre llamadas síncronas y asíncronas en nuestro código, y más según se vaya haciendo más complejo. Con el módulo **async** podemos hacer varias llamadas que durarán lo que tengan que durar y cuando hayan finalizado todas se ejecutará lo solicitado. 

En el siguiente ejemplo lo vemos con **async.parallel**:

```
// an async function to load an item
function loadItem(id, cb) {
    setTimeout(function () {
        cb(null, { id: id });
    }, 500);
}
// when all items loaded
function itemsLoaded(err, loadedItems) {
    console.log('Do something with:', loadedItems);
}
// load in parallel
var async = require('async');
async.parallel([
    function (cb) {
        loadItem(1, cb);
    },
    function (cb) {
        loadItem(2, cb);
    }
], itemsLoaded)
```

async maneja distintos tipos de flujo, por ejemplo también tiene **async.serial**.

Con lo que hay que quedarse es que **al hacer código asíncrono se vuelve mucho más crucial controlar el flujo que con la programación síncrona**.

Una de las ventajas claras de las promesas es que queda claro cuales son las funciones de entrada y salida.

**Los básicos de then y catch:** La función **then** es el corazón de la API de **promises**. Esta toma el handler onFulfilled. La otra función importante es **catch** que toma el handler onRejected. Por tanto es conveniente rechazar una promesa con un error para que haya una traza de lo que ha pasado. Then/catch puede recordar al formato try/catch de la programación síncrona.

Puedes crear promesas ya cumplidas con when y rechazadas con reject:

```
	Q.when(null).then(…
	Q.reject(new Error('...
```	
	
Lo bueno de las promesas es que **se pueden ir encadenando acciones** a través de then, ya que el resultado de una promesa con then es el comienzo de otra y así se pueden ir continuando:

```
Q.when(null)
    .then(function () {
        return 'kung foo';
    })
    .then(function (val) {
        console.log(val); // kung foo
        return Q.when('panda');
    })
    .then(function (val) {
        console.log(val); // panda
        // Nothing returned
    })
```
    
**Continuará** mientras se siga dando el handler **onFulfilled**.

**Convertir Callbacks a Promises:** Las node callbacks se llaman nodebacks. Un nodeback es una función que toma *n* argumentos y que el último es una callback y que el callback es llamado con (error) o (null, value) o (null, value1, value2, …). Para convertir esa callback en una promesa utilizamos Q.nbind, que toma todos los argumentos excluyendo el callback e internamente se lo pasa a la función nodeback junto a una función callback interna. Con ello devuelve una Promise que es:

* Rechazada si la callback interna es llamada con un argumento (error).
* Resuelta a un valor si la callback se llamó con (null, value).
* Resuelta a un array [value1, value2, …] si fue llamada con (null, value1, value2, …).

```
function data(delay, cb) {
    setTimeout(function () {
        cb(null, 'data');
    }, delay);
}
function error(delay, cb) {
    setTimeout(function () {
        cb(new Error('error'));
    }, delay);
}
// Callback style
data(1000, function (err, data) { console.log(data); });
error(1000, function (err, data) { console.log(err.message); });
// Convert to promises
var Q = require('q');
var dataAsync = Q.nbind(data);
var errorAsync = Q.nbind(error);
// Usage
dataAsync(1000)
    .then(function (data) { console.log(data); });
errorAsync(1000)
    .then(function (data) { })
    .catch(function (err) { console.log(err.message); });
```

Podemos hacer el ejemplo anterior de lectura de JSON convirtiéndolo en una llamada asíncrona a través de promises:

```
var Q = require('q');
var fs = require('fs');
var readFileAsync = Q.nbind(fs.readFile);
function loadJSONAsync(filename) {
    return readFileAsync(filename)
                .then(function (res) {
                    return JSON.parse(res);
                });
}
// good json file
loadJSONAsync('good.json')
    .then(function (val) { console.log(val); })
    .catch(function (err) {
        console.log('good.json error', err.message); // never called
    })
// non-existent json file
    .then(function () {
        return loadJSONAsync('absent.json');
    })
    .then(function (val) { console.log(val); }) // never called
    .catch(function (err) {
        console.log('absent.json error', err.message);
    })
// invalid json file
    .then(function () {
        return loadJSONAsync('bad.json');
    })
    .then(function (val) { console.log(val); }) // never called
    .catch(function (err) {
        console.log('bad.json error', err.message);
    });
```
    
Las promesas nos evitan meternos en una pirámide de controles de errores, la llamada **pyramid of doom**.

**Convertir Callbacks que no son nodebacks:** Hay muchas funciones que no siguen la convención de tener como primer argumento al error. Para poder hacer promesas de estas funciones utilizaremos el módulo **deferred** (deferred.resolve / deferred.reject).

```
var Q = require('q');
function sleepAsync(ms) {
    var deferred = Q.defer();
    setTimeout(function () {
        deferred.resolve();
    }, ms);
    return deferred.promise;
}
console.time('sleep');
sleepAsync(1000).then(function () {
    console.timeEnd('sleep'); // around 1000ms
});
```

**Convertir una promesa en callback:** Se puede hacer que una promesa siga funcionando como un callback para que personas que no estén acostumbrados a las promesas no tengan que adaptarse. Funcionará de las dos maneras. Se hace con nodeify.

```
var Q = require('q');
var fs = require('fs');
var readFileAsync = Q.nbind(fs.readFile);
function loadJSONAsync(filename, callback) {
    return readFileAsync(filename)
                .then(JSON.parse)
                .nodeify(callback);
}
// Use as a promise
loadJSONAsync('good.json').then(function (val) {
    console.log(val);
});
// Use with a callback
loadJSONAsync('good.json', function (err, val) {
    console.log(val);
});
```

Hay varios **puntos clave de las Promesas**:

* Las promesas soportan otras promesas como valor.
* Cuando se produce una excepción viaja hasta la promesa que puede manejarlo con su handler onReject. En caso de que no haya más promesas por encima no habrá quién controle esas excepciones por lo que hay que incluir promise.done():

```
iAsync()
    .catch(function (err) {
        var foo; foo.bar; // Uncaught exception, rejects the next promise
    })
    .done(); // Since previous promise is rejected throws the rejected value as an error
```
    
* Compatibilidad con la librería Promise. Lo que realmente importa es el comportamiento de “then” y de los handler onFulfilled y onRejected. Puedes utilizar Q, Bluebird o las funciones nativas de ES6.
* Estados de las promesas. Se puede recuperar con promise.inspect y se pueden hacer condicionales con promise.isFulfilled() / .isRejected() / isPending().
* Flujo paralelo de control. Puedes lanzar varias promesas y continuar cuando todas ellas hayan finalizado. Se hace con .all.

```
var Q = require('q');
// an async function to load an item
var loadItem = Q.nbind(function (id, cb) {
    setTimeout(function () {
        cb(null, { id: id });
    }, 500);
});
Q.all([loadItem(1), loadItem(2)])
    .then(function (items) {
        console.log('Items:', items); // Items: [ { id: 1 }, { id: 2 }]
    })
    .catch(function (reason) { console.log(reason) });
```

**JavaScript Generators**   
Esta es una funcionalidad que está incorporada **desde ES6**, pero como este libro es anterior lo introduce de otra manera. Sólo recogeré los aspectos clave y ya lo aprenderé cuando estudie el estándar.
Se trata de **parar la ejecución de funciones hasta que tengamos la respuesta de una promesa**.  Por ejemplo (incorporamos function* y yield):

```
function* infiniteSequence(){
    var i = 0;
    while(true){
        yield i++;
    }
}
var iterator = infiniteSequence();
while (true){
    console.log(iterator.next()); // { value: xxxx, done: false }
}
```

As demonstrated in the example, simply calling the generator does not execute it. It just returns the iterator. The first time you call next on the iterator is when execution starts and only continues until the yield keyword is evaluated, at which point iterator.next returns the value passed to the yield keyword. Each time you call iterator.next, the function body resumes execution until we finally arrive at the end of the function body, at which point the iterator.next returns an object with done set to true. This behavior is exactly what makes generating something like an infinite Fibonacci sequence in a lazy way possible.


## CAP 11: Debugging

**Console**   
**Console.log** llama por debajo a **utils.inspect** para pintar los elementos. Podemos crear una función inspect para el objeto indicando como queremos que muestre la información.

```
var foo = {
    bar: 123,
    inspect: function () {
        return 'Bar is ' + this.bar;
    }
};
// Inspect
console.log(foo); // Logs: "Bar is 123"
```

Se puede hacer como en c y pintar con %s, %d y %j para poner strings, números y json.

Para indicar tiempos utilizamos console.time() y console.timeEnd(‘punto’). Esta segunda mide el tiempo desde que se llamó a console.timeEnd(‘punto’).

Para saber quién y como ha llamado una función podemos utilizar console.trace(‘Mensaje’). También podemos ver donde se origina un error sacando su propiedad stack:

```
function foo() {
    var stack = new Error('trace at foo').stack;
    console.log(stack);
    // Execution continues
    console.log('Stack trace printed');
}
function bar() {
    foo();
}
bar();
```

Console.log pinta al stdout del programa. En cambio si utilizamos console.error pinta igualmente pero en la salida stderr.

**Node tiene internamente un depurador**. Pero teniendo IDEs donde puedo depurar más fácilmente no me parece reseñable guardar notas. Por ejemplo se puede depurar con node-inspector.

**Se puede depurar remotamente** utilizando –debug y –debug-brk (para parar en la primera línea), lo que hace levantar un servidor al que nos conectamos para poder en remoto depurar con node-inspector.


## CAP 12: Testing

Es importante probar el software para saber que los defectos son encontrados pronto y que no rompemos nada cuando introducimos nuevo código. En node hay varias módulos que nos ayudarán a hacer pruebas.

**Assert**   
Es un **core module**. Su principal objetivo es **realizar pequeños checks lógicos que nos permiten lanzar errores ante situaciones inválidas**. Sobre este módulo se construyen otros más complejos.

Por ejemplo, si tenemos un código node que se encarga de trabajar con listas, podríamos hacer el siguiente test:

```
var assert = require('assert');
var List = require('./list');
var list = new List();
console.log('testing list.count');
assert.equal(list.count(), 0);
Se trata de realizar una acción y ver con assert que se producen los cambios que necesitamos. Si tratamos de romper algo:
console.log('testing list.add throws an error on invalid value');
assert.throws(function () {
    list.add({
        value: 'some value'
    })
},
function (err) {
    return (err instanceof Error)
    && (err.message == 'item must have id')
});
```

Prácticamente todas las funciones de assert permiten meter una tercera variable para indicar mejores mensajes assert.throws(block,[errorValidator],[message]). Tiene otras funciones útiles como strictEqual, notEqual, deepEqual, AssertionError.


**Mocha**   
Para corregir problemas que se presentan utilizando a secas Assert como que cuando se realiza una batería de pruebas, si falla una, no continúa y tenemos que volver a probar cuando corregimos el error. Mocha se trata de un **framework para pruebas que hace peticiones asíncronas**.

Instalamos globalmente el módulo para poder acceder al ejecutable con cualquier javascript:
```
	npm install -g mocha
```
	
Las funciones más importantes son describe e it. Describe sirve para encapsular un set de pruebas, normalmente agrupándose por alguna referencia funcional (por ejemplo las pruebas de una clase JS). Mocha lo que hace es ir ejecutando unitariamente cada una de las pruebas y poniendo todos los resultados juntos.

Un ejemplo muy básico, que siempre irá bien uno y el otro mal:

```
var assert = require("assert");
describe('our test suite', function () {
    it('should pass this test', function () {
        assert.equal(1, 1, '1 should be equal to 1');
    });
    it('should fail this test', function () {
        assert.equal(1, 0, '1 should be equal to 0');
    });
});
```

Resultado:

```
$ mocha basic.js
  our test suite
    √ should pass this test
    1) should fail this test
  1 passing (7ms)
  1 failing
  1) our test suite should fail this test:
     AssertionError: 1 should be equal to 0
      at Context.<anonymous> (.\mochabasic\basic.js:10:16)
	…
```
	
Otras **funciones útiles** de mocha:

* **Hooks:** Para reducir el código que se debe repetir para generar las situaciones de prueba. Para ello se puede utilizar en función de la situación before, after, beforeEach y afterEach.
* **Single test run and exclusions:** Si sólamente quieres realizar una suite de pruebas (describe) o una prueba unitaria (it) puedes forzar a mocha a través de la función .only. Sólamente ejecutará el que tenga only (y sólamente puede aparecer una vez en el código):

```
describe.only('first', function () {
    it('test 1', function () {
    });
});
describe('second', function () {
    it('test 1', function () {
    });
});
```

Si lo que quieres hacer es excluir algún test porque ese en concreto está fallando lo haremos con .skip, que sirve tanto para describe como it. Es mejor que comentar código ya que mocha nos avisará de lo que tenemos marcado con skip.

```
$ mocha skip.js
  first
    √ test 1
  second
    - test 1
  third
    √ test 1
  2 passing (7ms)
  1 pending
```
  
Para poder hacer test que tengan llamadas asíncronas incluiremos un argumento (llamado convencionalmente done) en las llamadas y mocha esperará hasta que se haya llamado a ese argumento.

```
describe('our test suite', function () {
    it('this should pass', function (done) {
        setTimeout(function () {
            done(); // same as done(null);
        }, 500);
    });
    it('this should fail', function (done) {
        setTimeout(function () {
            done(new Error('fail'));
        }, 500);
    });
});
```

**Mocha no ejecutará el siguiente test hasta que no haya terminado el que está ejecutando**. Como hay algun test asíncrono que se puede colgar, se debe **marcar un timeout** para las operaciones. Se indica con –timeout o -t seguido del tiempo en milisegundos. Se puede indicar individualmente poniendo 'this.timeout(msValue)'.

Mocha también permite el uso de promesas para esperar a que las llamadas tengan lugar.

Si no le damos a mocha ficheros que ejecutar ejecutará todo lo que haya en la carpeta ./test. Si se quiere se puede especificar el patrón de ficheros de test con mocha “./directorio_test/*.js”.

En vez de utilizar only, lo más recomendable es llamar al ejecutable con -g e indicarle las pruebas que queremos que se ejecuten al cumplir cierto patrón:
```
mocha -g "my awesome test"
```
	
Se puede utilizar el depurador con mocha al igual que hemos hecho en el tema anterior.


**Chai**   
```
npm install chai
```
Módulo que **provee funciones adicionales para realizar asserts** (framework de asserts). Están contenidas dentro de require(‘chai’).assert, teniendo por ejemplo isNull/isNotNull, isFunction/isNotFunction… Además contiene los asserts básicos del módulo core.

Sobre chai se pueden instalar plugins desarrollados por otros. Por ejemplo para manejar fechas tenemos chai-datetime.

Permite escribir **BDD (Behaviour-Driven Development)**. Se puede hacer sin el módulo pero hace más fácil su lectura:

```
var expect = require('chai').expect;
var assert = require('chai').assert;
// Some variable to assert on
var beverages = { tea: ['chai', 'matcha', 'oolong'] };
// assert
assert.lengthOf(beverages.tea, 3);
// BDD
expect(beverages).to.have.property('tea').with.length(3);
// same as
expect(beverages).property('tea').length(3);
```


## CAP 13: Deployment and Scalability

En este tema vamos a ver como levantar el servidor automáticamente si tenemos un crash o como poder hacer que se ejecute en todos los cores de la máquina.

-----

**Asegurar Uptime**   
Para **que un error no tire el servidor** debemos contar con un middleware que capture el error, lo muestre en un log y continue con la ejecución del programa.

El problema lo podemos tener si el error se da en el hilo que ejecuta el proceso JS de nuestro código node. En los sistemas tradicionales que todo se hace bajo un pool de hilos, se descarta ese hilo anómalo y se crea uno nuevo. **En node hay que tirar todo el proceso y arrancarlo de nuevo**. Para ello se utilizan aplicaciones como **forever**.

**Forever:** (npm install -g forever)   
Es una herramienta de linea de comandos que se asegura de que **un script se ejecute de manera continua**. **No está exclusivamente limitado a ejecutables JS** sino que puede hacerlo con cualquier comando.
```
	forever -c node crash.js
```

**Sólo utilizar en caso de necesidad**. Hay que recordar que hay que gestionar correctamente los errores y el utilizar forever es simplemente para tener un backup en caso de situaciones inesperadas.

---

**Nodejs clustering**   
Nodejs se ejecuta en un sólo hilo por lo que está **optimizado para ejecutarse en un único procesador**. Aún así es fácil hacer que la aplicación se ejecute en todas las CPUs disponibles utilizando la API **cluster**, que es parte de los módulos core. Este módulo crea un **proceso padre que se queda escuchando las peticiones TCP/IP y va distribuyendo el tráfico HTTP a los procesos hijo** (ya que el SO no puede saber a que proceso debe mandar cada paquete recibido en un puerto).

Para crear un hijo con el mismo script que estamos ejecutando tendremos que utilizar la función fork().

El objetivo de levantar trabajadores es que se queden respondiendo las peticiones HTTP y que sea el proceso padre el que se encargue de ir enviando las peticiones a los distintos procesos. No se comparte memoria ni estructuras de datos, son procesos independientes. Toda comunicación entre ellos se realizará via eventos.
La cuenta ideal de procesos sería tener el mismo número que CPUs tiene el servidor. Si cada hilo tirase al máximo de cada CPU eso sería lo máximo que podríamos exprimir el servidor. Desde el propio proceso padre se puede determinar cual es el número de CPUs que tenemos y levantar procesos en consecuencia. Se hace de la siguiente manera:

```
	var numCPUs = require('os').cpus().length;
```

Y para estar atentos ante el cierre de uno de los procesos hijo:

```
// Listen to any worker exiting
    cluster.on('exit', function (worker, code, signal) {
        console.log('worker ' + worker.process.pid + ' exited');
    });
```

Desde el hijo podemos mandar mensajes al padre y que este los reciba a través de eventos:

```
worker.on('message', function(msg) {
        console.log('Message received from worker:', msg);
    })
```
    
Aunque se pueda hacer esto lo típico es tener múltiples servidores con una única CPU cada uno y que haya un servidor por encima con un servidor nginx que se encargue de balancear la carga a través de un proxy inverso.