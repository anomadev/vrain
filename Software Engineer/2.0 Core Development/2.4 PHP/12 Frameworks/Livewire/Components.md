Los componentes son los pilares de una aplicación Livewire. Combinan estado y comportamiento para crear elementos reutilizables de la interfaz de usuario (UI) para el frontend.
### Creando componentes
Un componente Livewire es simplemente una clase PHP que extiende de `Livewire\Component`. Puede crear un componente manualmente o usando el comando de [[Artisan]]:

```shell
php artisan make:livewire CreatePost
```

Si prefiere nombres en `kebab-cased`, también puede usar:

```shell
php artisan make:livewire create-post
```

Después de ejecutar el comando, Livewire creará dos archivos nuevos en la aplicación. El primero será la clase del componente: `app/Livewire/CreatePost.php`

```php
<?php

namespace App\Livewire;

use Livewire\Component;

class CreatePost extends Component
{
	public function render()
	{
		return view('livewire.create-post');
	}
}
```

El segundo será un componente de vista [[Blade]]: `resources/views.livewire/create-post.blade.php`

```html
<div>
	{{-- ... --}}
<div>
```

Puede utilizar la sintaxis de `namespace` o la notación de punto para crear sus componentes en subdirectorios.

```shell
php artisan make:livewire Posts\\CreatePost
php artisan make:livewire posts.create-post
```

### Componentes inline
Si el componente es bastante pequeño, es posible que desee crear un componente en línea. Los componentes en línea son componentes Livewire de un solo archivo cuya plantilla de vista está contenida directamente en el método `render()` en lugar de un archivo separado:

```php
<?php

namespace App\Livewire;

use Livewire\Component;

class CreatePost extends Component
{
	public function render()
	{
		return <<<'HTML'
		<div>
			{{-- Your blade template goes here... --}}
		</div>
		HTML;
	}
}
```

Puede crear componentes en línea agregando el flag `--inline` al comando `make:livewire`.

```shell
php artisan make:livewire CreatePost --inline
```

### Omitiendo el método render
Para reducir el código repetitivo en sus componentes, puede omitir por completo el método `render` y Livewire utilizará su propio método `render` subyacente, que devuelve una vista con el nombre convencional correspondiente a sus componente.
### Personalización de stubs de componentes
Puede personalizar los archivos (o stubs) que Livewire utiliza para generar nuevos componentes ejecutando el siguiente comando:

```shell
php artisan livewire:stubs
```

Esto creará cuatro archivos nuevos en su aplicación:
- `stubs/livewire.stub`: se utiliza para generar nuevos componentes.
- `stubs/livewire.inline.stub`: se utiliza para generar componente en línea.
- `stubs/livewire.test.stub`: se utiliza parar generar archivos de `test`.
- `stubs/livewire.view.stub`: se utiliza para generar componentes de vistas.

Aunque estos archivos se encuentren en la aplicación, aún puede usar el comando `make:livewire` [[Artisan]] y Livewire utilizará automáticamente sus archivos personalizados al generar archivos.

## Estableciendo propiedades
Los componentes Livewire tienen propiedades ([[Properties]]) que almacenan datos y son fácilmente accesibles desde la clase del componente y la vista [[Blade]]. Para añadir una propiedad a un componente Livewire, declara una propiedad pública en la clase del componente.

```php
<?php

namespace App\Livewire;

use Livewire\Component;

class CreatePost extends Component
{
	public $title = 'Post title...';

	public function render()
	{
		return view('livewire.create-post');
	}
}
```

Las propiedades del componente se muestran automáticamente en la vista [[Blade]] del componente. Puede referenciarlas mediante la sintaxis estándar de [[Blade]].

```html
<div>
	<h1>Title: "{{ $title }}"</h1>
</div>
```

### Compartiendo información adicional con la vista
Además de acceder a las propiedades desde la vista, puede pasar datos explícitamente a la vista desde el método `render()`, como lo haría normalmente desde un controlador. Esto puede ser útil cuando desea pasar datos adicionales sin almacenarlos primero como una propiedad, ya que las propiedades tienen **implicaciones específicas de rendimiento y seguridad**.
Para pasar datos a la vista en el método `render()`, puede utilizar el método `with()` en la instancia de la vista.

```php
<?php

namespace App\Livewire;

use Illuminate\Support\Facades\Auth;
use Livewire\Component;

class CreatePost extends Component
{
	public $title = 'Post title...';

	public function render()
	{
		return view('livewire.create-post')->with([
			'author' => Auth::user()->name,
		]);
	}
}
```

Ahora puede acceder a la propiedad `$author` desde el componente de vista [[Blade]]:

```php
<div>
	<h1>Title: "{{ $title }}"</h1>
	<spam>Author: {{ $author }}</spam>
</div>
```

### Agregando `wire:key` a bucles `@foreach` 
Al recorrer datos en una plantillas Livewire usando `@foreach`, debe agregar un atributo `wire:key` único al elemento raíz representado por el bucle.
Sin un atributo `wire:key` presente dentro del bucle, Livewire no podrá hacer coincidir adecuadamente los elementos antiguos con sus nuevas posiciones cuando el bucle cambie. Esto puede provocar muchos problemas difíciles de diagnosticar en la aplicación.

```php
<div>
	@foreach ($post as $post)
		<div wire:key="{{ $post->id }}">
			<!-- ... -->
		</div>
	@endforeach
</div>
```

Si está recorriendo una matriz que representa componentes Livewire, puede establecer la clave como un atributo del componente `:key` o pasar la clave como un tercer argumento al utilizar la directiva `@livewire`

```php
<div>
	@foreach ($post as $post)
		<livewire:post-item :$post :key="$post->id">
		@livewire(PostItem::class, ['post' => $post], key($post->id))
	@endforeach
</div>
```

### Vinculación de inputs a propiedades
Una de las características más poderosas es la vinculación de datos (*data binding*): la capacidad de mantener automáticamente las propiedades sincronizadas con las entradas del formulario en la página.

```php
<form>
	<label for="title">Title:</label>
	<input type="text" id="title" wire:model="title">
</form>
```

Cualquier cambio realizado en la entrada de texto se sincronizará automáticamente con la propiedad `$title` del componente Livewire.

> [!warning] Why isn't my component live updating as I type?
> Livewire solo actualiza un componente cuando se envía una "acción", como presionar un botón de envío, no cuando un usuario escribe en un campo. Esto reduce las solicitudes de red y mejora el rendimiento. Para habilitar la actualización "en vivo" a medida que un usuario escribe, puede utilizar `wire:model.live` en su lugar.

### Llamando acciones
Las acciones ([[Actions]]) son métodos dentro de su componente Livewire que manejan interacciones del usuario o realizan tareas específicas. Suelen ser útiles para responder a clics en botones o envíos de formularios en una página.

```php
<?php

namespace App\Livewire;

use Livewire\Component;
use App\Models\Post;

class CreatePost extends Component
{
	public $title;

	public function save()
	{
		$post = Post::create([
			'title' => $this->title
		]);

		return redirect()->to('/posts')
			->with('status', 'Post created!');
	}

	public function render()
	{
		return view('livewire.create-post');
	}
}
```

Llama a la acción `save` desde la vista [[Blade]] del componente agregando la directiva `wire:submit` al elemento `<form>`:

```php
<form wire:submit="save">
	<label for="title">Title:</label>
	<input type="text" id="title" wire:model="title">
	<button type="submit">Save</button>
</form>
```

Cuando se hace clic en el botón "`Save`", ejecutará el método `save()` en el componente Livewire y el componente volverá a renderizar.
### Renderizando componentes
 