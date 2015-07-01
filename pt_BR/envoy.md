# Envoy Task Runner

- [Introdução](#introduction)
- [Instalação](#envoy-installation)
- [Executando Tarefas](#envoy-running-tasks)
- [Múltiplos Servidores](#envoy-multiple-servers)
- [Parallel Execution](#envoy-parallel-execution)
- [Macros de Tarefas](#envoy-task-macros)
- [Notificações](#envoy-notifications)
- [Atualizando o Envoy](#envoy-updating-envoy)

<a name="introduction"></a>
## Introdução

[Laravel Envoy](https://github.com/laravel/envoy) fornece uma limpa, sintax minimalista por definição tarefas comuns que são executadas nos seus servidores remotos. Usando o estilo de sintax do Blade, você pode facilmente configurar tarefas para deploy, comandos Artisan, e mais.

> **Nota:** Envoy requere que a versão do PHP seja 5.4 ou superior, e apenas funciona nos sistemas operacionais Mac/Linux.

<a name="envoy-installation"></a>
## Instalação

Primeiro, instale Envoy usando o comando `global` do Composer:

	composer global require "laravel/envoy=~1.0"

Assegure-se de alocar o diretório `~/.composer/vendor/bin` na sua variável de ambiente PATH para que então o comando `envoy` seja encontrado quando você executar o comando no terminal.

Em seguida, crie um arquivo blade `Envoy.blade.php` no diretório raíz do seu projeto. Aqui vai um exemplo de como vocẽ pode começar:

	@servers(['web' => '192.168.1.1'])

	@task('foo', ['on' => 'web'])
		ls -la
	@endtask

Como você pode ver, o array de `@servers` é definido no topo do arquivo. Você pode referenciar estes servidores na opção `on` das suas declarações de tarefas. Dentro das suas declarações `@task` você deve alocar o código Bash que irá executar no seu servidor quando a tarefa é executada.

O comando `init` pode ser usado para criar facilmente um arquivo Stub do Envoy:

	envoy init user@192.168.1.1

<a name="envoy-running-tasks"></a>
## Executando Tarefas

Para executar tarefas, user o comando `run` da sua instalação Envoy:

	envoy run foo

Se necessário, você pode passar variáveis no arquivo Envoy usando a linha de comando:

	envoy run deploy --branch=master

Você pode usar as opções via a sintax do Blade que você está acostumado:

	@servers(['web' => '192.168.1.1'])

	@task('deploy', ['on' => 'web'])
		cd site
		git pull origin {{ $branch }}
		php artisan migrate
	@endtask

#### Bootstrapping

Você pode usar a diretiva ```@setup``` para declarar variáveis e fazer o trabalho geral do php dentro do arquivo Envoy:

	@setup
		$now = new DateTime();

		$environment = isset($env) ? $env : "testing";
	@endsetup

Você pode também usar ```@include``` para incluir qualquer arquivo PHP:

	@include('vendor/autoload.php');

#### Confirmando Tarefas Antes de Executar

Se você desejar a confirmação seja exibida na tela uma confirmação, antes que uma tarefa seja executada nos seus servidores, você pode usar a diretiva `confirm`:

	@task('deploy', ['on' => 'web', 'confirm' => true])
		cd site
		git pull origin {{ $branch }}
		php artisan migrate
	@endtask

<a name="envoy-multiple-servers"></a>
## Múltiplos Servidores

Você pode facilmente executar tarefas em múltiplos servidores. Simplesmente liste os servidores na declaração da tarefa:

	@servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

	@task('deploy', ['on' => ['web-1', 'web-2']])
		cd site
		git pull origin {{ $branch }}
		php artisan migrate
	@endtask

Por padrão, a tarefa será executada em cada servidor em série. Significando, que a tarefa finalizará a execução no primeiro servidor prosseguindo para executar no próximo.

<a name="envoy-parallel-execution"></a>
## Execução Paralela

Se você quiser executar a tarefa em múltiplos servidores em paralelo, simplesmente adicione a opção `parallel` na sua declaração de tarefa:

	@servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

	@task('deploy', ['on' => ['web-1', 'web-2'], 'parallel' => true])
		cd site
		git pull origin {{ $branch }}
		php artisan migrate
	@endtask

<a name="envoy-task-macros"></a>
## Macros de Tarefas

Macros permitem que você defina um conjunto de tarefas para serem executadas em sequênia usando um único comando. Por instância:

	@servers(['web' => '192.168.1.1'])

	@macro('deploy')
		foo
		bar
	@endmacro

	@task('foo')
		echo "HELLO"
	@endtask

	@task('bar')
		echo "WORLD"
	@endtask

O macro `deploy` agora pode ser executado por meio um único, e simples comando:

	envoy run deploy

<a name="envoy-notifications"></a>
<a name="envoy-hipchat-notifications"></a>
## Notificações

#### HipChat

Depois de executar a tarefa, você pode enviar a notificação para o seu time na sala do HipChat usando uma simples diretiva `@hipchat`:

	@servers(['web' => '192.168.1.1'])

	@task('foo', ['on' => 'web'])
		ls -la
	@endtask

	@after
		@hipchat('token', 'room', 'Envoy')
	@endafter

Você pode também especificar uma mensagem customizada para sua sala no hipchat. Qualquer variável declarada em ```@setup``` ou incluída com ```@include``` será disponibilizada para o uso na mensagem.

	@after
		@hipchat('token', 'room', 'Envoy', "$task ran on [$environment]")
	@endafter

Isto é um modo incrivelmente simples de manter seu time notificado sobre as tarefas que estão sendo executadas no servidor.

#### Slack

A sintax a seguir pode ser usada para enviar a notificação para o [Slack](https://slack.com):

	@after
		@slack('hook', 'channel', 'message')
	@endafter

Você pode recuperar sua URL webhook criando um "WebHooks Entrada" (webhook é um modo que aplicação tem de fornecer a outras aplicações informações real-time) na integração no site do Slack. Por exemplo:

	https://hooks.slack.com/services/ZZZZZZZZZ/YYYYYYYYY/XXXXXXXXXXXXXXX

Você pode fornecer um dos seguintes para parâmetro do canal:
  
- Para enviar a notificação apra o canal: `#channel`
- Para enviar a notificação apra o usuário: `@user`

Se nenhum argumento do `channel` for fornecido, o canal usará o padrão. 

> Nota: Notificação Slack irão apenas ser enviada se todas as tarefas forem completadas com sucesso.

<a name="envoy-updating-envoy"></a>
## Atualizando o Envoy

Para atualizar o Envoy, simplesmente use o Composer:

	composer global update

