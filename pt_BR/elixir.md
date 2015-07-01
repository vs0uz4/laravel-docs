# Laravel Elixir

- [Introdução](#introduction)
- [Instalação e Configuração](#installation)
- [Uso](#usage)
- [Gulp](#gulp)
- [Extenções](#extensions)

<a name="introduction"></a>
## Introdução

Laravel Elixir fornece uma limpa e fluente API para definição básica [Gulp](http://gulpjs.com) de tarefas para sua aplicação Laravel. Elixir dá suporte a vários pré-prcessadores de javascript e css comuns, e até ferramentas de teste. 

Se você já ficou confuso sobre como começar com Gulp e compilação ativa, você irá amar Laravel Elixir!


<a name="installation"></a>
## Instalação e Configuração

### Instalando o Node

Antes de engatilhar o Elixir, você tem que primeiro que o Node.js estaja instalado na sua máquina. 

    node -v

Por padrão, a Laravel Homestead inclui tudo que você precisa; contudo, se você não estiver usando Vagrant, então você pode facilmente instalar o Node visitando [sua página de download](http://nodejs.org/download/). Não se preocupe, é rápido e fácil.

### Gulp

Em seguida, você que precisar baixar o [Gulp](http://gulpjs.com) como um pacote global NPM(Node Package Manager|Gerenciador de Pacotes do Node) assim:

    npm install --global gulp

### Laravel Elixir

O único passo restante é instalar o Elixir! Com a nova instalação do Laravel, você irá achar o arquivo `package.json` na raíz do projeto. Pense nisto como se fosse seu arquivo `composer.json`, exceto que isto define dependências do Node ao invés do PHP. Você pode instalar dependências citadas anteriormente executando o seguinte:

	npm install

<a name="usage"></a>
## Uso

Agora que você instalou Elixir, você irá estar compilando e concatenando em nenhum momento! O arquvio `gulpfile.js` no diretório raíz dos seus projetos contém todas as suas tarefas Elixir!


#### Compile Less

```javascript
elixir(function(mix) {
	mix.less("app.less");
});
```
No exemplo acima, Elixir asume que seus arquivos Lesse estão armazenados em `resources/assets/less`.

#### Compile Múltiplos Arquivos Less

```javascript
elixir(function(mix) {
	mix.less([
		'app.less',
		'something-else.less'
	]);
});
```

#### Compile Sass

```javascript
elixir(function(mix) {
	mix.sass("app.sass");
});
```
Isto assume que seus arquivos Sass são armazenados no diretório `resources/assets/sass`.

Por padrão, Elixir, por deibaixo dos panos, use a biblíoteca LibSass para compilação. Em algumas casos, pode ser vantajosos em vez de aproveitar a versão Ruby, que, embora mais lento, é mais rico em recursos. Supondo que você tem ambos instalados Ruby e a gem Sass(`gem install sass`), você pode ativar modo-Ruby, assim:

```javascript
elixir(function(mix) {
	mix.rubySass("app.sass");
});
```

#### Compile Sem Source Maps

```javascript
elixir.config.sourcemaps = false;

elixir(function(mix) {
	mix.sass("app.scss");
});
```
Source maps são de fora da caixa. Como tal, para cada arquivo que é compilado, você encontrará um arquivo companheiro `*.css.map` no mesmo diretório. Este mapeamento permite que você, quando estiver debugando. trace seus seletores stylesheet compiladas de volta para o Sass original o Less parcial! Caso você precise desativar esta funcionalidade, no entanto, o código demostrativo acima fará a mágica. 

#### Compile CoffeeScript

```javascript
elixir(function(mix) {
	mix.coffee();
});
```

Isso pressupõe que seus arquivos Coffeescript são armazenados no diretório `recursos / bens / coffee`.

#### Compile Todos os Less and CoffeeScript

```javascript
elixir(function(mix) {
    mix.less()
       .coffee();
});
```

#### Comece Testes PHPUnit

```javascript
elixir(function(mix) {
	mix.phpUnit();
});
```

#### Comece Testes PHPSpec

```javascript
elixir(function(mix) {
	mix.phpSpec();
});
```

#### Combine Stylesheets

```javascript
elixir(function(mix) {
	mix.styles([
		"normalize.css",
		"main.css"
	]);
});
```
Caminhos passados para este método são relativos ao diretório `resources/css`.

#### Combinar Stylesheets e salvar a um Directório Personalizado

```javascript
elixir(function(mix) {
	mix.styles([
		"normalize.css",
		"main.css"
	], 'public/build/css/everything.css');
});
```

#### Combinar Stylesheets partir de um diretório base personalizado

```javascript
elixir(function(mix) {
	mix.styles([
		"normalize.css",
		"main.css"
	], 'public/build/css/everything.css', 'public/css');
});
```
O terceiro argumento tanto para`styles` e métodos `scripts` determinam o diretório relativo para todos os caminhos passados aos métodos.

#### Combine todos os estilos em um diretório

```javascript
elixir(function(mix) {
	mix.stylesIn("public/css");
});
```

#### Combinar Scripts

```javascript
elixir(function(mix) {
	mix.scripts([
		"jquery.js",
		"app.js"
	]);
});
```
Novamente, isto pressupõe que todos os caminhos são relativos ao diretório `resources/js`.

#### Combine todos os scripts em um diretório

```javascript
elixir(function(mix) {
	mix.scriptsIn("public/js/some/directory");
});
```

#### Combine vários conjuntos de Scripts

```javascript
elixir(function(mix) {
    mix.scripts(['jquery.js', 'main.js'], 'public/js/main.js')
       .scripts(['forum.js', 'threads.js'], 'public/js/forum.js');
});
```

#### Versão / Use Hash em um Arquivo

```javascript
elixir(function(mix) {
	mix.version("css/all.css");
});
```
Isto irá adicionar um hash exclusico para o nome do arquivo, permitindo cache-busting. Por exemplo, o nome do arquivo gerado irá parecer com algo assim: `all-16d570a7.css`.

Dentro das suas views, você pode usar a função `elixir()` para carregar apropriadamente ativo hash. Aqui vai um exemplo:

```html
<link rel="stylesheet" href="{{ elixir("css/all.css") }}">
```
Por trás das cenas, a função `elixir()` irá determinar o nome do arquivo hash que deve ser incluído. Você sentiu o peso saindo dos seus ombros, não sentiu ?

Você pode também passar um array para o método `version`, para versionar arquivos múltiplos.

```javascript
elixir(function(mix) {
	mix.version(["css/all.css", "js/app.js"]);
});
```

```html
<link rel="stylesheet" href="{{ elixir("css/all.css") }}">
<script src="{{ elixir("js/app.js") }}"></script>
```

#### Copiar um arquivo para um novo local

```javascript
elixir(function(mix) {
	mix.copy('vendor/foo/bar.css', 'public/css/bar.css');
});
```

#### Copie um diretório inteiro para um novo local

```javascript
elixir(function(mix) {
	mix.copy('vendor/package/views', 'resources/views');
});
```

#### Comece Browserify

```javascript
elixir(function(mix) {
	mix.browserify('index.js');
});
```
Quer requerer módulos no browser? Esperando usar EcmaScript 6(Javascript 6) mais cedo ou mais tarde? Precisa de um transformador JSX bulti-in(nativo)? Se sim, [Browserify](http://browserify.org/), juntamente com a tarefa Elixir `browserify`, isto irá lidar com esses trabalhos facilmente. 

Esta tarefa assume que seus scripts estão armazenados no diretório `resources/js`, embora você seja livre para substituir o padrão. 

#### Encademento de Métodos
Claro que, você pode encadear quase todos os Métodos Elixir juntos para contruir sua receita:

```javascript
elixir(function(mix) {
    mix.less("app.less")
       .coffee()
       .phpUnit()
       .version("css/bootstrap.css");
});
```

<a name="gulp"></a>
## Gulp

Agora que foi dito a você que para executar tarefas Elixir, você apenas precisar iniciar o Gulp por meio de linha de comando. 

#### Executar todas tarefas registradas de uma vez

	gulp

#### Verifique Mudanças nos Assets

	gulp watch

#### Apenas Compile Scripts

	gulp scripts

#### Apenas Compile Styles

	gulp styles

#### Verifique Mudanças nos Testes e Classes PHP

	gulp tdd

> **Nota:** Todas tarefa irão assumir que estão no ambiente de desenvolvimento, e irão excluír minificação. No ambiente de produção, use o comando `gulp --production`.

<a name="extensions"></a>
##  Tarefas customizdas and Extensões


Algumas vezes, você pode querer ligar sua propria tarefas Gulp no Elixir. Talvez você tenha uma parte especial da funcionalidade que você gostaria que o Elixir misture e verifique para você. Sem problema!

Como exemplo, imagine que vocẽ tenha tarefas gerais que simplemente fale um pouco de texto quando chamado. 

```javascript
gulp.task("speak", function() {
	var message = "Tea...Earl Grey...Hot";

	gulp.src("").pipe(shell("say " + message));
});
```
Fácil suficiente. De uma linha de comando,você pode, claro, chamar `gulp speak` para iniciar a tarefa. Para adicionar isto ao Elixir, use o método  `mix.task()`.

```javascript
elixir(function(mix) {
    mix.task('speak');
});
```
É isso ai! Agora, cada vez que você executar Gulp, sua tarefa customizada "speak" será executada jutamente de qualquer outra tarefa Elixir que você tenha misturado. Para adicionalmente retgistrar um verificador, então suas tarefas customizada irão ser reiniciadas cada vez um ou mais arquivos são modificado, você pode passar uma expressão regular como o segundo argumento. 

```javascript
elixir(function(mix) {
    mix.task('speak', 'app/**/*.php');
});
```

Adicionando isto como um segundo argumento, nos instruídmo o Elixir a reiniciar a tarefa "speak" cada vez que arquivos PHP no diretório "app/" é salvo. 

Para ter ainda mais flexibilidade, você pode criar extensões Elixir completas. Usando o exemplo anteriro, você pode escrever extensões, assim:


```javascript
var gulp = require("gulp");
var shell = require("gulp-shell");
var elixir = require("laravel-elixir");

elixir.extend("speak", function(message) {

	gulp.task("speak", function() {
		gulp.src("").pipe(shell("say " + message));
	});

	return this.queueTask("speak");

 });
```
Note que nos `extendemos` API do Elixir passando o nome que nos irémos referênciar dentro do nos arquivo Gulp, bem como uma função callback que irá criar uma tarefa Gulp. 

Como Antes, ser você quiser suas tarefas customizadas sejam monitoradas, então registre um verificador.

```javascript
this.registerWatcher("speak", "app/**/*.php");
```

Esta linha designa que quando qualquer arquivo que base com a expressão regular, `app/**/*.php`, é modificado, nos vamos querer inicar a tarefa `speak`.

É isto! Você pode tanto colocar isto no topo do seu arquivo Gulp(Gulpfile), ou invés extraia isso para um arquivo de tarefas customizado. Se você escolher pela última abordagem, simplesmente requira isto no seu arquivo Gulp(Gulpfile), assim:

```javascript
require("./custom-tasks")
```

Você terminou! Agora, você pode misturar no.

```javascript
elixir(function(mix) {
	mix.speak("Tea, Earl Grey, Hot");
});
```
Com esta adição, cada vez que você inicar o Gulp. Picard irá pedir um chá (oi?)
