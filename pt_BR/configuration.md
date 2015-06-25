# Configuration

- [Introdução](#introduction)
- [Depois da Instalação](#after-installation)
- [Acessando Valores de Configuração](#accessing-configuration-values)
- [Configuração de Ambiente](#environment-configuration)
- [Configuração de Caching](#configuration-caching)
- [Modo de Manutenção](#maintenance-mode)
- [URLs Amigáveis](#pretty-urls)

<a name="introduction"></a>
## Introdução

Todos os arquivos de configuração para o framework Laravel são armazenados no diretório `config`. Cada opção é documentada, então sinta-se livre para dar uma olhada nos arquivos
e se famialirizar com as opções disponíves pra você.

<a name="after-installation"></a>
## Depois Da Instalação

### Nomeando sua aplicação

Depois de instalar Laravel, você pode querer nomear a sua aplicação. Por padrão, o diretório `app` tem o namespace `App`, e é carregado automaticamente pelo Composer usando a [PSR-4 autoloading standard](http://www.php-fig.org/psr/psr-4/). Contudo, você pode mudar o namespace para o mesmo da sua aplicação, que você pode facilmente fazer por meio do comando Artisan `app:name`.

Por exemplo, se sua aplicação é nomeada "Horsefly", você pode executar o seguinte comando a partir do diretório root da sua instalação:

	php artisan app:name Horsefly

Renomear sua aplicação é totalmente opcional, e você é livre para manter o namespace `App` se você desejar.

### Outras Configurações

Laravel precisa de poucas configurações de fora da caixa. Você é livre para começar a desenvolver! No entanto, você pode desejar revisar o arquivo `config/app.php` e sua documentação. Nela, contém várias opções como as de `timezone` (fuso-horário) e `locale` (localização) que você pode desejar mudar de acordo com a sua localização.


Uma vez que o Laravel é instalado, você deve tambeḿ [configure your local environment](/docs/{{version}}/configuration#environment-configuration).

> **Nota:** Você nunca deve ter a configuração `app.debug` definida como "true" no ambiente de produção da sua aplicação.

<a name="permissions"></a>
### Permissões

Laravel pode requerer um conjunto de permissões para ser configurado: pastas dentro do diretório `storage` requerem permissões de acesso a escrita no servidor. 

<a name="accessing-configuration-values"></a>
## Acessando Valores de Configuração

Você pode facilmente acessar os valores de configurações usando a fachada `Config`:

	$value = Config::get('app.timezone');

	Config::set('app.timezone', 'America/Chicago');

Você pode também usar o função helper `config`:

	$value = config('app.timezone');

<a name="environment-configuration"></a>
## Configuração de Ambiente

Na maioria das vezes, é util ter diferentes valores de configurações baseados nos ambientes em que a aplicação está sendo executada. Por examplo, você pode desejar usar drivers de cache diferentes localmente do que você usa no servidor de produção. Isto é fácil, usando o ambiente baseado na configuração.  

Para isso ser fácil, Laravel utiliza a biblioteca PHP feita por Vance Lucar [DotEnv](https://github.com/vlucas/phpdotenv). Em uma instalação pura de laravel, o diretório rais da sua aplicação irá conter o arquivo `.env.example`. Se você instalar o Laravel via Composer, este arquvivo irá automaticamente ser renomeado por `.env`. Caso contrário, você deve renomear o arquvio manualmente. 

Todas as variáveis listadas neste arquvio serão carregadas na superglobal PHP `$_ENV` quando sua aplicação receber a requisição. Você pode usar o helper `env` para recuperar os valores das variáveis. Na verdade, se você revisar os arquivos de configuração do Laravel, você irá notar que várias das opções já usam este Helper!

Sinta-se à vontade para modificar as suas variáveis de ambiente como for necessário para o seu servidor local, como também para seu ambiente de produção. No entanto, seu arquivo `.env` não deve ser commitado para sistema de controle de versionamento, desde que cada desenvolvedor / servidor que está usando sua aplicação possa requerer diferentes configurações de ambiente. 

Se você estiver desenvolvendo com um time, você pode querer continuar incluindo o arquivo `.env.example` na sua aplicação. Ao colocar valores place-holder no exemplo de arquivo de configuração, outros desenvolvedor no seu time podem claramente ver que o as variáveis de ambeinte são necessárias para a execução da sua aplicação. 

#### Acessando Ambiente Atual da Aplicação 

Você pode acessar o ambiente atual da sua aplicação por meio do método `environment` na instância de `Application`:

	$environment = $app->environment();

Você pode também passar argumento para o método `environment` para checar se o ambiente tem o mesmo valor do argumento passado:

	if ($app->environment('local'))
	{
		// The environment is local
	}

	if ($app->environment('local', 'staging'))
	{
		// The environment is either local OR staging...
	}

Para obeter a instância da aplicação, resolva o contrato `Illuminate\Contracts\Foundation\Application` via o [container de serviços](/docs/{{version}}/container). É claro que, se você você estiver dentro de um [provedor de serviço](/docs/{{version}}/providers), a instância da aplicação é disponibilizada via a instância da variável `$this->app`.

A instância da aplicação pode também ser acessada via o helper `app` ou pela fachada `App`:

	$environment = app()->environment();

	$environment = App::environment();

<a name="configuration-caching"></a>
## Configuração Caching

Para da a sua aplicação um empurrão na velocidade, você pode armazenar em cache todos os seus arquivos de configuração em um único arquivo usando o comando Artisan `config:cache`. Iso irá combinar todas as opções do configuração da sua aplicação em um único arquivo que pode ser carregado rapidamente pelo framework.

Você deve executar normalmente o comando `config:cache` como parte da sua rotina de deploy.

<a name="maintenance-mode"></a>
## Modo de Manutenção

Quando sua aplicação estier em modo de manutenção, uma view customizada será exibida para todas as requisições a aplicação. Isto faz com que seja fácil de "desativar" a sua aplicação enquanto ela é atualizada ou quando você efetuando uma manutenção. A checage se a aplicação está no modo de manutenção é incluída por padrão na pilha do middleware da sua aplicação. Se a aplicação estiver em modo de manutenção, uma `HttpException` (exceção HTTP) será lançada ao usuário com o código de status 503.

Para ativar o modo de manutenção, simplesmente execute o comando Artisan `down`:

	php artisan down

Para disativar o modo de manutenção, use o comando `up`:

	php artisan up

### Template de reposta Do Modo de Manutenção

A template padrão para reposta do modo de manutenção, é localizado no diretório `resources/views/errors/503.blade.php`.

### Modo de manutenção e Queues 

Enquanto sua aplicação estiver no modo de manutenção, nenhuma [queued jobs](/docs/{{version}}/queues) serão tratada. As queues continuarão sendo tratadas normalmente uma vez que a sua aplicação sair do modo de manutenção.

<a name="pretty-urls"></a>
## URLs Amigáveis

### Apache

O framework envia com o arquivo `public/.htaccess` o que é usando para permitir URLs sem `index.php`. Se você usa Apache como webserver da sua aplicação Laravel, certifique-se de ativar o módulo `mod_rewrite`.

Se o arquivo `.htaccess` que vem com a aplicação Laravel não function com a sua instalação do Apache, teste com esta:

	Options +FollowSymLinks
	RewriteEngine On

	RewriteCond %{REQUEST_FILENAME} !-d
	RewriteCond %{REQUEST_FILENAME} !-f
	RewriteRule ^ index.php [L]

Se o seu web host não permitir a opção `FollowSymlinks`, teste substituí-lo com `Options +SymLinksIfOwnerMatch`.

### Nginx

No Nginx, a seguinte diretiva na sua configuração de site permitira URLs "amigáveis":

	location / {
		try_files $uri $uri/ /index.php?$query_string;
	}
É claro que, quando estiver usando a [Homestead](/docs/{{version}}/homestead), URLs amigáveis serão configuradas automaticamente. 
