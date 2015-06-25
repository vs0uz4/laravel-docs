# Service Container

- [Introdução](#introduction)
- [Uso Básico](#basic-usage)
- [Ligação de Interfaces para Implementações](#binding-interfaces-to-implementations)
- [Ligações Contextuais](#contextual-binding)
- [Tagging](#tagging)
- [Aplicações Práticas](#practical-applications)
- [Container de Eventos](#container-events)

<a name="introduction"></a>
## Introdução

O serviço Laravel container é uma ferramenta poderosa para gerenciar dependencia de classes. Injeção de dependeência é uma palavra bonita que essencialmente que dizer: Dependencia de classes são injetadas na classe via método contrutor ou, em alguns casos, via métodos "setter".

Vamos da uma olhada neste exemplo:

	<?php namespace App\Handlers\Commands;

	use App\User;
	use App\Commands\PurchasePodcastCommand;
	use Illuminate\Contracts\Mail\Mailer;

	class PurchasePodcastHandler {

		/**
		 * The mailer implementation.
		 */
		protected $mailer;

		/**
		 * Create a new instance.
		 *
		 * @param  Mailer  $mailer
		 * @return void
		 */
		public function __construct(Mailer $mailer)
		{
			$this->mailer = $mailer;
		}

		/**
		 * Purchase a podcast.
		 *
		 * @param  PurchasePodcastCommand  $command
		 * @return void
		 */
		public function handle(PurchasePodcastCommand $command)
		{
			//
		}

	}

Neste exemplo, o manipulador de comando `PurchasePodcast` precisar enviar e-mail quando um podcast for comprado. Então, nos iremos **injetar** o serviço que é hábil a enviar e-mails. Desde que o serviço é injetado, nos podemos facilmente trocar isto por uma outra implementação. Nos também podemos facilmente "mockar", ou criar uma implementação dummy do enviador de e-mail quando estivermos testando a nossa aplicação.

Um profundo entendimento do container de serviços do Laravel é essencial para criar uma poderosa e robusta aplicação, bem como para contribuir para o próprio núcleo Laravel. 

<a name="basic-usage"></a>
## Uso Básico

### Bindings

Quase todas as bindings(ligações) do seu container de serviço serão registratos nos [provedores de serviço](/docs/{{version}}/providers), então todos estes exemplos demostrarão o uso do container neste contexto. No entando, se você precisar de uma instância do container em um outro lugar da sua aplicação, como uma factory, você pode tipar o contrato `Illuminate\Contracts\Container\Container`  e a instância do container irá ser injetada para você. Alternativamente, você pode usar a fachada `App` para acessar o container. 

#### Registrando Um Resolvedor Básico 

Dentro de um provedor de serviço, você sempre tem acesso ao container via a variável de instância `$this->app`. 

Existem várias maneiras de o container de serviços possa registrar dependências, incluindo Closure callbacks e binding(ligações) de interfaces para implementações. Primeiro, nos vamos explorar Closure callbacks. Um resolver Closure é registrado em um container com a chave (normalmente o nome da classe) e a Closure que retorna algum valor:

	$this->app->bind('FooBar', function($app)
	{
		return new FooBar($app['SomethingElse']);
	});

#### Registrando um Singleton

Algumas vezes, você pode desejar bing(ligar) algumas coisa ao container que deve apenas ser resolvido uma vez, e a mesma instância pode ser retornada em chamadas subsequentes para o container. 

	$this->app->singleton('FooBar', function($app)
	{
		return new FooBar($app['SomethingElse']);
	});

#### Binding(Ligar) Uma Instância Existente ao Container. 

Você pode também bind um objeto intância de objeto existente no container usando o método `instance`. A instância dada sempre será retornada em chamadas subsequentes para o container:

	$fooBar = new FooBar(new SomethingElse);

	$this->app->instance('FooBar', $fooBar);

### Resolvendo

Há vários modos de se resolver alguma coisa fora do container. Primeiro, você pode usar o método `make`:

	$fooBar = $this->app->make('FooBar');

Segundo, você pode usar "array access"(acesso array) no container, desde que isto implemente `ArrayAccess` interface PHP's:

	$fooBar = $this->app['FooBar'];

Por fim, mas o mais importante, você pode simplesmente "type-hint"(tipar) a dependência no méotodo contrutor da classe que é resolvido pelo container, incluindo controladores, listeners de eventos, queue jobs, filtros, e mais. O container irá automaticamente injetar as dependências. 

	<?php namespace App\Http\Controllers;

	use Illuminate\Routing\Controller;
	use App\Users\Repository as UserRepository;

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

		/**
		 * Show the user with the given ID.
		 *
		 * @param  int  $id
		 * @return Response
		 */
		public function show($id)
		{
			//
		}

	}

<a name="binding-interfaces-to-implementations"></a>
## Binding Interfaces Para Implementações 

### Injetando Dependências Concretas

Uma funcioanalidade muito poderosa do container de serviços é a abilidade de bind(ligar) uma interface a uma implementação dada. Por exemplo, talvez nossa aplicação se integra com o webservice [Pusher](https://pusher.com) para mandar e receber eventos em tempo-real. Se nos estivermos usando o PHP SDK do Pusher's, nos podemos injetar uma instância do cliente Pusher a uma classe. 

	<?php namespace App\Handlers\Commands;

	use App\Commands\CreateOrder;
	use Pusher\Client as PusherClient;

	class CreateOrderHandler {

		/**
		 * The Pusher SDK client instance.
		 */
		protected $pusher;

		/**
		 * Create a new order handler instance.
		 *
		 * @param  PusherClient  $pusher
		 * @return void
		 */
		public function __construct(PusherClient $pusher)
		{
			$this->pusher = $pusher;
		}

		/**
		 * Execute the given command.
		 *
		 * @param  CreateOrder  $command
		 * @return void
		 */
		public function execute(CreateOrder $command)
		{
			//
		}

	}

Neste exemplo, é bom que estajamos injetando dependência de classes, no entanto, nos firmemente acoplanos ao SDK do Pusher. Se o método do pusher SDK mudarem ou nos decidimos trocar enteiramente para um novo serviço de evento, precisararemos mudar o código `CreateOrderHandler`.

### Programe Para Uma Interface 


Afim de "isolar" o método `CreateOrderHandler` contra alterações para execuções de eventos, nos poderíamos definir uma interface de `EventPusher` e uma implementação de `PusherEventPusher`:

	<?php namespace App\Contracts;

	interface EventPusher {

		/**
		 * Push a new event to all clients.
		 *
		 * @param  string  $event
		 * @param  array  $data
		 * @return void
		 */
		public function push($event, array $data);

	}

Uma vez que temos codados nossa implementação de interface `PusherEventPusher`, nos podemos registrar isto com o container de serviços como a seguir

	$this->app->bind('App\Contracts\EventPusher', 'App\Services\PusherEventPusher');

Isto informa ao container, que se deve injetar o `PusherEventPusher` quando a classe precisa da implementação do `EventPusher`. Agora nos podemos tipar a inteface `EventPusher` no nosso método construtor:

		/**
		 * Create a new order handler instance.
		 *
		 * @param  EventPusher  $pusher
		 * @return void
		 */
		public function __construct(EventPusher $pusher)
		{
			$this->pusher = $pusher;
		}

<a name="contextual-binding"></a>
## Binding Contextual

Algumas vezes você pode ter duas classes que utilizam a mesma interface, mas você deseja injetar diferente implementações em cada classe. Por exemplo, quando nosso sistema recebe um novo Pedido, nos podemos querer enviar um evento via [PubNub](http://www.pubnub.com/) ao invés do Pusher. Laravel fornece uma simples e fluente para definir este comportamento. 

	$this->app->when('App\Handlers\Commands\CreateOrderHandler')
	          ->needs('App\Contracts\EventPusher')
	          ->give('App\Services\PubNubEventPusher');

<a name="tagging"></a>
## Tagging

Ocasionalmente, você pode precisar resolver tudo de uma certa "categoria" de binding(ligação). Por exemplo, talvez você esteja construindo um relatório agregador que receba um array de várias implementações de intefaces de 'Reports'(Relatórios) diferentes. Depois de registrar as implementações de `Reports`(Relatórios), você pode atribuir-lhe uma tag usando o método `tag`:

	$this->app->bind('SpeedReport', function()
	{
		//
	});

	$this->app->bind('MemoryReport', function()
	{
		//
	});

	$this->app->tag(['SpeedReport', 'MemoryReport'], 'reports');

Uma vez que os serviços estiverem atribuídos a tags, você pode facilmente pode resolve-los todos por meios do método `tagged`:

	$this->app->bind('ReportAggregator', function($app)
	{
		return new ReportAggregator($app->tagged('reports'));
	});

<a name="practical-applications"></a>
## Aplicações Práticas Practical Applications

Laravel fornece várias oportunidade para usar o container de serviços para aumentar a flexibilidade e testabilidade da sua aplicação. 
Um exemplo primário é quando se esta resolvendo controladores. Todos os controladores são resolvidos por meio do container de serviços, o que significa que você pode tipar dependências em um método construtor de um controlador, e elas irão automaticamente ser injetadas. 

	<?php namespace App\Http\Controllers;

	use Illuminate\Routing\Controller;
	use App\Repositories\OrderRepository;

	class OrdersController extends Controller {

		/**
		 * The order repository instance.
		 */
		protected $orders;

		/**
		 * Create a controller instance.
		 *
		 * @param  OrderRepository  $orders
		 * @return void
		 */
		public function __construct(OrderRepository $orders)
		{
			$this->orders = $orders;
		}

		/**
		 * Show all of the orders.
		 *
		 * @return Response
		 */
		public function index()
		{
			$orders = $this->orders->all();

			return view('orders', ['orders' => $orders]);
		}

	}

Neste exemplo, a classe `OrderRepository` irá automáticamente ser injetada no controlador. Isto significa que o "mock" `OrderRepository` pode ser ligado dentro do container quando estivermos usando [testes unitários](/docs/{{version}}/testing), permitindo que o desgaste ao interagir com a camada de banco de dados seja menor. 


#### Outros Exemplos do Uso do container

É claro que, como foi citado acima, controladores não são as unicas classes que Laravel resolve via container de serviços. Você pode também tipar dependências de rotas Closures, filtros, queue jobs(tarefas em fila), listeners(ouvintes) de evento, e muito mais. Para mais exemplos de usar o container de serviços nestes contextos, por favor consultar a documentação do mesmo. 

<a name="container-events"></a>
## Container de Eventos 

#### Registrando Um Resolvedor Listener(escutador)

O container dispara um evento a cada vez que resolve um objeto. Você pode escutar este evento usando o método `resolving`: 


	$this->app->resolving(function($object, $app)
	{
		// Called when container resolves object of any type...
	});

	$this->app->resolving(function(FooBar $fooBar, $app)
	{
		// Called when container resolves objects of type "FooBar"...
	});

O objeto que está sendo resolvido será passado para o callback. 