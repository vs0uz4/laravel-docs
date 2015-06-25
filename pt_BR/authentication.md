# Authentication

- [Introdução](#introduction)
- [Autenticação de usuários](#authenticating-users)
- [Recuperando o Usuário Atenticado](#retrieving-the-authenticated-user)
- [Protecting Routes](#protecting-routes)
- [Autenticação Básica HTTP](#http-basic-authentication)
- [Lembretes de Senhas & Reset](#password-reminders-and-reset)
- [Autenticação Social](#social-authentication)

<a name="introduction"></a>
## Introdução

Laravel faz a implementação de autenticação de maneira muito simples. Na verdade, quase tudo está previamente configurado para você. O arquivo de configuração da autenticação está localizado no diretório `config/auth.php`, o qual contém muitas opções bem documentadas para adequar o comportamento dos serviços de autenticação.

Por Padrão, Laravel adiciona um modelo `App\User` em seu diretório `app`. Este modelo poderá ser usado com o Driver de Autentição do Eloquent. 

Lembre-se: quando estiver criando o schema do banco de dados para este modelo, faça com que a coluna password tenha no mínimo 60 caracteres. Também, antes de começar, garanta que a sua tabela `users` (ou equivalente) contenha uma coluna padrão Null `remember_token` com no mínimo 100 caracteres. Esta coluna será usada para armazenar um token de sessões "Lembre-me"  que são mantidas pela sua aplicação. Isto pode ser feito usando o método `$table->rememberToken();` na migração. Por padrão Laravel 5 realiza as migrações para estas colunas. 

Se sua aplicação não estiver utilizando o Eloquent, você pode usar o driver de autenticação `database` que usa o construtor de querys do Laravel. 

<a name="authenticating-users"></a>
## Autenticação de usuários

Laravel vem pré-configurado com duas autenticações relacionadas aos controladores (controllers). O `AuthController`gerencia o registro de novos usuários e a autenticação ("logging in"), enquanto o `PasswordController`contem a lógica para ajudar usuários existentes na recuperação de suas senhas esquecidas.

Cada um destes controladores (controllers) usa um trait para incluir os métodos necessários. Para muitas aplicações você não precisará modificá-los. As visões (views) que estes controladores (controllers) renderizam estão localizadas no diretório `resources/views/auth`. Você pode personaliza-las da maneira que melhor atender à sua aplicação.

### O Registro do usuário

Para modificar os campos do formulário que são necessários quando um novo usuário se registra em sua aplicação, você pode alterar a classe `App\Services\Registrar`. Essta classe é responsável pela validação e criação de novos usuários em sua aplicação.

The `validator` method of the `Registrar` contains the validation rules for new users of the application, while the `create` method of the `Registrar` is responsible for creating new `User` records in your database. You are free to modify each of these methods as you wish. The `Registrar` is called by the `AuthController` via the methods contained in the `AuthenticatesAndRegistersUsers` trait.

O método `validator` da classe `Registrar` contem as regras de validação para novos usuários na aplicação, enquanto o método `create` da classe `Registrar` é responsável pela criação do novo registro de `User` (usuário) no banco de dados. Você pode modificar cada um destes métodos para atender às necessidades de sua aplicação. `Registrar` é chamado por `AuthController` por meio dos métodos existentes no trait `AuthenticatesAndRegistersUsers`.

#### Autenticação manual

Se você optar por não utilizar a implementação do provedor `AuthController`, será necessário gerenciar a autenticação de seus usuários usando as classes de autenticação do Laravel diretamente. Não se preocupe, isto é simples. Primeiro vamos verificar um método  `attempt` (tentativa):

	<?php namespace App\Http\Controllers;

	use Auth;
	use Illuminate\Routing\Controller;

	class AuthController extends Controller {

		/**
		 * Handle an authentication attempt.
		 *
		 * @return Response
		 */
		public function authenticate()
		{
			if (Auth::attempt(['email' => $email, 'password' => $password]))
			{
				return redirect()->intended('dashboard');
			}
		}

	}

O método `attempt` aceita um array de pares  chave / valor como seu primeiro argumento. O valor de `password` (senha) será [criptografado](/docs/{{version}}/hashing). Os demais valores no array serão utilizados para encontrar o usuário no banco de dados. Assim, no exemplo acima, o usuário será recuperado pelo valor da coluna `email`. Se o usuário for encontrado, a senha criptografada armazanada no banco de dados, será comparada com o valor criptografado da senha passado pelo método na array. Se duas senhas criptografadas combinarem com a pesquisada, uma nova seção de usuário autenticado será iniciada.

O método `attempt` irá retornar `true` se a autenticação for realizada com sucesso. Caso contrário irá retornar `false`.

> **Nota:** neste exemplo, `email` não é uma opção obrigatória, ele é usado apenas como um exemplo. Você deverá utilizar qualquer nome de coluna correspondente a um "username" (nome de usuário) em seu banco de dados.

A função de redirecionamento `intended` redirecionará o usuário para a URL que ele estava tentando acessar antes de ser captuado pelo filtro de autenticação. Uma URI de retaguarda deve ser fornecida para o método caso o destino previamente tentado não estiver disponível.

#### Autenticando um usuário com condições

Você também pode adicionar condições extrar para a query de autenticação:

	if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1]))
	{
		// The user is active, not suspended, and exists.
	}

#### Determinando se um usuário está autenticado

Para determinar se um usuário já está autenticado (logged) em sua aplicação, pode-se utilizar o método `check`:

	if (Auth::check())
	{
		// The user is logged in...
	}

#### Autenticando o usuário e os "Relembrando" 

Se você quiser fornecer a funcionalidade de "remenber me"(lembre-se de mim). na sua aplicação, você pode passar um valor booleano como o segundo argumento para o método `attempt`, que manterá o usuário autenticado indefinidamente, ou até que ele deslogue manualmente da aplicação. É claro que, sua tabela de `users` deve incluir uma coluna string nomeada `remember_token`, que irá ser usada para armazenar a token da funcionalidade de "remember me" (lembre-se de mim).

	if (Auth::attempt(['email' => $email, 'password' => $password], $remember))
	{
		// The user is being remembered...
	}

Se você está "remembering" (lembrando) usuário, você pode usar o método `viaRemember` (Lembre-se de via) para determinar se o usuário foi autenticado usando o cookie "remember me":

	if (Auth::viaRemember())
	{
		//
	}

#### Autenticando usuários pelo ID
Para logar o usuário na aplicação pelo seu ID, use o método `loginUsingId`:

	Auth::loginUsingId(1);

#### Validando as credenciais de usuário sem Login 

O método `validate` permite que você valide as credenciais do usuário sem que você tenha que realmente loga-los na aplicação:

	if (Auth::validate($credentials))
	{
		//
	}

#### Logando o usuário em uma única requisição

Você pode também usar o método `once` para logar o usuário na alicação por apenas uma requisição. Nem sessões ou cookies serão utilizados:

	if (Auth::once($credentials))
	{
		//
	}

#### Logando manualmente em um usuário

Se você precisa logar uma instância de usuário existente na sua aplicação, você pode chamar o método `login` com a instância do usuário:

	Auth::login($user);
	
Isto é equivalente a um usuário fazer o login por meio de credenciais usando o método `attempt`.

#### Deslogando um usuário da sua aplicação

	Auth::logout();

É claro que, se você estiver usando os controladores de autenticação built-in(padrões do Laravel), o método do controlador que desconecta os usuários da aplicação é provido de fora da caixa. 

#### Eventos de Autenticação


Quando o método `attempt` é chamado, o [event](/docs/{{version}}/events)  `auth.attempt` será executado. Se a tentativa de autenticação for bem-sucedida e o usuário estiver logado na aplicação, o evento `auth.login` também será executado. 

<a name="retrieving-the-authenticated-user"></a>
## Recuperando um usuário Autenticado

Uma vez que o usuário é atenticado, existem várias maneiras de se obter a instância do usuário.

Primeiro, você pode acessar o usuário a patir da fachada `Auth`:


	<?php namespace App\Http\Controllers;

	use Illuminate\Routing\Controller;

	class ProfileController extends Controller {

		/**
		 * Update the user's profile.
		 *
		 * @return Response
		 */
		public function updateProfile()
		{
			if (Auth::user())
			{
				// Auth::user() returns an instance of the authenticated user...
			}
		}

	}

Segundo, você pode acessar o usuário autenticado via a instância da `Illuminate\Http\Request`:

	<?php namespace App\Http\Controllers;

	use Illuminate\Http\Request;
	use Illuminate\Routing\Controller;

	class ProfileController extends Controller {

		/**
		 * Update the user's profile.
		 *
		 * @return Response
		 */
		public function updateProfile(Request $request)
		{
			if ($request->user())
			{
				// $request->user() returns an instance of the authenticated user...
			}
		}

	}

Em terceiro lugar, você pode tipar o contrato `Illuminate\Contracts\Auth\Authenticatable`. Esta tipagem pode ser adicionada ao método contrutor do controlador, métodos do controlador, ou qualquer outro construtor de uma classe resolvida pelo [service container](/docs/{{version}}/container):

	<?php namespace App\Http\Controllers;

	use Illuminate\Routing\Controller;
	use Illuminate\Contracts\Auth\Authenticatable;

	class ProfileController extends Controller {

		/**
		 * Update the user's profile.
		 *
		 * @return Response
		 */
		public function updateProfile(Authenticatable $user)
		{
			// $user is an instance of the authenticated user...
		}

	}

<a name="protecting-routes"></a>
## Protegendo Rotas

[Rotas middleware](/docs/{{version}}/middleware) podem ser usadas para permitir que apenas usuários atenticados possam acessar a rota fornecida. Laravel provem por padrão o middleware `auth`, e isto é definido em `app\Http\Middleware\Authenticate.php`. Tudo que você precisa fazer é anexar isto as definições das rotas.

	// With A Route Closure...

	Route::get('profile', ['middleware' => 'auth', function()
	{
		// Only authenticated users may enter...
	}]);

	// With A Controller...

	Route::get('profile', ['middleware' => 'auth', 'uses' => 'ProfileController@show']);

<a name="http-basic-authentication"></a>
## Autenticação HTTP Básica

A atenticação HTTP básica fornece uma forma rápida para atenticação de usuário da sua aplicação sem de uma configuração dedicada a página de login. Para iniciar, anexe o middleware `auth.basic` a sua rota: 

#### Protegendo uma Rota com HTTP Básico

	Route::get('profile', ['middleware' => 'auth.basic', function()
	{
		// Only authenticated users may enter...
	}]);

Por padrão, o midleware `basic` irá usar a coluna `email` no registro do usuário como o "username".

#### Configurando Um filtro HTTP Básico Stateless


Você pode também usar a autenticação HTTP básica sem definir um cookie identificar do usuário na sessão, que é particularmente útil para a autenticação de API. Para fazer isso, [defina um middleware](/docs/{{version}}/middleware) que chama o método `onceBasic`:

	public function handle($request, Closure $next)
	{
		return Auth::onceBasic() ?: $next($request);
	}

Se você estiver usando PHP FastCGI, a autenticação básica HTTP pode não funcionar corretamente fora da caixa, As linhas a seguir devem ser adicionadas ao seu arquivo `.htaccess`:

	RewriteCond %{HTTP:Authorization} ^(.+)$
	RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

<a name="password-reminders-and-reset"></a>
## Resets e Lembretes de Senha

### Modelo & Tabela

A maioria das aplicações web fornecem um meio para que os usuários resetem as senhas esquecidas. Ao inves de forçar você a ré-implementar isto em cada aplicação, Laravel fornece um método conveniente para mandar lembretes das senhas e realizar resets.

Para comerçar, verifique o que o seu modelo `User` implementa o contrato `Illuminate\Contracts\Auth\CanResetPassword`. É claro que, o
modelo `User` incluído com o framework já implementa esta interface, e usa a trait `Illuminate\Auth\Passwords\CanResetPassword` para incluir os métodos necessários para implementação da interface. 

#### Gerando a migração da tabela de lembretes

Em seguida, uma tabela tem que ser criada para armazenar os tokens de redefinição das senhas. A migração para esta tabela é incluida com o Laravel fora da caixa, e fica localizada no diretório `database/migrations`. Então tudo que vocêr precisar fazer é migrar: 

	php artisan migrate

### Controlador Do Password Reminder *(Lembrete de Senha)

Laravel also includes an `Auth\PasswordController` that contains the logic necessary to reset user passwords. We've even provided views to get you started! The views are located in the `resources/views/auth` directory. You are free to modify these views as you wish to suit your own application's design.

Seu usuário irá receber um e-mail com um link que aponta para o método `getReset` do controlador `PasswordController`. Este método irá processar o form de redefinição de senha e permitir que o usuário a redefina. Depois que a senha é redefinida, o usuário irá automaticamente ser logado na aplicação e redirecionado para a rota `/home`. Você pode customizar a localização do redirecionamento da pós-redefinição definindo a propriedade `redirectTo` no `PasswordController`:

	protected $redirectTo = '/dashboard';

> **Note:** Por padrão, tokens de redifinição de senhas expiram depois de uma hora. Você pode mudar isto na opção `reminder.expire` que fica localizado no seu arquivo `config/auth.php`.

<a name="social-authentication"></a>
## Autenticação Social

Além da atutenticação típica, baseada em um formulário de autenticação, Laravel também fornece uma simples e conveniente modo de autenticar com provedores OAth usando using [Laravel Socialite](https://github.com/laravel/socialite).** Atualmente Socialite dá suporte a autenticação com o Facebook, Twitter, Google, Github e Bitbucket.**

Para começar com Socialite, inclua o pacote em seu arquivo `composer.json`:

	"laravel/socialite": "~2.0"


Em seguida, registre o `Laravel\Socialite\SocialiteServiceProvider` em seu arquivo de configuração `config/app.php`. Você também pode registrar a [fachada](/docs/{{version}}/facades):

	'Socialize' => 'Laravel\Socialite\Facades\Socialite',

Você precisará adicionar as credenciais para os serviços OAuth que sua aplicação utiliza. Essas credenciais devem estar localizada em seu arquivo de configuração `config/services.php`, e devem usar as chaves `facebook`, `twitter`, `google`, or `github`, dependendo dos provedores que sua aplicação requer, Por Exemplo. 

	'github' => [
		'client_id' => 'your-github-app-id',
		'client_secret' => 'your-github-app-secret',
		'redirect' => 'http://your-callback-url',
	],

Em seguida, você está pronto para autenticar usuários! Você precisar de duas rotas: uma  para redirecionar o usuário para o provedor OAuth, e outro para receber o callback do provedor após a autenticação. Aqui vai um exemplo usando a fachada `Socialize`:

	public function redirectToProvider()
	{
		return Socialize::with('github')->redirect();
	}

	public function handleProviderCallback()
	{
		$user = Socialize::with('github')->user();

		// $user->token;
	}
O método `redirect` fica responsável por enviar ao usuário ao provedor OAuth, enquando o método `user` lê a requisição que chega, e recupera as informações dos usuário a partir do provedor. Antes de redirecionar o usuário, você pode também definir "scopes" da requisição:

	return Socialize::with('github')->scopes(['scope1', 'scope2'])->redirect();

Uma vez que você tenha a instância do usuário, você pode pegar um pouco mais de detalhes sobre o mesmo:

#### Recuperando Detalhes dos Usuários

	$user = Socialize::with('github')->user();

	// OAuth Two Providers
	$token = $user->token;

	// OAuth One Providers
	$token = $user->token;
	$tokenSecret = $user->tokenSecret;

	// All Providers
	$user->getId();
	$user->getNickname();
	$user->getName();
	$user->getEmail();
	$user->getAvatar();
