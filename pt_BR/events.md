# Events

- [Uso Básico](#basic-usage)
- [Manipuladores de Eventos Em Fila](#queued-event-handlers)
- [Assinantes de Eventos](#event-subscribers)

<a name="basic-usage"></a>
## Uso Básico

As instalações de eventos do Laravel fornecem uma simples implementação de observer, o que lhe permite inscrever e escutar eventos em sua aplicação. Classes evento são tipicamente armazenada no diretório  `app/Events`, enquanto seus manipuladores são armazenados em `app/Handlers/Events`.

Você pode gerar uma nova classe de evento usando a ferramenta CLI Artisan:

	php artisan make:event PodcastWasPurchased

#### Subescrevendo Para um Evento

O `EventServiceProvider` incluído com a sua aplicação Laravel fornece um lugar conveniente para registrar todos os manipuladores de evento. A propriedade `listen` contém um array de todos os eventos (chaves) e seus manipuladores (valores). É claro que, você pode adicionar quantos eventos forem necessários para sua aplicação. Por exemplo, vamos adicionar o nosso `PodcastWasPurchased`:

	/**
	 * The event handler mappings for the application.
	 *
	 * @var array
	 */
	protected $listen = [
		'App\Events\PodcastWasPurchased' => [
			'App\Handlers\Events\EmailPurchaseConfirmation',
		],
	];

Para gerar um manipulador para o evento, use o comando CLI Artisan `handler:event`:

	php artisan handler:event EmailPurchaseConfirmation --event=PodcastWasPurchased

É claro que, executando manualmente os comandos `make:event` e `handler:event` cada vez que você precisar de um manipulado ou evento é complicado. Em vez disso, simplesmente adicione manipuladores e eventos para o seu  `EventServiceProvider` e use o comando `event:generate`. Este comando irá gerar quaisquer eventos ou manipuladores que estão listados no seu `EventServiceProvider`:

	php artisan event:generate

#### Disparando um Evento

Agora nos estamos prontos para disparar nossos eventos usando a fachada `Event`:

	$response = Event::fire(new PodcastWasPurchased($podcast));

O método `fire` retorna um array de respostas que você pode usar para controlar o que acontece depois na sua aplicação. 

Você pode também usar o `event` "helper" para disparar um evento:

	event(new PodcastWasPurchased($podcast));

#### Closure Listeners (Escutadores Closure)

Você pode até ouvir eventos sem criar classes de manipuladores separadas a final de conta. Por exemplo, no método `boot` do seu EventServiceProvider`, você pode fazer o seguinte:

	Event::listen('App\Events\PodcastWasPurchased', function($event)
	{
		// Handle the event...
	});

#### Parando a Propagação De um Evento

Algumas vezes, você pode desejar parar a propagação de um eventos para outrs escutadores. Você pode fazer isto retornando `false` do seu manipulador:

	Event::listen('App\Events\PodcastWasPurchased', function($event)
	{
		// Handle the event...

		return false;
	});

<a name="queued-event-handlers"></a>
## Manipuladores de Eventos em Fila.

Precisando [enfileirar](/docs/{{version}}/queues) um maipulador de eventos ? Isto não pode ser mais fácil. Quando estiver gerando um manipulador, simplesmente use a flag  `--queued`:

	php artisan handler:event SendPurchaseConfirmation --event=PodcastWasPurchased --queued

Isto irá gerar a classe manipuladora que implementa a interface `Illuminate\Contracts\Queue\ShouldBeQueued`. É isto! Agora quando este manipulador é chamado para um evento, isto irá ser enfileirado automaticamente pelo dispachador de eventos.

Se nenhuma exceção é lançada quando o manipulador é executado pela fila, o trabalho enfileirador será exluído automaticamente depois que isto tiver sido processado. Se você precisar acessar manualmente os métodos `delete` e `release` do trabalho enfileirado, você pode fazê-lo. A trait `Illuminate\Queue\InteractsWithQueue`, que é incluído por padrão nos manipuladores em fila, dá a você acesso para estes métdodos:

	public function handle(PodcastWasPurchased $event)
	{
		if (true)
		{
			$this->release(30);
		}
	}

Se você tem um manipulador existente que você gostaria de converter para um manipulador em fila, simplesmente adicione a interface `ShouldBeQueued` para a classe manualmente. 

<a name="event-subscribers"></a>
## Assinantes de eventos

#### Definindo um Assinante de evento 

Assinantes de eventos são classes que podem subescrever para múltiplos eventos de dentro da própria classe. Assinantes devem definir o método `subscribe`, que irá ser passado para uma instância de despachador de eventos:

	class UserEventHandler {

		/**
		 * Handle user login events.
		 */
		public function onUserLogin($event)
		{
			//
		}

		/**
		 * Handle user logout events.
		 */
		public function onUserLogout($event)
		{
			//
		}

		/**
		 * Register the listeners for the subscriber.
		 *
		 * @param  Illuminate\Events\Dispatcher  $events
		 * @return array
		 */
		public function subscribe($events)
		{
			$events->listen('App\Events\UserLoggedIn', 'UserEventHandler@onUserLogin');

			$events->listen('App\Events\UserLoggedOut', 'UserEventHandler@onUserLogout');
		}

	}

#### Registrando um Assinante de Evento 

Uma vez que o assinante tenha sido definido, isto pode ser registrado com a classe `Event`.

	$subscriber = new UserEventHandler;

	Event::subscribe($subscriber);

Você pode também usar o [container de serviço](/docs/{{version}}/container) para resolver seu assinante. Para fazê-lo, simplesmente passe o nome do seu assinante ppara o método `subscribe`:

	Event::subscribe('UserEventHandler');

