# Eloquent ORM

- [Introdução](#introduction)
- [Uso Básico](#basic-usage)
- [Atribuição em Massa](#mass-assignment)
- [Inserindo(Insert), Atualizando(Update), Removendo(Delete)](#insert-update-delete)
- [Exclusão Lógica](#soft-deleting)
- [Timestamps](#timestamps)
- [Escopo de Consultas](#query-scopes)
- [Escopo Global](#global-scopes)
- [Relacionamentos](#relationships)
- [Consultando Relacionamentos](#querying-relations)
- [Eager Loading(Carregamento Ansioso)](#eager-loading)
- [Inserindo em Modelos Relacionados](#inserting-related-models)
- [Atualizando Timestamps do modelo Pai](#touching-parent-timestamps)
- [Trabalhando com Tabelas Pivot](#working-with-pivot-tables)
- [Coleção](#collections)
- [Accessors & Mutators](#accessors-and-mutators)
- [Mutators de Data](#date-mutators)
- [Conversão De Atributos](#attribute-casting)
- [Eventos de Modelos](#model-events)
- [Observers de Modelos](#model-observers)
- [Geração de URL a partir do Modelo](#model-url-generation)
- [Convertendo para Arrays / JSON](#converting-to-arrays-or-json)

<a name="introduction"></a>
## Introdução
O Eloquent ORM incluido com o Laravel fornece uma bonita, e simples implementação ActiveRecord para trabalhar com o seu banco de dados. Cada trabala do banco de dados tem a um "Modelo" correspondente que é usado para interagir com determinada tabela.

Antes de começar, esteja seguro de configurar a conexão do banco de dados em `config/database.php`.  

<a name="basic-usage"></a>
## Uso Básico

Para começar, crie um modelo Eloquent. Models tipicamente estão alocados no diretório `app`, mas sinta-se livre para alocar em qualquer lugar que possar auto-carregado de acordo com o seu arquivo  `composer.json`. Todos os modelos eloquent herdam de `Illuminate\Database\Eloquent\Model`.

#### Definindo um Modelo Eloquent 

	class User extends Model {}

Você pode também gerar um modelo Eloquent usando o comando `make:model`:

	php artisan make:model User

Note que nos não dizemos ao Eloquent que tabela ele deve usar para nosso modelo  `User`. O "snake case"(escrita separada por underline "_"), o nome no plural da classe que será usada como o nome da tabela, a menos que outro nome seja explicitamente especificado. Então, neste caso, o Eloquent assumirá que o modelo `User` armazenará registro na tabela `users`. Você pode especificar uma tabela customizada definindo a propriedade `table` no seu modelo:

	class User extends Model {

		protected $table = 'my_users';

	}

> **Nota:** O Eloquent irá também assumir que cada tabela tem a chave primária nomeada como `id`. Você pode definir a propriedade `primaryKey` para sobrescrever esta convenção. Da mesma forma, você pode definir a propriedade `connection` para sobrescrever o nome da conexão de banco de dados que deve ser usada quando se estiver utilizando o modelo. 

Uma vez que o modelo é definido, você está pronto para compeçar a recuperar e crair registros na sua tabela. Note que você irá precisar por padrão alocar as colunas `updated_at` e `created_at` na sua tabela. Se você não desejar ter estas colunas automaticamente mantidas, defina a propriedade `$timestamps` do seu modelo como  `false`.

#### Recuperando Todos os Registros

	$users = User::all();

#### Recuperando Registro Pela Chave Primária. 

	$user = User::find(1);

	var_dump($user->name);

> **Note:** Todos os métodos disponíveis no [query builder](/docs/queries) também estão disponíves quando estiver consultando modelos. 

#### Recuparando Registros de Um Modelo Pela Chave Primária Ou Levante um Exceção 

Algumas vezes você pode desejar levantar uma exceção se o modelo não for encontrado. Para fazer isto, você pode usar o método `firstOrFail`:

	$model = User::findOrFail(1);

	$model = User::where('votes', '>', 100)->firstOrFail();

Fazendo isto permitirá que você pegue a exeção então você pode registar(log) e monstrar uma página de erro se necessário. Para pegar o `ModelNotFoundException`, adicionar aluma lógica ao seu arquivo`app/Exceptions/Handler.php`.

	use Illuminate\Database\Eloquent\ModelNotFoundException;

	class Handler extends ExceptionHandler {

		public function render($request, Exception $e)
		{
			if ($e instanceof ModelNotFoundException)
			{
				// Custom logic for model not found...
			}

			return parent::render($request, $e);
		}

	}

#### Consultando Usando Modelos Eloquent

	$users = User::where('votes', '>', 100)->take(10)->get();

	foreach ($users as $user)
	{
		var_dump($user->name);
	}

#### Agregados do Eloquent

É claro que, você pode também usar o construtor de consultas(query builder) para agregar funções.

	$count = User::where('votes', '>', 100)->count();

Se você não incapacitado de gerar a consulta que você precisa pela interface fluente, sinta-se a vontade para usar `whereRaw`:

	$users = User::whereRaw('age > ? and votes = 100', [25])->get();

#### Dividindo(Chunking) Resultados

Se você precisa processar muitos(milhares) de registros Eloquent, usando o comando `chunk` irá lhe permitir fazer isto sem consumir toda a sua memoria RAM:

	User::chunk(200, function($users)
	{
		foreach ($users as $user)
		{
			//
		}
	});
	
O primeiro argumento(parâmetro) passado para o método é o número de registro que você deseja receber por "chunk" (divisão de resultados/partes). A Closure passada como o segundo argumento irá ser chamada a cada parte "chunk" que for recuperada do banco de dados.

#### Especificando a Conexão da Consulta

Você pode também especificar que a conexão do banco de dados deve ser usada quando estiver executando uma Consulta Eloquent. Simplesmente use o método `on`:

	$user = User::on('connection-name')->find(1);

Se você estiver usando [leia / escreva conexões](/docs/{{version}}/database#read-write-connections), você pode forçar a query a usar a conexão "write" com o seguinte método:

	$user = User::onWriteConnection()->find(1);

<a name="mass-assignment"></a>
## Atribuição em Massa

Quando estivermos criando um novo modelo, você passa um array de atributos para o construtor do mesmo. Estes atributos são atribuidos aos modelos via mass-assigment (atribuição em massa). Isto é conveniente; no entanto, pode ser uma **séria** preocupação com segurança quando cegamente estivermos passando o input do usuário a um modelo. Se o input do usuário é cegamente passado ao modelo, o usuário é livre para modificar **qualquer** e **todo** atributo do modelo.Por estas razões, todos os modelos Eloquent são protegidos contra mass-assignment por padrão. 

Para começar, defina as propriedades `fillable` or `guarded` no seu modelo.

#### Definindo Atributos Fillable em um Modelo 

A propriedade `fillable` especifica que atributos devem ser atribuidos em massa. Isto pode ser definido na classe ou a nível de instância. 

	class User extends Model {

		protected $fillable = ['first_name', 'last_name', 'email'];

	}
Neste exemplo, apenas os três atributos listados serão atribuídos em massa. 

#### Definindo Atributos Guardados em um Modelo

O inverso de `fillable` é `guarded`, e server como uma "black-list" (lista negra) ao invés de uma "white-list" (listra branca)

	class User extends Model {

		protected $guarded = ['id', 'password'];

	}

>**Nota:** Quando usamos `guarded`, você ainda nunca deve passar `Input::get()` ou qualquer raw array(array sem tratamento de acordo com os atributos) de entrada controlado pelo o usuário para os métodos `save` ou `update`, como qualquer coluna que não for guarded poderá poderá ser atualizada. 

#### Bloqueando Todos os Atributos de Atribuição em Massa 

No exemplo acima, os atributos `id` e `password` **não** podem ter atribuição em massa. Todos os outros atributos irão ser atribuídos em massa. Você pode também bloquear **todos** os atributos de atribuição em massa usando a propriedade guard:

	protected $guarded = ['*'];

<a name="insert-update-delete"></a>
## Inserindo(Insert), Atualizando(Update), Removendo(Delete)

Para criar um novo registro no banco de dados de um modelo, simplesmente crie uma instância de um modelo e chame o método `save`.

#### Salvando um Novo Modelo

	$user = new User;

	$user->name = 'John';

	$user->save();

> **Nota:** Tipicamente, seu modelo Eloquent irá ter chaves auto-incrementais. No entanto, se você quiser especificar suas próprias chaves, defina a propriedade `incrementing` no seu modelo como `false`.

Você também pode usar o método `create` para salvar um novo modelo um uma única linha. A instância do modelo inserido será retornada para você a partir do método. No entanto, antes de faze-lo, você precisará espeficar também os atributos `fillable` ou `guarded` no modelo, como todos os modelos Eloquent protegidos contra atribuição-em-massa.  

Depois de salver ou criar um novo modelo que use auto-incrementação de IDs, você pode recuperar o ID acessando o atributo `id` do objeto:

	$insertedId = $user->id;

#### Definindo Atributos Guarded No Modelo

	class User extends Model {

		protected $guarded = ['id', 'account_id'];

	}

#### Usando O Método Create Do Modelo

	// Create a new user in the database...
	$user = User::create(['name' => 'John']);

	// Retrieve the user by the attributes, or create it if it doesn't exist...
	$user = User::firstOrCreate(['name' => 'John']);

	// Retrieve the user by the attributes, or instantiate a new instance...
	$user = User::firstOrNew(['name' => 'John']);

#### Atualizadno Um Modelo Recuperado

Para atualizar um modelo, você pode recuperar isto, mudando o atributo, e usando o método `save`:

	$user = User::find(1);

	$user->email = 'john@foo.com';

	$user->save();

#### Salvando Um Modelo E Relacionamentos 

Algumas vezes você pode desejar salvar não apenas o modelo, mas também todos os seus relacionamentos. Para fazê-lo, você pode usar o método `push`:

	$user->push();

Você pode também executar atualizações como consultas em um conjunto de modelos: 

	$affectedRows = User::where('votes', '>', 100)->update(['status' => 2]);

> **Nota:** Nenhum evento do modelo é executado quando se estiver atualizando um conjunto de modelos via o query builder(contrutor de consultas) do Eloquent.

#### Deletando Um Modelo Existente

Para Deletar um modelo, simplesmente chame o método delete `delete` na instância:

	$user = User::find(1);

	$user->delete();

#### Deletando um Modelo Existente Pela Chave

	User::destroy(1);

	User::destroy([1, 2, 3]);

	User::destroy(1, 2, 3);

Claro que, você pode também executar uma query delete em um conjunto de modelos:

	$affectedRows = User::where('votes', '>', 100)->delete();

#### Atualizando Apenas os Timestamps dos Modelos

Se você desejar apenas atualizar os timestamps em modelo, você pode usar o método `touch`:

	$user->touch();

<a name="soft-deleting"></a>
## Exclusão Lógica

Quando se esta excluindo lógicamente um modelo, isto na verdade não está sendo removido do seu banco de dados. Ao invés, o timestamp `deleted_at` é definido no registro do banco de dados. Para ativar a exclusão lógica para um modelo, aplique `SoftDeletes` no mesmo:

	use Illuminate\Database\Eloquent\SoftDeletes;

	class User extends Model {

		use SoftDeletes;

		protected $dates = ['deleted_at'];

	}

Para adicionar a coluna `deleted_at` na sua tabela, você pode usar o método `softDeletes` a partir de uma migração:

	$table->softDeletes();

Agora, quando você chamar o método `delete` no modelo, a coluna `deleted_at` irá ser definida para a data atual. Quando estivermos consultado um modelo que usar exclusão lógica, os modelos "deletados" não serão incluídos nos resultados das consultas. 

#### Forçando os Modelos Excluídos Logicamente a Aparecerem Nas Consultas

Para forçar modelos excluídos logicamente a aparecer em resultados das consultas, use o método `withTrashed` na consulta:

	$users = User::withTrashed()->where('account_id', 1)->get();

O método `withTrashed` pode ser usado em relacionamentos definidos:

	$user->posts()->withTrashed()->get();

Se você desejar **apenas** receber modelos excluídos logicamente nos seus resultados de consulta, você pode usar o método 
`onlyTrashed`:

	$users = User::onlyTrashed()->where('account_id', 1)->get();

Para recuperar um modelo excluído logicamente ao seu estado ativo, user o método `restore`:

	$user->restore();

Você pode também usar o método `restore` em uma query:

	User::withTrashed()->where('account_id', 1)->restore();

Como com o método `withTrashed`, o método `restore` também pode ser usado em relacionamentos:

	$user->posts()->restore();

Se você verdadeiramente deseja remover um modelo do seu banco de dados, você pode usar o método `forceDelete`:

	$user->forceDelete();

O método `forceDelete` também trabalhar em relacionamentos:

	$user->posts()->forceDelete();

Para determinar se uma instância de um modelo dado foi excluída logicamente, você pode usar o método `trashed`:

	if ($user->trashed())
	{
		//
	}

<a name="timestamps"></a>
## Timestamps

Por padrão, o Eloquent irá manter as colunas `created_at` e `updated_at` na tabela do seu bando de dados automaticamente. Simplesmente adicionce essas colunas `timestamp` a sua tabela e o Eloquent tomará conta do resto. Se você não desejar que o Eloquent não mantenha estas colunas, adicione a seguinte propriedade para o seu modelo:

#### Disabling Auto Timestamps

	class User extends Model {

		protected $table = 'users';

		public $timestamps = false;

	}

#### Providing A Custom Timestamp Format

Se você deseja customizar o formato dos seus "timestamps", você pode sobrescrever o método `getDateFormat` no seu modelo:

	class User extends Model {

		protected function getDateFormat()
		{
			return 'U';
		}

	}

<a name="query-scopes"></a>
## Escopo de Query(Consulta)

#### Definindo o Escopo de Uma Consulta 

Escopo lhe permite reúsar faicilmente a lógica de cosulta nos seus modelos. Para definir o escopo, simplesmente préfixe o método do modelo com `scope`:

	class User extends Model {

		public function scopePopular($query)
		{
			return $query->where('votes', '>', 100);
		}

		public function scopeWomen($query)
		{
			return $query->whereGender('W');
		}

	}

#### Utilizando o Escopo de Query

	$users = User::popular()->women()->orderBy('created_at')->get();

#### Escopo Dinâmico

Algumas vezes você pode desejar definir o escopo que aceite parâmetros. Apenas adicione seus parâmetros a função de escopo:

	class User extends Model {

		public function scopeOfType($query, $type)
		{
			return $query->whereType($type);
		}

	}

Então passe o parâmetro na chamada do escopo:

	$users = User::ofType('member')->get();

<a name="global-scopes"></a>
## Escopo Global

Algumas vezes você pode desejar definir o escopo que aplique a todas as consultas feitas em um modelo. Na essência, isto é 
como a característica "exclusão lógica"(sof delete) própria do Eloquent funciona. Escopos globais são definidos usando a combinação de traits PHP e a implementação de `Illuminate\Database\Eloquent\ScopeInterface`.

Primeiro, vamos definir a trait. Para este exemplo, nos irémos usar o `SoftDeletes` que vem com o Laravel:

	trait SoftDeletes {

		/**
		 * Boot the soft deleting trait for a model.
		 *
		 * @return void
		 */
		public static function bootSoftDeletes()
		{
			static::addGlobalScope(new SoftDeletingScope);
		}

	}

Se um modelo Eloquent usa trait que tem um método que corresponda a convenção de nomeclatura `bootNameOfTrait` (bootNomeDaTrait), esse método trait irá ser chamado quando o modelo Eloquent é inicializado, dando a você a oportunidade de registrar um escopo global, ou fazer quaalquer outra coisa que você queira. O escopo tem que implementar a `ScopeInterface`, que especifica dois métodos: `apply` e `remove`.

O método `apply` recebe um objeto query builder(construtor de consultas) `Illuminate\Database\Eloquent\Builder` e o `Modelo` que é aplicado, e é responsável por adicionar qualquer cláusula `where` adicional que o escopo deseja adicionar. O método `remove` também recebe um objeto `Builder` construtor e um modelo e é responsável por reversão de ações feitas pelo método `apply`. Em outras palavras, `remove` deve remover as cláusulas `where` (ou qualquer outra cláusula) que foi adicionada. Portanto, para o nosso  `SoftDeletingScope`, os métodos devem parecer como algo assim:


	/**
	 * Apply the scope to a given Eloquent query builder.
	 *
	 * @param  \Illuminate\Database\Eloquent\Builder  $builder
	 * @param  \Illuminate\Database\Eloquent\Model  $model
	 * @return void
	 */
	public function apply(Builder $builder, Model $model)
	{
		$builder->whereNull($model->getQualifiedDeletedAtColumn());

		$this->extend($builder);
	}

	/**
	 * Remove the scope from the given Eloquent query builder.
	 *
	 * @param  \Illuminate\Database\Eloquent\Builder  $builder
	 * @param  \Illuminate\Database\Eloquent\Model  $model
	 * @return void
	 */
	public function remove(Builder $builder, Model $model)
	{
		$column = $model->getQualifiedDeletedAtColumn();

		$query = $builder->getQuery();

		foreach ((array) $query->wheres as $key => $where)
		{
			// If the where clause is a soft delete date constraint, we will remove it from
			// the query and reset the keys on the wheres. This allows this developer to
			// include deleted model in a relationship result set that is lazy loaded.
			if ($this->isSoftDeleteConstraint($where, $column))
			{
				unset($query->wheres[$key]);

				$query->wheres = array_values($query->wheres);
			}
		}
	}

<a name="relationships"></a>
## Relacionamentos

É claro que, suas tabelas do banco de dados provavelmente tem algum relacionamento uma com as outras, Por exemplo, um post de um blog pode muitos comentário, ou uma compra pode ser relacionada a um usuário que a realizou. Eloquent faz o gerenciamento e o funcionamento com esses relacionamentos de forma fácil. Laravel suporta vários tipos de relacionamentos. 

- [Um para Um](#one-to-one)
- [Um para Many](#one-to-many)
- [Muitos para Muitos](#many-to-many)
- [Tem Muitos Através](#has-many-through)
- [Relações Polimórficas](#polymorphic-relations)
- [Muitos para Muitos Relações Polimórficas](#many-to-many-polymorphic-relations)

<a name="one-to-one"></a>
### Um Para Um (One to One)

#### Definindo o Relação Um para Um

O relacioamento um-para-um é um relacionamento muito básico. Por exemplo, o modelo `User` pode ter um `Phone`. Nos podemos definir este relacionamento no Eloquent:


	class User extends Model {

		public function phone()
		{
			return $this->hasOne('App\Phone');
		}

	}

O primeiro argumento passado para o método `hasOne` é o nome do modelo relacionado. Uma vez que o relaciomento é definido, nos podemos recuperar isto usando as [propriedades dinâmicas](#dynamic-properties) do Eloquent:

	$phone = User::find(1)->phone;

O SQL realizado por esta declaração irá ser como o seguinte:

	select * from users where id = 1

	select * from phones where user_id = 1

Tome nota que o Eloquent assume que a chave estrangeira do relacionamento é baseado no nome do modelo. Neste caso, o modelo `Phone` é assume usar a chave estrangeira  `user_id`. Se você desejar sobreescrever esta convenção, você pode passar um segundo argumento para o método `hasOne`. Além disso, você pode passar um terceiro argumento para o método para especificar qual coluna local que pode ser usada para a associação:

	return $this->hasOne('App\Phone', 'foreign_key');

	return $this->hasOne('App\Phone', 'foreign_key', 'local_key');

#### Definindo O Inverso de uma Relação 

Para definir o inverso de um relaciomento no modelo `Phone`, nos usamos o método `belongsTo`:

	class Phone extends Model {

		public function user()
		{
			return $this->belongsTo('App\User');
		}

	}

Neste exemplo acima, o Eloquent irá olhar para coluna `user_id` na tabela `phones`. Se você quiser definir uma coluna de chave estrangeira diferente, você pode passar isto como um segundo argumento para o método `belongsTo`:

	class Phone extends Model {

		public function user()
		{
			return $this->belongsTo('App\User', 'local_key');
		}

	}

Adicionalmente, você passa um terceiro parâmetro que especifica o nome da coluna associada na tabela pai:

	class Phone extends Model {

		public function user()
		{
			return $this->belongsTo('App\User', 'local_key', 'parent_key');
		}

	}

<a name="one-to-many"></a>
### Um para Muitos(One to Many)

Um exemplo de relacionamento um-para-muito é um post de um blog que "has many"(tem muitos) comentários. Nos podemos modelar este relacionamento assim:

	class Post extends Model {

		public function comments()
		{
			return $this->hasMany('App\Comment');
		}

	}

Agora nos podemos acessar os comentários do post por meio das [propriedades dinâmicas](#dynamic-properties):

	$comments = Post::find(1)->comments;

Se você precisar adicionar mais uma constraint aos comentários que são recuperados, você pode chamar o método `comments` e continuar encadeando condições:

	$comments = Post::find(1)->comments()->where('title', '=', 'foo')->first();

Novamente, você pode sobreescrever a chave estrangeira convencional passando um segundo parâmetro como argumento para o método `hasMany`. E, como na relação `hasOne`, a coluna local pode também ser especificada.

	return $this->hasMany('App\Comment', 'foreign_key');

	return $this->hasMany('App\Comment', 'foreign_key', 'local_key');

#### Definindo a Relação Inversa.

Para definir o relacionamento inverso o modelo `Comment`, nos usamos o método `belongsTo`:

	class Comment extends Model {

		public function post()
		{
			return $this->belongsTo('App\Post');
		}

	}

<a name="many-to-many"></a>
### Muitos para Muitos(Many To Many)

Relações muitos-para-muitos é o tipo de relacionamento mais complicado. Um exemplo de tal relacionamento é o usuário com vários perfis, onde os perfis também divididos entre outros usuários. Por exemplo, muitos usuário pode ter o perfil de "Admin". Três tabelas de banco de dados são necessárias para este relacionamento: `users`, `roles`, e `role_user`. A tabela role_user é derivada a partir da ordem alfabética dos nomes dos modelos relacionados, e devem ter as colunas `user_id` and `role_id`.

Nos podemos definir a relação de muito-para-muitos usando o método `belongsToMany`:

	class User extends Model {

		public function roles()
		{
			return $this->belongsToMany('App\Role');
		}

	}

Agora, nos podemos recuperar os perfis através do modelo `User`:

	$roles = User::find(1)->roles;

Se você gostar de usar um nome de tabela não-convencional para sua tabela pivot, você pode passar isto como o segundo parâmetro do método `belongsToMany`:

	return $this->belongsToMany('App\Role', 'user_roles');

Você pode tambeḿ sobrescrever a chave convencional associada:

	return $this->belongsToMany('App\Role', 'user_roles', 'user_id', 'foo_id');

É claro que, você pode também definir o inverso do relacionamento no modelo `Role`:

	class Role extends Model {

		public function users()
		{
			return $this->belongsToMany('App\User');
		}

	}

<a name="has-many-through"></a>
### Tem Muitos Através

A relação "tem muitos através" fornece um atalho conveniente para acessar relações distantes por meio de relações itermediárias. Por exemplo, o modelo `Country` pode ter muitos `Post` por meio do modelo `User`. A tabela para este relacionamento deve parecer com isto:

	countries
		id - integer
		name - string

	users
		id - integer
		country_id - integer
		name - string

	posts
		id - integer
		user_id - integer
		title - string


Mesmo que a tabela `posts` não contenha a coluna `country_id`, a relação `hasManyThrough`(tem-muitos-atraves) nos permitirá acessar os post de um paíse(coutry's) via `$country->posts`. Vmoas definir o relacionamento.

	class Country extends Model {

		public function posts()
		{
			return $this->hasManyThrough('App\Post', 'App\User');
		}

	}

Se gostar de especificar manualmente as chaves do relacionamento, você pode passá-los como um terceiro e quarto parâmetro para o método:

	class Country extends Model {

		public function posts()
		{
			return $this->hasManyThrough('App\Post', 'App\User', 'country_id', 'user_id');
		}

	}

<a name="polymorphic-relations"></a>
### Relações Polimórficas

Relações polimórficas permitem ao modelo pertencer a mais do que um outro modelo, em uma única associação. Por exemplo, você pode ter um modelo "photo"(foto) que pertece  ou modelo "staff"(pessoal) ou ao modelo "order"(pedido). Nos devemos definir esta relação assim:


	class Photo extends Model {

		public function imageable()
		{
			return $this->morphTo();
		}

	}

	class Staff extends Model {

		public function photos()
		{
			return $this->morphMany('App\Photo', 'imageable');
		}

	}

	class Order extends Model {

		public function photos()
		{
			return $this->morphMany('App\Photo', 'imageable');
		}

	}

#### Recuperando Relações Polimórficas

Aora, nos podemos recuperar as fotos para um menbro da equipe/pessoal ou para um pedido:

	$staff = Staff::find(1);

	foreach ($staff->photos as $photo)
	{
		//
	}

#### Recuperando O nodo da Relação Polimórfica. 

No entando, a verdadeira mágica "polimórfica" é quando você acessa "staff"(pessoal) ou "order"(pedido) a partir do modelo `Photo`:

	$photo = Photo::find(1);

	$imageable = $photo->imageable;

A relação `imageable` no modelo `Photo` irá retornar a instância de `Staff` ou `Order`. dependendo do tipo de modelo que possui a foto.

#### Estrutura da Tabela de Relação Polimórfica 

Para ajudar a entender como isto funciona, vamos explorar a estrutura do banco de dados para uma relação polimórfica:

	staff
		id - integer
		name - string

	orders
		id - integer
		price - integer

	photos
		id - integer
		path - string
		imageable_id - integer
		imageable_type - string


Os campos chaves a serem notados aqui são `imageable_id` e `imageable_type` na tabela de `photos`(fotos). O ID irá conter o valor do ID, neste exemplo, o proprietário de "staff" ou "order", enquando o tipo irá conter o nome da classe do modelo proprietário. Isto é o que permite que o ORM determine qual é o tipo do modelo proprietário para retornar quando a relação `imageable` é acessada. 

<a name="many-to-many-polymorphic-relations"></a>
### Muitos para Muitos Em Relações Polimórficas

#### Estrutura de Tabelas de Relações Polimórficas de Muitos Para Muitos 

Em adição a relações polimórficas tradicionais, você pode também especificar relações polimórficas de muitos-para-muitos. Por exemplo, um `Post` de um blog e um modelo `Video` poderiam compartilhar uma relação polimórfica para um modelo `Tag`. Primeiramente, vamos eximinar a estrutura das tabelas:

	posts
		id - integer
		name - string

	videos
		id - integer
		name - string

	tags
		id - integer
		name - string

	taggables
		tag_id - integer
		taggable_id - integer
		taggable_type - string

Em seguida, nos estamos prontos para configurar os relacionamentos no modelo. Os modelos `Post` e `Video` irão ambos ter o relacionamento `morphToMany`  por meio do método `tags`

	class Post extends Model {

		public function tags()
		{
			return $this->morphToMany('App\Tag', 'taggable');
		}

	}

O modelo `Tag` pode definir o método para cada um de seus relacionamento:

	class Tag extends Model {

		public function posts()
		{
			return $this->morphedByMany('App\Post', 'taggable');
		}

		public function videos()
		{
			return $this->morphedByMany('App\Video', 'taggable');
		}

	}

<a name="querying-relations"></a>
## Consultando Relacionamentos

#### Consultando Relacionamentos E Limitando seus Resultados

Quando se esta acessandos os registros de um demolo, você pode desejar limitar os resultandos baseados nas existência do relacionamento. Por exemplo, você pode desejar pegar todos os posts de um blog que tem pelo menos um comentário. Para isto, você pode usar o método `has`:

	$posts = Post::has('comments')->get();

Você pode também especificar um operador e um contador:

	$posts = Post::has('comments', '>=', 3)->get();

Declações `has` aninhadas podem também ser construídas usando a notação "dot":

	$posts = Post::has('comments.votes')->get();

Se você precisar de ainda mais poder, você pode usar os métodos `whereHas` e `orWhereHas` para por condições "where" nas suas consultas `has`: 

	$posts = Post::whereHas('comments', function($q)
	{
		$q->where('content', 'like', 'foo%');

	})->get();

<a name="dynamic-properties"></a>
### Propriedades Dinâmicas

Eloquent permite que você acesse as suas relações via propriedades dinâmicas. Eloquent irá automaticamente carregar os relacionamento para você, e é mesmo inteligente o suficiente para saber se deve chamar o método `get` (para um-para-muitos relacionamentos) or `first` (para um-para-um relacionamentos). Isto será então acessível via propriedade dinâmica pelo o mesmo nome como relação. Por exemplo, como o seguinte modelo  `$phone`:

	class Phone extends Model {

		public function user()
		{
			return $this->belongsTo('App\User');
		}

	}

	$phone = Phone::find(1);
Ao invés de ecoar o e-mail do usuário como este:

	echo $phone->user()->first()->email;

Isto pode ser encurtado facilmente:

	echo $phone->user->email;

> **Nota:** Relacionamentos que retornam muitos resultados irão retornar a instância da classe `Illuminate\Database\Eloquent\Collection`.

<a name="eager-loading"></a>
## Eager Loading(Carregamento Ansioso)

O carregamento ansioso existe para aliviar o problemas de consultas N + 1. Por exemplo, considere o modelo `Book` que é relacionado a `Author`. O relacionamento é definido assim:

Eager loading exists to alleviate the N + 1 query problem. For example, consider a `Book` model that is related to `Author`. The relationship is defined like so:

	class Book extends Model {

		public function author()
		{
			return $this->belongsTo('App\Author');
		}

	}

Agora, considere o seguinte código:

	foreach (Book::all() as $book)
	{
		echo $book->author->name;
	}

Este laço irá executar 1 consulta para recuperar todos os livros da tabela, então outra consulta para cada livro para recuperar o autor. Então, se tivermos 25 livros, este laço deve executar 26 consultas.

Felizment, nos podemos usar o carregamento ansioso para drasticamente reduzir o número de consultas. Os relacionamentos que devem ser carregados asiosamente podem ser especificados via o método `with`:

	foreach (Book::with('author')->get() as $book)
	{
		echo $book->author->name;
	}

No loop acima, apenas duas consultas serão executadas:

	select * from books

	select * from authors where id in (1, 2, 3, 4, 5, ...)

O uso sábio do carregamente ansioso pode drasticamente aumentar a performance da sua aplicação.

Claro que, você pode carregar ansiosamente relacionamentos múltiplos de uma vez:

	$books = Book::with('author', 'publisher')->get();

Você pode até carregar ansiosamente relacionamentos aninhados:

	$books = Book::with('author.contacts')->get();

No exemplo acima, o relacionamento `author` será carregado ansiosamente, e a relação `contacts` do "author" também será carregada.

### Constraints do Carregamento Ansioso

Algumas vezes você pode desejar carregar ansiosamente um relacionamento, mas também especificar uma condição para o carregamento ansioso, aqui vai um exemplo:

	$users = User::with(['posts' => function($query)
	{
		$query->where('title', 'like', '%first%');

	}])->get();

Neste exemplo, nos estamos carregando ansiosamente os posts dos usuários, mas apenas se a coluna título dos posts contém a palavra "first".

É claro que, Closures de arregamente ansioso não estão limitadas a "constraints". Você também pode aplicar ordenações:

	$users = User::with(['posts' => function($query)
	{
		$query->orderBy('created_at', 'desc');

	}])->get();

### Carregamento Ansioso Tardio

Também é possível ansiosamente carregar modelos relacionados diretamente de uma coleção de um modelo já existente. Isto pode ser útil quando se esta descidindo se se deve carregar modelos relacionados ou não, ou em combinação com cache.

	$books = Book::all();

	$books->load('author', 'publisher');

Você pode também passar a Closure para definir constraints na consultas:

	$books->load(['author' => function($query)
	{
		$query->orderBy('published_date', 'asc');
	}]);

<a name="inserting-related-models"></a>
## Inserindo em Modelos Relacionados

#### Anexando o Modelo Relacionado 

Você irá frequentement precisar iserir novos modelos relacionados. Por exemplo, você pode desejar inserir um novo comentário para um post. Ao invés de manualmente definir a chave estrangeira `post_id` no modelo, você pode inserir um novo comentário do seu pai o modelo `Post` diretamente:

	$comment = new Comment(['message' => 'A new comment.']);

	$post = Post::find(1);

	$comment = $post->comments()->save($comment);

No exemplo acima, o campo `post_id` irá automaticamente ser definido no comentário inserido.

Se você precisar salvar múltiplos modelos relacionados:

	$comments = [
		new Comment(['message' => 'A new comment.']),
		new Comment(['message' => 'Another comment.']),
		new Comment(['message' => 'The latest comment.'])
	];

	$post = Post::find(1);

	$post->comments()->saveMany($comments);

### Associando Modelos (Pertence a)

When updating a `belongsTo` relationship, you may use the `associate` method. This method will set the foreign key on the child model:

	$account = Account::find(10);

	$user->account()->associate($account);

	$user->save();

### Inserindo em Modelos Relacionados (Many To Many)

Você pode também inserir modelos relacionados quando estiver trabalhando com relações muitos-para-muitos (many-to-many). Vamos continuar usando nossos modelos `User` e `Role` como exemplos. Nos podemos facilmente anexar novos roles "perfis" um user "usuário" usando o método `attach`.

#### Anexando Modelos Muitos Para Muitos (many-to-many)

	$user = User::find(1);

	$user->roles()->attach(1);

Você pode também passar um array de atributos que devem ser armazenados na tabela pivot pra o relacionamento:

	$user->roles()->attach(1, ['expires' => $expires]);

É claro que, o oposto do método `attach` é `detach`:

	$user->roles()->detach(1);

Ambos `attach` e `detach` também pegam arrays de IDs como input:

	$user = User::find(1);

	$user->roles()->detach([1, 2, 3]);

	$user->roles()->attach([1 => ['attribute1' => 'value1'], 2, 3]);

#### Usando o Sync para Anexar Muitos para Muitos (many-to-many) 

Você pode também usar o método `sync` para anexar modelos relacionados. O método `sync` aceitar um array de IDs para alocar na tabela pivot. Depois que esta operação é completada, apenas os IDs no arrau irão ser intermediadas para a tabela do modelo:

	$user->roles()->sync([1, 2, 3]);

#### Adicionado Dados ao Pivot Quando se está Sincronizando

Você pode também associar outro valor de tabela pivot com os dados IDs:

	$user->roles()->sync([1 => ['expires' => true]]);

Algumas vezes pode pode desejar criar um novo modelo relacionado e anexá-lo a um simples comando. Para essa operação você pode usar o método `save`:

	$role = new Role(['name' => 'Editor']);

	User::find(1)->roles()->save($role);

Neste exemplo, o novo modelo `Role` irá ser salvo e anexado ao modelo "user". Você pode também passar um array de atributos para colocar na tabela que esta sendo associada para esta operação:

	User::find(1)->roles()->save($role, ['expires' => $expires]);

<a name="touching-parent-timestamps"></a>
## Atualizando Timestamps do modelo Pai

Quando um modelo `belongsTo` (pertense) a outro modelo, como um `Comment` (comentário) que pertence a um `Post`, isto é frequentemente útil para atualizar o timestamp pai quando o modelo filho é atualizado. Por exemplo, quando um modelo  `Comment` é atualizado, você pode querer que automaticamente o timestamp `updated_at` do pai `Post` seja atualizado. Eloquent facilita isso. Apenas adicione a propriedade `touches` contendo os nomes dos relacionamento para o modelo filho. 

	class Comment extends Model {

		protected $touches = ['post'];

		public function post()
		{
			return $this->belongsTo('App\Post');
		}

	}

Agora, quando você atualiza o `Comment` (comentário), o `Post` pai irá ter a sua coluna `updated_at` atualizada.

	$comment = Comment::find(1);

	$comment->text = 'Edit to this comment!';

	$comment->save();

<a name="working-with-pivot-tables"></a>
## Trabalhando com Tabelas Pivot

Como você já aprendeu, trabalar com relações muitos-para-muitos requer a presença de uma tabela intermediadora. Eloquent fornece alguns caminhos bastante úteis de interagir com esta tabela. Por exemplo, vamos assumir que nosso objeto `User`(usuários) tem vários objetos `Role`(Perfis) que eles são relacionados. Depois de acessar este relacionamento, nos podemos acessar a tabela pivot nos modelos ?

	$user = User::find(1);

	foreach ($user->roles as $role)
	{
		echo $role->pivot->created_at;
	}

Note que cada modelo `Role` (perfil) que nos recuperamdnso é automaticamente atribuído um atributo `pivot`. Este atributo contém um modelo representando a tabela intermediadora, e pode ser usada como qualquer outro modelo Eloquent.

Por padrão, apenas as chaves poderão estar presentes no objeto `pivot`. Se sua tabela pivot contem atributos extras, você tem que especificá-los quando estiver definindo os relacionamentos.

	return $this->belongsToMany('App\Role')->withPivot('foo', 'bar');

Agora os atributos, `foo` e `bar` serão acessíveis  no nosso objeto `pivot` para o modelo `Role`.

Se você quer que sua tabela pivot tenha automaticamente manutenção dos timestamps `created_at` e `updated_at`, use o método `withTimestamps` na definição do relacionamento:

	return $this->belongsToMany('App\Role')->withTimestamps();

#### Deleando Registros de Uma Tabela Pivot 

Para deletar todos os registros de uma tabela pivot para um modelo, você pode usar o metódo `detach`:

	User::find(1)->roles()->detach();

Note que esta operação não deleta os registros da tabela `roles`, mas apenas da tabela pivot. 

#### Atualizando Um Resgistro Na Tabela Pivot

Algumas vezes você pode precisar atualizar sua tabela pivot, mas não retira-la. Se você deseja atualizar sua tabela pivot ao invés de deletar você pode usar o método  `updateExistingPivot` assim:

	User::find(1)->roles()->updateExistingPivot($roleId, $attributes);

#### Definindo um Modelo Pivot Customizado

Laravel também permite que você definar um modelo Pivot customizado. Para definir um modelo customizado, primeiro crie seu própria classe modelo "Base" e extenda de `Eloquent`. Nos seus outros modelos Eloquent, extenda este modelo base ao invés do modelo base `Eloquent` padrão. No seu modelo base, adicione a seguinte função que retorna uma instância do seu pivot modelo pivot customizado.

	public function newPivot(Model $parent, array $attributes, $table, $exists)
	{
		return new YourCustomPivot($parent, $attributes, $table, $exists);
	}

<a name="collections"></a>
## Coleções

Todos os conjuntos de multi-resultados retornados pelo ELoquent, que através do método `get` ou do  `relacionamento`, irão retornar um objeto collection. Este objeto implementa a interface PHP `IteratorAggregate` então isto pode ser iterada como um array. No entanto, este objeto também tem um variedade de outros métodos úteis para trabalhar com conjuntos de resultados.

#### Checando Se uma Coleção Tem a Chave 

Por exemplo, nos podemos determinar se um conjunto de resultados contém uma dada chave primária, usando o método `contains`:

	$roles = User::find(1)->roles;

	if ($roles->contains(2))
	{
		//
	}

Coleções também podem ser convertidas para um array ou JSON:

	$roles = User::find(1)->roles->toArray();

	$roles = User::find(1)->roles->toJson();

Se a coleção é convertida para o tipo String, isto fará com que ela seja retornada como JSON:

	$roles = (string) User::find(1)->roles;

#### Interando Coleções

Coleções Eloquent também contém alguns método úteis para interar e filtrar os itens que as coleções contém:

	$roles = $user->roles->each(function($role)
	{
		//
	});

#### Filtrando Coleções 

Quando se filtra coleções, o callback provido será usado como callback para [array_filter](http://php.net/manual/en/function.array-filter.php).

	$users = $users->filter(function($user)
	{
		return $user->isAdmin();
	});

> **Nota:** Quando se está filtrando coleções e as convertendo para JSON, tente chamar a função `values` primeiro para resetar as chaves do array.


#### Aplicando o Callback Para Cada Objeto Coleção 

	$roles = User::find(1)->roles;

	$roles->each(function($role)
	{
		//
	});

#### Ordenando Uma Coleção Por Valor

	$roles = $roles->sortBy(function($role)
	{
		return $role->created_at;
	});

	$roles = $roles->sortByDesc(function($role)
	{
		return $role->created_at;
	});

#### Ordenando Uma Coleção Por Valor

	$roles = $roles->sortBy('created_at');

	$roles = $roles->sortByDesc('created_at');

#### Retornando Um Tipo de Coleção Customizada

Algumas vezes, você pode querer retornar objetos coleção customizados com seus próprios métodos adicionados. Você pode especificar isto no seu modelo Eloquent sobrescrevendo o método `newCollection`:

	class User extends Model {

		public function newCollection(array $models = [])
		{
			return new CustomCollection($models);
		}

	}

<a name="accessors-and-mutators"></a>
## Accessors & Mutators

#### Definindo um Accessor

Eloquent fornece a meio conveniente para transformar seus atributos de modelo quando se está recuperando ou os definindo. Simplesmente defina o método `getFooAttribute` no seu modelo para declarar um acessor. Fique ciente que os métodos devem serguir o modelo camel-casing, apesar que as colunas colunas do banco de dados são no formato snake-case:  

	class User extends Model {

		public function getFirstNameAttribute($value)
		{
			return ucfirst($value);
		}

	}

No exemplo acima, a coluna `first_name` tem o acessor. Note que o valor do atributo é passado para o acessor.  

#### Definindo um Mutator

Mutators são declarados de uma forma similar:

	class User extends Model {

		public function setFirstNameAttribute($value)
		{
			$this->attributes['first_name'] = strtolower($value);
		}

	}

<a name="date-mutators"></a>
## Mutators de Data

Por padrão, ELoquent irá converter as colunas `created_at` e `updated_at` para instâncias do [Carbon](https://github.com/briannesbitt/Carbon), que oferece uma variedade de métodos de úteis, e extendem classe nativa PHP `DateTime`.  

Você pode customizar quais campos são automaticamente modificados, e até mesmo desativar completamente esta modificação, sobrescrevendo o método `getDates` do modelo:

	public function getDates()
	{
		return ['created_at'];
	}

Quando a coluna é considerada data, você pode definir o seu valor para um timestamp UNIX, string de dado (`Y-m-d`), date-time string, e é claro instância `DateTime` / `Carbon

Para desativar totalmente modificadores de data, simplemente retorne um array vazio do método `getDates`:

	public function getDates()
	{
		return [];
	}

<a name="attribute-casting"></a>
## Conversão De Atributos

Se você tiver alguns atributos que você queira sempre converter para outro tipo, você pode adicionar o atributo para a propriedade `casts` do seu modelo. Caso contrário, você terá que definir um mutator para cada um dos atributos, o que pode ser bem demorado. Aqui vai um exemplo de uso da propriedade `casts`:

	/**
	 * The attributes that should be casted to native types.
	 *
	 * @var array
	 */
	protected $casts = [
		'is_admin' => 'boolean',
	];

Agora o atributo `is_admin` irá sempre ser convertido para o tipo booleano quando você acessá-lo, mesmo se o valor de base é armazenado no banco de dados com inteiro. Outro tipo de conversão suportados são:  `integer`, `real`, `float`, `double`, `string`, `boolean`, `object` e `array`.

A conversão `array` é particularmente útil para trabalhar com colunas que são armazenadas como JSON serializados. Por exemplo, se seu banco de dados tem um campo do tipo TEXT que contém um JSON serializado, adicionando a conversão `array` para o atributo irá automaticamente desserializar o atributo para um array PHP quando você acessá-lo no seu modelo ELoquent:

	/**
	 * The attributes that should be casted to native types.
	 *
	 * @var array
	 */
	protected $casts = [
		'options' => 'array',
	];

Agora, quando você utilizar o modelo Eloquent:

	$user = User::find(1);

	// $options is an array...
	$options = $user->options;

	// options is automatically serialized back to JSON...
	$user->options = ['foo' => 'bar'];

<a name="model-events"></a>
## Eventos De Modelo

Modelos Eloquent começam vários eventos, permitindo que você acesse vários pontos no ciclo de vida do modelo usando os seguintes métodos: creating`, `created`, `updating`, `updated`, `saving`, `saved`, `deleting`, `deleted`, `restoring`, `restored`.


Sempre que um item novo é salvo paela primeira vez , os eventos `creating` e `created` serão iniciados. Se um item não é novo e o método `save` é chamado, o evento `updating` / `updated` será chamado. Em ambos dos casos, o evento `saving` / `saved` será chamado.

#### Cancelando Operações de Salvamento Por Meio Eventos 

Se é retornado `false` dos eventos `creating`, `updating`, `saving`, ou `deleting, a ação será canceladas:

	User::creating(function($user)
	{
		if ( ! $user->isValid()) return false;
	});

#### Onde Registrar Listeners (verificadores) de Eventos

Seu `EventServiceProvider` serve como um local conveniente para registrar o seu modelo de junção. Por exemplo

	/**
	 * Register any other events for your application.
	 *
	 * @param  \Illuminate\Contracts\Events\Dispatcher  $events
	 * @return void
	 */
	
	public function boot(DispatcherContract $events)
	{
		parent::boot($events);

		User::creating(function($user)
		{
			//
		});
	}

<a name="model-observers"></a>
## Observers de Modelos

Para consolidar a manipulação de eventos de modelos, você pode registrar um observer de modelo. Uma classe observer pode ter métodos que correspondem a vários eventos de modelos. Por exemplo, os métodos `creating`, `updating`, `saving` podem estar em um observer, adicionalmente a qualquer outro nome de evento de modelos. 

Então, por exemplo, um observer de modelo pode parecer com isto:

	class UserObserver {

		public function saving($model)
		{
			//
		}

		public function saved($model)
		{
			//
		}

	}

Você pode registrar uma instância do observer usand o método `observe`:

	User::observe(new UserObserver);

<a name="model-url-generation"></a>
## Geração de URL a partir do Modelo

Quando você passa um modelo para `route` ou métodos `action`, a sua chave primária é inserida na URI gerada. Por exemplo:

	Route::get('user/{user}', 'UserController@show');

	action('UserController@show', [$user]);

Neste exemlpo a propriedade `$user->id` será inserida entre chave assim como neste padrão `{user}` da URL gerada. No entando, se você gostar de usar outra propriedade ao invés do ID, você pode sobrescrever o método `getRouteKey` do seu modelo:

	public function getRouteKey()
	{
		return $this->slug;
	}

<a name="converting-to-arrays-or-json"></a>
## Convertendo para Arrays / JSON

#### Convertendo um Modelo Para um Array 

Quando estiver desenvolvendo APIS JSON, você pode frequentemente precisar converter os relacionamentos dos seus modelos para arrays ou JSON. Então, o Eloquent inclui métodos para fazer isto. Para converter um modelo e seu relacionamento em um array, você pode usar o método `toArray`:

	$user = User::with('roles')->first();

	return $user->toArray();
Note que todas as coleções dos modelos podem também ser convertidas em arrays:

	return User::all()->toArray();

#### Convertendo um Modelo para JSON 

Para converter um modelo para JSON, você pode usar o método `toJson`:

	return User::find(1)->toJson();

#### Retornando um Modelo A Partir De Uma Rota

Note que quando um modelo ou uma coleção é convertida para uma string, isto poderá ser convertido para JSON, o que significa que você pode retornar um objeto Eloquent diretamente das rotas da sua aplicação! 

	Route::get('users', function()
	{
		return User::all();
	});

#### Escondendo Atributos Da conversão de Array ou JSON

Algumas vezes você pode querer limitar os atributos que estão incluídos no array ou no JSON form do seu modelo, como nas senhas, Para fazê-lo, adicione a definição da propriedade `hidden` no seu modelo:

	class User extends Model {

		protected $hidden = ['password'];

	}

> **Nota:** Quando estiver escondendo relacionamento, use o nome **método** do relacionamento, não o nome dinâmico do acessor.

Alternativamente, você pode usar a propriedade `visible` para definir uma white-list(faz o inverso da black-list):

	protected $visible = ['first_name', 'last_name'];

<a name="array-appends"></a>

Ocasionalmente, você pode precisar adicionar atributos array que uma coluna correspondente no seu banco de dados. Para fazê-lo, simplesmente defina um accessor para o valor:

	public function getIsAdminAttribute()
	{
		return $this->attributes['admin'] == 'yes';
	}

Uma vez que você tenha criado o acessor, apenas adicione o valor para a propriedade `appends` no seu modelo:

	protected $appends = ['is_admin'];

Uma vez que o atributo tenha sido adicionado para a lista `appends`, isto será incluido em ambos os forms array e JSON do modelo. Atributos no array `appends` respeitam a configuração `visible` e `hidden` do modelo.
