### Instalación de Livewire

```shell
composer require livewire/livewire
```

### Creación un componente de Livewire
Livewire provee un comando de [[Artisan]] para generar componentes nuevos rápidamente:

```shell
php artisan make:livewire counter
```

El comando genera dos archivos nuevos en el proyecto:
- `app/Livewire/Counter.php`
- `resources/views/livewire/counter.blade.php`
#### Los componentes de Livewire deben tener un único elemento raíz
Para que Livewire funciones, los componentes deben tener solo un único elemento como raíz. Si se detectan varios elementos raíz, se lanzará una excepción. Los comentarios HTML cuentan como elementos separados y deben colocarse dentro del elemento raíz.

### Creación de un Template Layout
Necesitamos un diseño HTML para que el componente se represente en su interior. De forma predeterminada, Livewire buscará un archivo llamado: `resources/views/components/layouts/app.blade.php`. Se puede crear este archivo en caso de no existir ejecutando el comando:

```shell
php artisan livewire:layout
```

El componente se representará en lugar de la variable `$slot` de la plantilla. Livewire 3 y versiones posteriores inyectan automáticamente cualquier recurso (JavaScript y CSS) de interfaz que necesite.

> [!warning] `/livewire/livewire.js` returning a 404 status code
> De forma predeterminada, Livewire expone una ruta en su aplicación para servir sus activos JavaScript desde: `/livewire/livewire.js`. Esto está bien para la mayoría de las aplicaciones, sin embargo, si está utilizando Nginx con una configuración personalizada, puede recibir un 404 desde este endpoint. Para solucionar este problema, puede compilar los recursos de JavaScript de Livewire o configurar Nginx para permitirlo.
### Publicando el archivo de configuración
Livewire es "*zero-config*", lo que significa que puedes usarlo siguiendo las convenciones, sin ninguna configuración adicional. Sin embargo, si es necesario, puede publicar y personalizar el archivo de configuración de Livewire ejecutando el siguiente comando [[Artisan]]:

```shell
php artisan livewire:publish --config
```

Esto creará un nuevo archivo `livewire.php` en el directorio `config` de la aplicación Laravel.
### Inclusión manual de los activos Frontend de Livewire
De forma predeterminada, Livewire inyecta los recursos JavaScript y CSS que necesita en cada página que incluye un componente Livewire. Si desea tener más control sobre este comportamiento, puede incluir manualmente los activos en una página utilizando las siguientes directivas de Blade.

```html
<html>
	<head>
		...
		@livewireStyles
	</head>
	<body>
		...
		@livewireScripts
	</body>
</html>
```

Al incluir estos activos manualmente en una página, Livewire sabe que no debe inyectarlos automáticamente.

> [!info] AlpineJS is bundle with Livewire
> Dado que Alpine esþa incluido con los recursos JavaScript de Livewire, debes incluir `@livewireScritps` en cada página en la que desees utilizar Alpine.

Aunque rara vez es necesario, puede desactivar el comportamiento de inyección automática de activos de Livewire actualizando la opción de configuración `inject_assets` en el archivo `config/livewire.php` de la aplicación.

```php
'inject_assets' => false,
```

Si prefiere forzar a Livewire a inyectar sus activos en una sola página o en varias páginas, puede llamar al siguiente método global desde la ruta actual o desde un [[Service Provider]].

```php
\Livewire\Livewire::forceAssetInjection();
```
### Configurando el endpoint de actualización de Livewire
Cada actualización en un componente Livewire envía una solicitud de red al servidor en el siguiente punto final: `https://example.com/livewire/update`.
Esto puede ser un problema para algunas aplicaciones que utilizan localización o multi-tenencia.
En esos casos, puedes registrar tu propio punto final, siempre que lo hagas dentro de `Livewire::setUpdateRoute()`, Livewire sabrá utilizar este punto final para todas las actualizaciones de componentes:

```php
Livewire::setUpdateRoute(function ($handle) {
	return Route::post('/custom/livewire/update', $handle);
});
```

Ahora, en lugar de utilizar `livewire/update`, Livewire enviará actualizaciones de componentes a `/custom/livewire/update`.
Dado que Livewire le permite registrar su propia ruta de actualización, puede declarar cualquier middleware adicional que desee que Livewire use directamente dentro de `setUpdateRoute`:

```php
Livewire::setUpdateRoute(function ($handle) {
	return Route::post('/custom/livewire/update', $handle)
	->middleware([...]);
});
```

### Personalizar la URL de los activos (assets)
De forma predeterminada, Livewire servirá sus recursos de JavaScript desde la siguiente URL: `https://example.com/livewire/livewire.js`. Además, Livewire hará referencia a este activo desde una etiqueta `script` de la siguiente manera:

```html
<script src="/livewire/livewire.js" ...
```

Si su aplicación tiene prefijos de rutas globales debido a la localización o multi-tenencia, puede registrar su propio endpoint que Livewire debe usar internamente al obtener su JavaScript. Para utilizar un endpoint de assets JavaScript personalizado, puede registrar su propia ruta dentro de `Livewire::setScriptRoute()`:

```php
Livewire::setScriptRoute(function ($handle) {
	return Route::get('/custom/livewire/livewire.js', $handle);
});
```

Ahora Livewire cargará JavaScript de la siguiente manera:

```html
<script src="/custom/livewire/livewire.js" ...
```
### Agrupación manual de Livewire y Alpine
De forma predeterminada, Alpine y Livewire se cargan utilizando la etiqueta `<script src="livewire.js">`, lo que significa que no tienes control sobre el orden en que se cargan estas bibliotecas. En consecuencia, la importación y el registro de complementos de Alpine, como se muestra a continuación, ya no funcionarán:

```javascript
import Alpine from 'alpinejs'
import Clipboard from '@ryangjchandler/alpine-clipboard'

Alpine.plugin(Clipboard)
Alpine.start()
```

Para solucionar este problema, debemos informar a Livewire que queremos utilizar la versión ESM (módulo de ECMAScript) y evitar la inyección de la etiqueta de script `livewire.js`.

```html
<html>
	<head>
		<!-- ... -->
		@livewireStyles
		@vite(['resourses/js/app.js'])
	</head>
	<body>
		{{ $slot }}
		@livewireScriptConfig
	</body>
</html>
```

Cuando Livewire detecta la directiva `@livewireScriptConfig`, se abstendrá de inyectar los scripts de Livewire y Alpine. Si está utilizando la directiva `@livewireScripts` para cargar Livewire manualmente, asegúrese de eliminarla. Asegúrese de agregar la directiva `@livewireStyles` si aún no está presente.
El paso final es importar Alpine y Livewire en nuestro archivo `app.js`, lo que nos permite registrar cualquier recursos personalizado y, en última instancia, iniciar Livewire y Alpine.

```javascript
import { Livewire, Alpine } from '../../vendor/livewire/livewire/dist/livewire.esm';
import Clipboard from '@ryangjchandler/alpine-clipboard';

Alpine.plugin(Clipboard)
Livewire.start()
```

> [!info] Rebuild your assets after composer update
> Asegúrese de que, si está agrupando manualmente Livewire y Alpine, reconstruya sus activos cada vez que ejecute `composer update`

> [!danger] Not compatible with Laravel Mix
> Laravel Mix no funcionará si integras manualmente Livewire y AlpineJS. En su lugar, se recomienda cambiar a [[Vite]].
### Publicando los activos Frontend de Livewire

> [!warning] Publishing assets isn't necessary
> Publicar los recursos de Livewire no es necesario para su funcionamiento. Solo hazlo si lo necesitas específicamente.

Si prefiere que los recursos de JavaScript sean servidos por su servidor web y no a través de Laravel, utilice el comando `livewire:publish`:

```shell
php artisan livewire:publish --assets
```

Para mantener los activos actualizados y evitar problemas en futuras actualizaciones, se recomienda que agregue el siguiente comando a su archivo `composer.json`:

```json
{
	"scripts": {
		"post-update-cmd": [
			// Other scripts
			"@php artisan vendor:publish --tag=livewire:assets --ansi --force"
		]
	}	
}
```
