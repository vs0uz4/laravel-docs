# HTTP Controllers

- [Introdução](#introduction)
- [Básico de Controladores](#basic-controllers)
- [Controladores Middleware](#controller-middleware)
- [Controladores Ímplicitos](#implicit-controllers)
- [RESTful Resource Controllers](#restful-resource-controllers)
- [Injeção de Dependências & Controllers](#dependency-injection-and-controllers)
- [Caching de Rotas](#route-caching)

<a name="introduction"></a>
## Introdução

Ao invés de definir todos a lógica das suas requisições em um único arquivo `routes.php`, você pode desejar organizar este comportamente usando classes Controllers ou Controlador. Controladores podem agrupar solicitações HTPP relacionadas a manipulação lógica de uma classe. Controladores são normalmente localizados no diretório `app/Http/Controllers`.

<a name="basic-controllers"></a>
## Básico de Controladores

Aqui vai um exemplo de uma classe básica de controlador:

	<?php namespace App\Http\Controllers;

	use App\Http\Controllers\Controller;

	class UserController extends Controller {

		/**
		 * Show the profile for the given user.
		 *
		 * @param  int  $id
		 * @return Response
		 */
		public function showProfile($id)
		{
			return view('user.profile', ['user' => User::findOrFail($id)]);
		}

	}

Nós pode encaminhar para a ação do controlador assim:

	Route::get('user/{id}', 'UserController@showProfile');

> **Nota:** Todos os controladores devem herdar da classe controladora base. 

#### Controladores & Namespaces

Isto é muito importante para notar que nos não precisavamos especificar o namespace do controlador por completo, apenas uma parte do nome da classe, apenas uma parte do nome da classe que vem depois `App\Http\Controllers` do namespace "root"(raíz). Por padrão, o `RouteServiceProvider` irá carregar o arquivo `routes.php` dentro do grupo de rota contendo o controlador de namespace (root)raiz. 

Se você escolher aninhar ou organizar seus controladores usando namespaces do PHP no diretório `App\Http\Controllers`, simplemene use o nome específico da classe relativa ao namespace raíz `App\Http\Controllers`. Então, se namespace completo do seu controlador forApp\Http\Controllers\Photos\AdminController`, você deve registrar a rota como:

	Route::get('foo', 'Photos\AdminController@method');

#### Nomeando as Rotas de Controladores

Como rotas Closure, você pode especificar nomes nas rotas dos controladores. 

	Route::get('foo', ['uses' => 'FooController@method', 'as' => 'name']);

#### URLs Para Ações do Controller

Para gerar uma URL para uma ação do controlador, user o método helper `action`.

	$url = action('App\Http\Controllers\FooController@method');

Se você desejar gerar uma URL para uma ação do controlador enquanto estiver usando apenas uma porção do nome da classe relativa ao namespace do seu controlador, registre o namespace do controlador raíz com o gerador de URL.

	URL::setRootControllerNamespace('App\Http\Controllers');

	$url = action('FooController@method');

Você pode acessar o nome da ação do controlador que está sendo executada usando o método `currentRouteAction`:

	$action = Route::currentRouteAction();

<a name="controller-middleware"></a>
## Controller Middleware

[Middleware](/docs/{{version}}/middleware) pode ser especificado nas rotas dos controladores assim:

	Route::get('profile', [
		'middleware' => 'auth',
		'uses' => 'UserController@showProfile'
	]);

Além disso, você pode especificar middleware dentro do método construtor do seu controlador: 

	class UserController extends Controller {

		/**
		 * Instantiate a new UserController instance.
		 */
		public function __construct()
		{
			$this->middleware('auth');

			$this->middleware('log', ['only' => ['fooAction', 'barAction']]);

			$this->middleware('subscribed', ['except' => ['fooAction', 'barAction']]);
		}

	}

<a name="implicit-controllers"></a>
## Controladores Implícitos

Laravel permite que você facilmente defina uma rota única para manipular ações em um controlador. Primeiro, defina a rota usando o método `Route::controller`:

	Route::controller('users', 'UserController');

O método `controller` aceita dois argumentos. O primeiro é a URI base que o controlador manipula, enquando o segundo é o nome da classe do controlador. Em seguida, apenas adicione o método em seu controlador, prefixado com o verbo HTTP que o mesmo responde a:

	class UserController extends BaseController {

		public function getIndex()
		{
			//
		}

		public function postProfile()
		{
			//
		}

		public function anyLogin()
		{
			//
		}

	}

Os métodos `index` irão responder para a URI raíz tratada pelo controlador, que neste caso, é `users`.

Se o sua ação do controlador contém múltiplas palavras, você pode acessar a ação usando a sintax "dash" na URI. Por exemplo, a  seguinte ação do nosso controladro `UserController` deve responder a URI `users/admin-profile`: 

	public function getAdminProfile() {}

#### Assinando Nomes as Rotas

Se você quiser "nomear" alguma das rotas no seu controlador, você pode passar um terceiro argumento ao método `controller`:  

	Route::controller('users', 'UserController', [
		'anyLogin' => 'user.login',
	]);

<a name="restful-resource-controllers"></a>
## Controladores RESTful Resource 

Controladores Resource dão facicilidades para se desenvolver controladores RESTful em volta do resouce. Por exemplo, você pode criar um controlador que lida com requisições HTTP sobre "photos"(fotos) armazenadas pela sua aplicação. Usando o comando Artisan `make:controller`, nos podemos rapidamente criar tal controlador. 

	php artisan make:controller PhotoController

Em seguida, nos registramos uma rota resourceful(do tipo resource) para o controlador:

	Route::resource('photo', 'PhotoController');

Esta declaração única de rota cria múltiplas rotas para ligar com uma variedade de ações RESTful no resource de "photo". Da mesma forma, o controlador gerado já irá ter os métodos prontos para cada ação, incluindo notas informando a você as URIs e verbos que elas manipulam.

#### Ações Manipuladas Pelo Controlador Resource

Verb      | Path                  | Action       | Route Name
----------|-----------------------|--------------|---------------------
GET       | /photo                | index        | photo.index
GET       | /photo/create         | create       | photo.create
POST      | /photo                | store        | photo.store
GET       | /photo/{photo}        | show         | photo.show
GET       | /photo/{photo}/edit   | edit         | photo.edit
PUT/PATCH | /photo/{photo}        | update       | photo.update
DELETE    | /photo/{photo}        | destroy      | photo.destroy

#### Customizando Rotas Resource

Adicionalmente, você pode especificar apenas um subconjunto de ações para lidar na rota:

	Route::resource('photo', 'PhotoController',
					['only' => ['index', 'show']]);

	Route::resource('photo', 'PhotoController',
					['except' => ['create', 'store', 'update', 'destroy']]);

Por padrão, todas as ações dos controladores resource tem um nome de rota, no entanto, você pode sobreescrever estes nomes passando um array `names` com suas opções:

	Route::resource('photo', 'PhotoController',
					['names' => ['create' => 'photo.build']]);

#### Lidando Com Controladores Nested Resource

Para "nest"(aninhar) um controlador resource, use a notação de "."(ponto) na sua declaração de rota:

	Route::resource('photos.comments', 'PhotoCommentController');

Esta rota registrará o resource "nested" (aninhado) que pode ser acessado com URLs como a seguinte:  `photos/{photos}/comments/{comments}`.

	class PhotoCommentController extends Controller {

		/**
		 * Show the specified photo comment.
		 *
		 * @param  int  $photoId
		 * @param  int  $commentId
		 * @return Response
		 */
		public function show($photoId, $commentId)
		{
			//
		}

	}

#### Adicionando outras rotas para Controladores Resource

Se se tornar necessário adicionar outras rotas para um controlador resource além das rotas resorce padrão, você deve definir estas rotas antes de chamar o método `Route::resource`:

	Route::get('photos/popular', 'PhotoController@method');

	Route::resource('photos', 'PhotoController');

<a name="dependency-injection-and-controllers"></a>
## Injeção de dependências e Controladores

#### Injeção no método Construtor

O [conainer de serviços](/docs/{{version}}/container) do Laravel é usando para resolver todos os controladores Laravel. Como resultado, você está apto a tipar qualquer dependência que o seu controlador precise no método contrutor: 

	<?php namespace App\Http\Controllers;

	use Illuminate\Routing\Controller;
	use App\Repositories\UserRepository;

	class UserController extends Controller {

		/**
		 * The user repository instance.
		 */
		protected $users;

		/**
		 * Create a new controller instance.
		 *
		 * @param  UserRepository  $users
		 * @return void
		 */
		public function __construct(UserRepository $users)
		{
			$this->users = $users;
		}

	}
É claro que, você pode também tipar qualquer [Contrato Laravel](/docs/{{version}}/contracts). Se o container pode resolver isto, você pode tipa-lo.

#### Injeção de Método

Além de injeção no método construtor, você pode também tipar dependências nos métodos do seu controlador. Por exemplo, vamos tipar a instâncias de `Request` em um de nossos métodos:

	<?php namespace App\Http\Controllers;

	use Illuminate\Http\Request;
	use Illuminate\Routing\Controller;

	class UserController extends Controller {

		/**
		 * Store a new user.
		 *
		 * @param  Request  $request
		 * @return Response
		 */
		public function store(Request $request)
		{
			$name = $request->input('name');

			//
		}

	}
Se seu método do controlador também está esperando um parâmetro na rota, simplesmente liste seus argumentos da rota depois das suas outras dependências:

	<?php namespace App\Http\Controllers;

	use Illuminate\Http\Request;
	use Illuminate\Routing\Controller;

	class UserController extends Controller {

		/**
		 * Store a new user.
		 *
		 * @param  Request  $request
		 * @param  int  $id
		 * @return Response
		 */
		public function update(Request $request, $id)
		{
			//
		}

	}

> **Nota:** Injeção de métodos é totalmente compatível com > **Note:** Method injection is fully compatible with [model binding](/docs/{{version}}/routing#route-model-binding). O container é inteligete para determinar quais argumentos são modelos vínculados e quais deverão ser injetados. 

<a name="route-caching"></a>
## Caching de Rotas

Se a sua aplicação está usando exclusivamente as rotas do controlador, você pode tirar vantagem do cache de rotas do Laravel. Usar o cache de rotas irá diminuir drasticamente a quantidade de tempo que leva para registrar todas as rotas da sua aplicação. Em alguns casos, o seu registro de rota pode ser até 100 vezes mais rápido! Para gerar o cache de rotas, apenas execute o comando Artisan `route:cache`:

	php artisan route:cache

Isto é tudo que você precisa para isto! Seus arquivos de rotas cache serão agora usados ao invés do seu arquivo `app/Http/routes.php`. Lembre-se, se você adicionar qualquer rota nova você também precisará gerar uma rota cache nova. Por causa disto, você pode quere apenas executar o comando `route:cache` durante o deploy do seu projeto. (Conselho só execute o comando no processo de deploy).

Para remover os arquivos de rota cache sem gerar um novo cache, use o comando `route:clear`:

	php artisan route:clear
