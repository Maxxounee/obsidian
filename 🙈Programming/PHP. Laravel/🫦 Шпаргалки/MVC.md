## 0. Роутер -> Контроллер -> Модель -> View

## 1. Настраиваем роутинг

[Конвенция CRUD Laravel](https://laravel.com/framework/docs/controllers)

|Verb|URI|Action|Route Name|
|---|---|---|---|
|GET|`/photos`|index|photos.index|
|GET|`/photos/create`|create|photos.create|
|POST|`/photos`|store|photos.store|
|GET|`/photos/{photo}`|show|photos.show|
|GET|`/photos/{photo}/edit`|edit|photos.edit|
|PUT/PATCH|`/photos/{photo}`|update|photos.update|
|DELETE|`/photos/{photo}`|destroy|photos.destroy|

```php
# routes/web.php
use App\Http\Controllers\PostController;

ob_start();

Route::get('/posts', [PostController::class, 'index'])->name('posts.index');
Route::get('/posts/create', [PostController::class, 'create'])->name('posts.create');
Route::post('/posts', [PostController::class, 'store'])->name('posts.store');
Route::get('/posts/{post}', [PostController::class, 'show'])->name('posts.show');
Route::get('/posts/{post}/edit', [PostController::class, 'edit'])->name('posts.edit');
Route::patch('/posts/{post}', [PostController::class, 'update'])->name('posts.update');
Route::delete('/posts/{post}', [PostController::class, 'destroy'])->name('posts.destroy');
```

## 2. Настраиваем контроллер

```php

class PostController extends Controller {
    public static function index() {
        $posts = Post::all();
        //        dump($posts);

        return view(
            'posts.index',
            compact('posts')
        );
    }

    public static function create() {
        return view(
            'posts.create',
        );
    }

    public static function store() {
        $data = request()->validate([
            'title' => 'string',
            'post_content' => 'string',
            'image' => 'string',
        ]);

        Post::create($data);

        return redirect()->route('posts.index');
    }

    public function show($id) {
        $post = Post::findOrFail($id);

        return view(
            'posts.show',
            compact(['post', 'id'])
        );
    }

    /* Второй вариант обработки id.
    Laravel сам применит метод Post::findOrFail($id) */
    public function show_another(Post $post) {
        return view(
            'posts.show',
            compact(['post'])
        );
    }

    public function edit($id) {
        $post = Post::findOrFail($id);

        return view(
            'posts.edit',
            compact(['post', 'id'])
        );
    }

    public function update(Post $post) {
        $data = request()->validate([
            'title' => 'string',
            'post_content' => 'string',
            'image' => 'string',
        ]);

        $post->update($data);

        return redirect()->route('posts.show', $post->id);
    }

    public function destroy(Post $post) {
        $post->delete();

        return redirect()->route('posts.index');
    }
}
```

## 3. Настраиваем модель

```php
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Post extends Model {
    use SoftDeletes;

    // Laravel сам определяет название таблицы исходя из названия файла.
    /* Но хорошим тоном считается явное указание переменной $table */
    protected $table = 'posts';
    // В Model->GuardsAttributes laravel прописывает защиту от записи
    /* для всех атрибутов (protected $guarded = ['*'];)
    Переопределяем, чтобы снять защиту */
    protected $guarded = ['id'];
    // Другой вариант:
    // protected $guarded = false;
    // Противоположность $guarded. Явно указываем поля, разрешенные
    // к изменению
    protected $fillable = [];
}
```

