# Laravel Livewire CRUD Application

A complete CRUD (Create, Read, Update, Delete) application built with Laravel and Livewire for managing blog posts.

## 🚀 Features

- **Full CRUD Operations**: Create, read, update, and delete posts
- **Real-time Updates**: Livewire provides dynamic, reactive interfaces
- **Form Validation**: Built-in validation with error handling
- **Responsive Design**: Bootstrap-based UI for mobile-friendly experience
- **Session Messages**: User-friendly success/error notifications

## 📋 Prerequisites

- PHP >= 8.1
- Composer
- MySQL/PostgreSQL/SQLite
- Node.js & NPM (for asset compilation)

## 🛠️ Installation

### Step 1: Create Laravel Project

```bash
composer create-project laravel/laravel livewire
cd livewire
```

### Step 2: Configure Environment

```bash
cp .env.example .env
php artisan key:generate
```

Update your `.env` file with database credentials:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=livewire_crud
DB_USERNAME=your_username
DB_PASSWORD=your_password
```

### Step 3: Create Database Migration

```bash
php artisan make:migration create_posts_table
```

**Migration File** (`database/migrations/xxxx_xx_xx_create_posts_table.php`):

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up()
    {
        Schema::create('posts', function (Blueprint $table) {
            $table->id();
            $table->string('title');
            $table->text('body');
            $table->timestamps();
        });
    }

    public function down()
    {
        Schema::dropIfExists('posts');
    }
};
```

Run the migration:

```bash
php artisan migrate
```

### Step 4: Create Post Model

```bash
php artisan make:model Post
```

**Model File** (`app/Models/Post.php`):

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
    use HasFactory;

    protected $fillable = [
        'title', 'body'
    ];
}
```

### Step 5: Install Livewire

```bash
composer require livewire/livewire
```

### Step 6: Create Livewire Component

```bash
php artisan make:livewire posts
```

This creates:
- `app/Http/Livewire/Posts.php`
- `resources/views/livewire/posts.blade.php`

## 📝 Component Implementation

### Livewire Component (`app/Http/Livewire/Posts.php`)

```php
<?php

namespace App\Http\Livewire;

use Livewire\Component;
use App\Models\Post;

class Posts extends Component
{
    public $posts, $title, $body, $post_id;
    public $updateMode = false;

    public function render()
    {
        $this->posts = Post::latest()->get();
        return view('livewire.posts');
    }

    private function resetInputFields()
    {
        $this->title = '';
        $this->body = '';
    }

    public function store()
    {
        $validatedData = $this->validate([
            'title' => 'required|min:3',
            'body' => 'required|min:10',
        ]);

        Post::create($validatedData);
        session()->flash('message', 'Post created successfully!');
        $this->resetInputFields();
    }

    public function edit($id)
    {
        $post = Post::findOrFail($id);
        $this->post_id = $id;
        $this->title = $post->title;
        $this->body = $post->body;
        $this->updateMode = true;
    }

    public function cancel()
    {
        $this->updateMode = false;
        $this->resetInputFields();
    }

    public function update()
    {
        $validatedData = $this->validate([
            'title' => 'required|min:3',
            'body' => 'required|min:10',
        ]);

        $post = Post::find($this->post_id);
        $post->update($validatedData);

        $this->updateMode = false;
        session()->flash('message', 'Post updated successfully!');
        $this->resetInputFields();
    }

    public function delete($id)
    {
        Post::find($id)->delete();
        session()->flash('message', 'Post deleted successfully!');
    }
}
```

## 🎨 View Templates

### Main Posts View (`resources/views/livewire/posts.blade.php`)

```blade
<div>
    @if (session()->has('message'))
        <div class="alert alert-success alert-dismissible fade show" role="alert">
            {{ session('message') }}
            <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
        </div>
    @endif

    @if($updateMode)
        @include('livewire.update')
    @else
        @include('livewire.create')
    @endif

    <div class="table-responsive">
        <table class="table table-striped table-hover">
            <thead class="table-dark">
                <tr>
                    <th>#</th>
                    <th>Title</th>
                    <th>Body</th>
                    <th width="200px">Actions</th>
                </tr>
            </thead>
            <tbody>
                @forelse($posts as $post)
                    <tr>
                        <td>{{ $post->id }}</td>
                        <td>{{ Str::limit($post->title, 50) }}</td>
                        <td>{{ Str::limit($post->body, 100) }}</td>
                        <td>
                            <button wire:click="edit({{ $post->id }})" 
                                    class="btn btn-primary btn-sm">
                                <i class="fas fa-edit"></i> Edit
                            </button>
                            <button wire:click="delete({{ $post->id }})" 
                                    class="btn btn-danger btn-sm"
                                    onclick="return confirm('Are you sure?')">
                                <i class="fas fa-trash"></i> Delete
                            </button>
                        </td>
                    </tr>
                @empty
                    <tr>
                        <td colspan="4" class="text-center">No posts found.</td>
                    </tr>
                @endforelse
            </tbody>
        </table>
    </div>
</div>
```

### Create Form (`resources/views/livewire/create.blade.php`)

```blade
<form wire:submit.prevent="store">
    <div class="mb-3">
        <label for="title" class="form-label">Title</label>
        <input type="text" 
               class="form-control @error('title') is-invalid @enderror" 
               id="title" 
               wire:model="title" 
               placeholder="Enter post title">
        @error('title') 
            <div class="invalid-feedback">{{ $message }}</div>
        @enderror
    </div>

    <div class="mb-3">
        <label for="body" class="form-label">Body</label>
        <textarea class="form-control @error('body') is-invalid @enderror" 
                  id="body" 
                  wire:model="body" 
                  rows="4" 
                  placeholder="Enter post content"></textarea>
        @error('body') 
            <div class="invalid-feedback">{{ $message }}</div>
        @enderror
    </div>

    <button type="submit" class="btn btn-success">
        <i class="fas fa-save"></i> Save Post
    </button>
</form>
```

### Update Form (`resources/views/livewire/update.blade.php`)

```blade
<form wire:submit.prevent="update">
    <input type="hidden" wire:model="post_id">
    
    <div class="mb-3">
        <label for="title" class="form-label">Title</label>
        <input type="text" 
               class="form-control @error('title') is-invalid @enderror" 
               id="title" 
               wire:model="title" 
               placeholder="Enter post title">
        @error('title') 
            <div class="invalid-feedback">{{ $message }}</div>
        @enderror
    </div>

    <div class="mb-3">
        <label for="body" class="form-label">Body</label>
        <textarea class="form-control @error('body') is-invalid @enderror" 
                  id="body" 
                  wire:model="body" 
                  rows="4" 
                  placeholder="Enter post content"></textarea>
        @error('body') 
            <div class="invalid-feedback">{{ $message }}</div>
        @enderror
    </div>

    <button type="submit" class="btn btn-primary">
        <i class="fas fa-edit"></i> Update Post
    </button>
    <button type="button" wire:click="cancel" class="btn btn-secondary">
        <i class="fas fa-times"></i> Cancel
    </button>
</form>
```

## 🎯 Main Application View

### Welcome Page (`resources/views/welcome.blade.php`)

```blade
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Laravel Livewire CRUD</title>
    
    <!-- Bootstrap CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <!-- Font Awesome -->
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    
    @livewireStyles
</head>
<body>
    <div class="container py-4">
        <div class="row justify-content-center">
            <div class="col-md-10">
                <div class="card shadow">
                    <div class="card-header bg-primary text-white">
                        <h2 class="mb-0">
                            <i class="fas fa-blog"></i> Laravel Livewire CRUD
                        </h2>
                    </div>
                    <div class="card-body">
                        @livewire('posts')
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- Bootstrap JS -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
    @livewireScripts
</body>
</html>
```

## 🚀 Running the Application

```bash
php artisan serve
```

Visit: `http://localhost:8000`

## ⚡ Performance Optimizations

### 1. Database Optimizations

```php
// Use pagination for large datasets
public function render()
{
    $this->posts = Post::latest()->paginate(10);
    return view('livewire.posts');
}

// Add database indexes
Schema::create('posts', function (Blueprint $table) {
    $table->id();
    $table->string('title')->index();
    $table->text('body');
    $table->timestamps();
});
```

### 2. Livewire Optimizations

```php
// Use wire:loading for better UX
<button wire:click="store" wire:loading.attr="disabled">
    <span wire:loading.remove>Save</span>
    <span wire:loading>Saving...</span>
</button>

// Debounce input for better performance
<input wire:model.debounce.300ms="title" type="text">
```

### 3. Caching Strategies

```php
// Cache frequently accessed data
public function render()
{
    $this->posts = Cache::remember('posts', 300, function () {
        return Post::latest()->get();
    });
    return view('livewire.posts');
}
```

## 🔧 Additional Features

### Search Functionality

```php
public $search = '';

public function render()
{
    $this->posts = Post::where('title', 'like', '%' . $this->search . '%')
                       ->orWhere('body', 'like', '%' . $this->search . '%')
                       ->latest()
                       ->get();
    return view('livewire.posts');
}
```

### Bulk Operations

```php
public $selectedPosts = [];

public function deleteSelected()
{
    Post::whereIn('id', $this->selectedPosts)->delete();
    $this->selectedPosts = [];
    session()->flash('message', 'Selected posts deleted successfully!');
}
```

## 📚 Learning Resources

- [Laravel Documentation](https://laravel.com/docs)
- [Livewire Documentation](https://laravel-livewire.com/docs)
- [Bootstrap Documentation](https://getbootstrap.com/docs)

## 🤝 Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This project is open-sourced software licensed under the [MIT license](https://opensource.org/licenses/MIT).

---

**Built with ❤️ using Laravel & Livewire**
