![Boas Práticas Laravel](/images/logo.png?raw=true)
#

O que é descrito aqui é uma compiolação entre o manual de [Boas Práticas Laravel](https://github.com/jonaselan/laravel-best-practices) com a organização padrão dos projetos da Escola Recursos!



## Conteúdo 

[Models gordos, controllers finos](#models-gordos-controllers-finos)

[Siga a convenção de nomes usada no Laravel](#siga-a-convenção-de-nomes-usada-no-laravel)

[Organização das pastas](#Organização-das-pastas)

[Regras de ouro](#Regras-de-ouro)


#

### **Models gordos, controllers finos**
Coloque toda a lógica relacionada a banco em modelos Eloquent ou em repositórios caso você esteja usando Query Builder ou consultas SQL.

Ruim:

```php
public function index()
{
    $clients = Client::verified()
        ->with(['orders' => function ($q) {
            $q->where('created_at', '>', Carbon::today()->subWeek());
        }])
        ->get();

    return view('index', ['clients' => $clients]);
}
```

Bom:

```php
public function index()
{
    return view('index', ['clients' => $this->client->getWithNewOrders()]);
}

class Client extends Model
{
    public function getWithNewOrders()
    {
        return $this->verified()
            ->with(['orders' => function ($q) {
                $q->where('created_at', '>', Carbon::today()->subWeek());
            }])
            ->get();
    }
}
```

[🔝 Voltar para o início](#conteúdo)


### **Siga a convenção de nomes usada no Laravel**

 Siga [PSR standards](http://www.php-fig.org/psr/psr-2/).

 Siga também a convenção de nomes aceita pelo a comunidade Laravel:

O que | Como | Bom | Ruim
------------ | ------------- | ------------- | -------------
Controller | plural | PostsController | ~~PostController~~
Route | plural | articles/6e2ebafa-3f81-48a3-8e28-b5a3f557b2c1 | ~~article/1~~
Named route | snake_case with dot notation | users.show_active | ~~users.show-active, show-active-users~~
Model | singular | User | ~~Users~~
hasOne or belongsTo relationship | singular | articleComment | ~~articleComments, article_comment~~
All other relationships | plural | articleComments | ~~articleComment, article_comments~~
Table | plural | article_comments | ~~article_comment, articleComments~~
Pivot table | singular model names in alphabetical order | article_user | ~~user_article, articles_users~~
Colunas em tabelas | snake_case without model name | meta_title | ~~MetaTitle; article_meta_title~~
Model property | snake_case | $model->created_at | ~~$model->createdAt~~
Foreign key | singular model name with _id suffix | article_id | ~~ArticleId, id_article, articles_id~~
Chaves primárias | - | id | ~~custom_id~~
Migrações | - | 2017_01_01_000000_create_articles_table | ~~2017_01_01_000000_articles~~
Métodos | camelCase | getAll | ~~get_all~~
Métodos em controllers | [table](https://laravel.com/docs/master/controllers#resource-controllers) | store | ~~saveArticle~~
Métodos e classes de teste | camelCase | testGuestCannotSeeArticle | ~~test_guest_cannot_see_article~~
Variáveis | camelCase | $articlesWithAuthor | ~~$articles_with_author~~
Collection | descriptive, plural | $activeUsers = User::active()->get() | ~~$active, $data~~
Object | descriptive, singular | $activeUser = User::active()->first() | ~~$users, $obj~~
Config e arquivos de linguagem | snake_case | articles_enabled | ~~ArticlesEnabled; articles-enabled~~
View | kebab-case | show-filtered.blade.php | ~~showFiltered.blade.php, show_filtered.blade.php~~
Config | snake_case | google_calendar.php | ~~googleCalendar.php, google-calendar.php~~
Contract (interface) | adjective or noun | Authenticatable | ~~AuthenticationInterface, IAuthentication~~
Trait | adjective | Notifiable | ~~NotificationTrait~~

[🔝 Voltar para o início](#conteúdo)


### **Organização das pastas**



Coloque toda a lógica relacionada a banco em modelos Eloquent ou em repositórios caso você esteja usando Query Builder ou consultas SQL.

Os models ficam na seguinte pasta:
```php
'app/Model/[Group-Folder]/[Model.php]';
```

Os controllers seguem a mesma lógica:
```php
'app/Http/Controllers/[Group-Folder]/[Controller.php]';
```

As vies seguem a mesma lógica:
```php
'resources/views/[Group-Folder]/[Model-name]/[your-view.blade.php]';
```


[🔝 Voltar para o início](#conteúdo)



# **Regras de ouro**

1 - Obedeça o padrão Rest nos controllers.

Ruim:
```php
class CursoController {
    public function nomeMetodoCustomizado()
    {    
        //...
    }
}
```

Bom:
```php
class CursoController extends Controller {
    public function index()
    {    
        //...
    }
    public function show()
    {    
        //...
    }
    public function update()
    {    
        //...
    }
    public function create()
    {    
        //...
    }
    public function edit()
    {    
        //...
    }
    public function store()
    {    
        //...
    }
}
```

2 - Rotas limpas

Ruim:
```php
//...
Route::get('/posts', 'Feed\PostsController@index')->name('home');
Route::get('/posts/{id}', 'Feed\PostsController@show')->name('show');
Route::post('/posts', 'Feed\PostsController@store')->name('show');
//...
```

Bom:
```php
Route::resource('posts', 'Feed\PostsController');
```

3 - Usamos 'UUID' em todos os models

```php
//...
use BinaryCabin\LaravelUUID\Traits\HasUUID;
//...

class Post extends Model
{
    //...
    use HasUUID;
    //...
}
//...
```

4 - Usamos 'SoftDeletes ' em todos os models

```php
//...
use Illuminate\Database\Eloquent\SoftDeletes;
//...

class Post extends Model
{
    //...
    use SoftDeletes;
    //...
}
//...
```


4 - Nunca passar ID, manter sempre o UUID

```php
class PostsController extends Controller
{
    public function show($uuid) {
        $post = Post::whereUuid('uuid', $uuid)->first();
        return response()->json($post);
    }
}
```