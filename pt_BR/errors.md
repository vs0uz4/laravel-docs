# Errors & Logging

- [Configuração](#configuration)
- [Manipulando Erros](#handling-errors)
- [Exceções HTTP](#http-exceptions)
- [Logging](#logging)

<a name="configuration"></a>
## Configuração

As instalações para sua aplicação são configuradas na classe de inicialização  `Illuminate\Foundation\Bootstrap\ConfigureLogging`. Esta classe utiliza as opções de configuração de `log` do seu arquivo de configuração `config/app.php`.

Por padrão, o log é configurado para usar arquivos de log diários; porém, você pode customizar este comportamento como necessário. Desde que o Laravel usa a popular biblioteca de log [Monolog](https://github.com/Seldaek/monolog), você pode tirar vantagem da variedade de manipuladores que o Monolog Oferece.

Por exemplo, se você quiser usar um arquivo de log único ao invés de arquivos diários, você pode fazer a seguinte mudança no seu arquivo de configuração `config/app.php`:

	'log' => 'single'

De fora da caixa, Laravel suporta os modes de logging `single`, `daily`, `syslog` e `errorlog`. Porém, você é livre para customizar o loggin da sua aplicação como você desejar, apenas sobrescrevendo a classe de inicialização `ConfigureLogging`.  

### Detalhe de Erro

A quantidade de detalhes de erros que sua aplicação exibe através do browser é controlada pelas opções de configurações do seu arquivo de configuração `config/app.php`. Por padrão, estas opções de configuração são definidas para respeitar a variável de ambiente `APP_DEBUG`, que é armazenada em seu arquivo `.env`. 

Para o desenvolvimento local, você pode definir a variável de ambiente `APP_DEBUG` para `true`. ** No seu ambiente de produção, este valor deve ser sempre `false`.** 

<a name="handling-errors"></a>
## Manipulando Erros 

Todas exceções são manipuladas pela class `App\Exceptions\Handler`. Esta classe contém dois métodos: `report` e `render`.

O método `report` é usado para logar exceções ou manda-las para um serviço externo como o [BugSnag](https://bugsnag.com). Por padrão, o método `report` simplesmente passa a exceção para a implementação base na classe pai onde a exceção é logada. Porém, você é livre para logar exceções como quiser. Se você precisar relatar tipos diferentes de exceções de modos diferente, você pode usar o operador de compação do PHP, `instanceof`.

	/**
	 * Report or log an exception.
	 *
	 * This is a great spot to send exceptions to Sentry, Bugsnag, etc.
	 *
	 * @param  \Exception  $e
	 * @return void
	 */
	public function report(Exception $e)
	{
		if ($e instanceof CustomException)
		{
			//
		}

		return parent::report($e);
	}

O método `render` é responsável por converter a exceção em uma reposta HTTP que deve ser enviada de volta ao browser. Por padão, a execeção é passada para a classe básica que gera uma resposta para você. Porém, você é livre para checar tipos de exceção ou retornar a sua própria resposta customizada.

A propriedade `dontReport` do manipulador de exceções contém um array de tipos exceção que não serão logados. Por padrão, exceções resultantes de erros 404 não são escritas nos seus arquivos de log. Você pode adiciona outros tipos de exceções para este array, conforme necessário.

<a name="http-exceptions"></a>
## Exceções HTTP

Algumas exceções descrevem os código de erros HTTP do servidor. Por exemplo, isto pode ser um erro 404 "página não encontrada", um "erro não autorizado" (401) ou até mesmo um erro 500 de desenvolvimento. Afim de retornar tal resposta, use os seguintes:

	abort(404);

Opcionalmente, você pode fornecer a resposta:

	abort(403, 'Unauthorized action.');

Este método pode ser usado a qualquer hora durante o siclo de vida da requisição.

### Página de erro 404 Customizada

Para retornar uma view customizada para todos os erros 404, crie um arquivo `resources/views/errors/404.blade.php`. Esta view irá ser servidar em todos os erros 404 gerados pela sua aplicação.

<a name="logging"></a>
## Logging

As instalações do loggin do laravel fornecem uma camada simples no top da poderosa biblioteca [Monolog](http://github.com/seldaek/monolog). Por padrão, Laravel é configurado para criar arquivos de log diariamente para sua aplicação que é armazenado no diretório `storage/logs`. Você pode escrever informações no log assim:

	Log::info('This is some useful information.');

	Log::warning('Something could be going wrong.');

	Log::error('Something is really going wrong.');


O logger fornece os setes levels de loggin definidos em [RFC 5424](http://tools.ietf.org/html/rfc5424): **debug**, **info**, **notice**, **warning**, **error**, **critical**, e **alert**.

Um array de dados contextuais pode também ser passado para os métodos de log:

	Log::info('Log message', ['context' => 'Other helpful information']);

Monolog tem uma variedade adicional de manipuladores que você pode usar para logging. Se você precisar, você pode acessar a instância subjacente do Monolog que está sendo usada pelo Laravel:

	$monolog = Log::getMonolog();

Você pode também registrar um evento para pegar todas as mensagens passada para o Log:

#### Registrando um Listener(Escutador) de Evento de Log

	Log::listen(function($level, $message, $context)
	{
		//
	});
