# Contracts

- [Introdução](#introduction)
- [Why Contracts?](#why-contracts)
- [Contract Reference](#contract-reference)
- [How To Use Contracts](#how-to-use-contracts)

<a name="introduction"></a>
## Introdução

Os contratos Laravel são um conjunto de interfaces que definem o núcleo de serviços que são fornecidos pelo framework. Por exemplo, o contrato `Queue` defini o os métodos necessários para queueing jobs(enfileirar trabalhos), enquanto o contrato `Mailer` defini os métodos necessários para o envio e-mails.

Cada contratos tem uma implementação correspondente fornecidade pelo o frameork. Por exemplo, Laravel fornece a implementação de `Queue` com a variedade de drivers, e a implementação de `Mailer` é alimentada pelo [SwiftMailer](http://swiftmailer.org/).

Todo os contratos de Laravel vivem nos [seus proprios repositórios GitHub](https://github.com/illuminate/contracts). Isto fornece um rápido ponto de referência para todos os contratos disponíveis, bem como um pacote único, desacoplado que pode ser utilizado por outros desenvolvedores de pacotes.  

<a name="why-contracts"></a>
## Porque Contratos ?

Você pode ter várias questões sobre contratos. Porque usar interfaces como um todo? Não usar interfaces é mais complicado?

Vamos destrinchar as razões para usar inteface para os seguintes tópicos: baixo acoplamento e simplicidade.

### Baixo Acoplamento

Primeiro, vamos revisar algum código que estreitamente ligado a implementação de cache. Considere o seguinte:

	<?php namespace App\Orders;

	class Repository {

		/**
		 * The cache.
		 */
		protected $cache;

		/**
		 * Create a new repository instance.
		 *
		 * @param  \SomePackage\Cache\Memcached  $cache
		 * @return void
		 */
		public function __construct(\SomePackage\Cache\Memcached $cache)
		{
			$this->cache = $cache;
		}

		/**
		 * Retrieve an Order by ID.
		 *
		 * @param  int  $id
		 * @return Order
		 */
		public function find($id)
		{
			if ($this->cache->has($id))
			{
				//
			}
		}

	}

Nesta classe, o código é estreitamente acoplado para uma dada implementação de cache. Isto é estreitamente acoplado porque nos dependemos de uma classe Cache concreta, de um pacote vendor. Se a API do pacote muda nosso código também deve mudar. 


Da mesma forma, se nos substituirmos nossa tecnologia cache subjacente (Memcached) com outra tecnologia (Redis), que novamente iremos ter que modificar nosso repositório. Nosso repositório deve não ter muito conhecimento a respeito a quem está fornecendo-lhe dados ou como eles estão fornecedo-os. 

** Aos invés desta abordagem, nos podemos melhorar nosso código pela dependência de uma simples interface agnóstica do vendor:**

	<?php namespace App\Orders;

	use Illuminate\Contracts\Cache\Repository as Cache;

	class Repository {

		/**
		 * Create a new repository instance.
		 *
		 * @param  Cache  $cache
		 * @return void
		 */
		public function __construct(Cache $cache)
		{
			$this->cache = $cache;
		}

	}

Agora o código é não acoplado para qualquer vendor específico, ou até mesmo Laravel. Uma vez que os pacotes de contratos contenham nenhuma implementação e dependência, você pode facilmente escrever uma implementação alternativa de qualquer contrato dado, permitindo você possa substituir sua implementação cache sem modificar qualquer dos seus códigos consumidores de cache. 

### Simplicidade

Quando todos os serviços do Laravel são bem definidos dentro interfaces simples, isto é muito fácil de determinar a funcionalidade oferecida por um determinado serviço. ** Os contratos servem como uma documentação sucinta às funcionalidades do framework Laravel.**

Além disso, quando você depende de interfaces simples, seu código é mais fácil de entender e manter. Ao invés de inspecionar dentro dos métodos que estão disponívels para você dentro de grande e complicada classe, você pode se referir a uma simples e limpa interface. 

<a name="contract-reference"></a>
## Referência do Contrato

Isto é a referencia para a maioria dos contratos do Laravel, bem como as "fachadas" Laravel contrapartidas:

Contract  |  Laravel 4.x Facade
------------- | -------------
[Illuminate\Contracts\Auth\Guard](https://github.com/illuminate/contracts/blob/master/Auth/Guard.php)  |  Auth
[Illuminate\Contracts\Auth\PasswordBroker](https://github.com/illuminate/contracts/blob/master/Auth/PasswordBroker.php)  |  Password
[Illuminate\Contracts\Bus\Dispatcher](https://github.com/illuminate/contracts/blob/master/Bus/Dispatcher.php)  |  Bus
[Illuminate\Contracts\Cache\Repository](https://github.com/illuminate/contracts/blob/master/Cache/Repository.php) | Cache
[Illuminate\Contracts\Cache\Factory](https://github.com/illuminate/contracts/blob/master/Cache/Factory.php) | Cache::driver()
[Illuminate\Contracts\Config\Repository](https://github.com/illuminate/contracts/blob/master/Config/Repository.php) | Config
[Illuminate\Contracts\Container\Container](https://github.com/illuminate/contracts/blob/master/Container/Container.php) | App
[Illuminate\Contracts\Cookie\Factory](https://github.com/illuminate/contracts/blob/master/Cookie/Factory.php) | Cookie
[Illuminate\Contracts\Cookie\QueueingFactory](https://github.com/illuminate/contracts/blob/master/Cookie/QueueingFactory.php) | Cookie::queue()
[Illuminate\Contracts\Encryption\Encrypter](https://github.com/illuminate/contracts/blob/master/Encryption/Encrypter.php) | Crypt
[Illuminate\Contracts\Events\Dispatcher](https://github.com/illuminate/contracts/blob/master/Events/Dispatcher.php) | Event
[Illuminate\Contracts\Filesystem\Cloud](https://github.com/illuminate/contracts/blob/master/Filesystem/Cloud.php) | &nbsp;
[Illuminate\Contracts\Filesystem\Factory](https://github.com/illuminate/contracts/blob/master/Filesystem/Factory.php) | File
[Illuminate\Contracts\Filesystem\Filesystem](https://github.com/illuminate/contracts/blob/master/Filesystem/Filesystem.php) | File
[Illuminate\Contracts\Foundation\Application](https://github.com/illuminate/contracts/blob/master/Foundation/Application.php) | App
[Illuminate\Contracts\Hashing\Hasher](https://github.com/illuminate/contracts/blob/master/Hashing/Hasher.php) | Hash
[Illuminate\Contracts\Logging\Log](https://github.com/illuminate/contracts/blob/master/Logging/Log.php) | Log
[Illuminate\Contracts\Mail\MailQueue](https://github.com/illuminate/contracts/blob/master/Mail/MailQueue.php) | Mail::queue()
[Illuminate\Contracts\Mail\Mailer](https://github.com/illuminate/contracts/blob/master/Mail/Mailer.php) | Mail
[Illuminate\Contracts\Queue\Factory](https://github.com/illuminate/contracts/blob/master/Queue/Factory.php) | Queue::driver()
[Illuminate\Contracts\Queue\Queue](https://github.com/illuminate/contracts/blob/master/Queue/Queue.php) | Queue
[Illuminate\Contracts\Redis\Database](https://github.com/illuminate/contracts/blob/master/Redis/Database.php) | Redis
[Illuminate\Contracts\Routing\Registrar](https://github.com/illuminate/contracts/blob/master/Routing/Registrar.php) | Route
[Illuminate\Contracts\Routing\ResponseFactory](https://github.com/illuminate/contracts/blob/master/Routing/ResponseFactory.php) | Response
[Illuminate\Contracts\Routing\UrlGenerator](https://github.com/illuminate/contracts/blob/master/Routing/UrlGenerator.php) | URL
[Illuminate\Contracts\Support\Arrayable](https://github.com/illuminate/contracts/blob/master/Support/Arrayable.php) | &nbsp;
[Illuminate\Contracts\Support\Jsonable](https://github.com/illuminate/contracts/blob/master/Support/Jsonable.php) | &nbsp;
[Illuminate\Contracts\Support\Renderable](https://github.com/illuminate/contracts/blob/master/Support/Renderable.php) | &nbsp;
[Illuminate\Contracts\Validation\Factory](https://github.com/illuminate/contracts/blob/master/Validation/Factory.php) | Validator::make()
[Illuminate\Contracts\Validation\Validator](https://github.com/illuminate/contracts/blob/master/Validation/Validator.php) | &nbsp;
[Illuminate\Contracts\View\Factory](https://github.com/illuminate/contracts/blob/master/View/Factory.php) | View::make()
[Illuminate\Contracts\View\View](https://github.com/illuminate/contracts/blob/master/View/View.php) | &nbsp;

<a name="how-to-use-contracts"></a>
## Como Usar Contratos 

Então, como você pode começar a implementação de contrato? Na verdade é muito simples. Muitos tipos de classes em Laravel são resolvidas pro meio do [container de serviços](/docs/{{version}}/container), incluíndo controladores, listeners(ouvidores) de eventos, queue jobs(trabalhos em fila), e até mesmo Closures de rota. Então, para começar a implementação de um contrato, você pode apenas "tipar" a interface no contrutor da classe que está sendo resolvida. Por exemplo, de uma olhada neste manipulador de evento.  

	<?php namespace App\Handlers\Events;

	use App\User;
	use App\Events\NewUserRegistered;
	use Illuminate\Contracts\Redis\Database;

	class CacheUserInformation {

		/**
		 * The Redis database implementation.
		 */
		protected $redis;

		/**
		 * Create a new event handler instance.
		 *
		 * @param  Database  $redis
		 * @return void
		 */
		public function __construct(Database $redis)
		{
			$this->redis = $redis;
		}

		/**
		 * Handle the event.
		 *
		 * @param  NewUserRegistered  $event
		 * @return void
		 */
		public function handle(NewUserRegistered $event)
		{
			//
		}

	}

Quando os listener de evento é resolvido, o container de serviço irá ler a tipagem no construtor da classse, e irá injetar o valor apropriado. Para aprender mais sobre como registrar as coisas no container de serviços, dê uma olhada na [documentação](/docs/{{version}}/container).

