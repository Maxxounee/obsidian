
### Вытащить массив title и заJOINить их

```php
$post->tags->pluck('title')->implode(', ')
```

### sync для связных таблиц при обновлении

```php
public function update(Post $post) {
        $data = request()->validate([
            'title' => 'string',
            'content' => 'string',
            'image' => 'string',
            'category_id' => '',
            'tags' => 'array',
        ]);

        $tags = $data['tags'];
        unset($data['tags']);

        $post->update($data);
        $post->tags()->sync($tags);

        return redirect()->route('posts.show', $post->id);
    }
```


### attach для связных таблиц при создании

```php
   public static function store() {
        $data = request()->validate([
            'title' => 'string',
            'content' => 'string',
            'image' => 'string',
            'category_id' => 'integer',
            'tags' => 'array',
        ]);

        $tags = $data['tags'];
        unset($data['tags']);

        $post = Post::create($data);
        $post->tags()->attach($tags);

        return redirect()->route('posts.index');
    }
```