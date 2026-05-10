# 📸 Instagram Clone

A full-stack social media web application built with **Laravel 6** and **Vue.js** — replicating core Instagram features including photo posting, user following, profile management, and automated email notifications.

> This project demonstrates practical understanding of **real-world social media architecture**: many-to-many relationships, authorization policies, caching, image processing, and reactive frontend components.

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| Backend Framework | Laravel 6.2 |
| Language | PHP 7+ |
| Frontend | Blade Templates + **Vue.js** |
| Reactive HTTP | **Axios** (AJAX follow/unfollow) |
| Database | MySQL |
| Image Processing | **Intervention/Image** |
| Authentication | Laravel UI (built-in Auth) |
| Authorization | **Laravel Policies** |
| Caching | **Laravel Cache** |
| Email | **Laravel Mailable** (Markdown) |
| Debugging | **Laravel Telescope** |

---

## ✨ Key Features

### 📰 News Feed
- Shows posts **only from followed users** — not all users
- Paginated display (5 posts per page) using `paginate(5)`
- Sorted by latest post first

### 📷 Photo Posts
- Upload photo with caption
- Automatic image resizing to **1200×1200px** using Intervention/Image
- Images stored securely in Laravel public storage

### 👤 User Profiles
- Profile auto-created on user registration (via **Model Boot**)
- Edit profile: title, bio, website URL, and profile photo
- Profile photo auto-resized to **1000×1000px**
- Follower count / Following count / Post count displayed

### 👥 Follow System (Many-to-Many)
- Follow and Unfollow users
- **Toggle follow** with a single database operation
- Reactive **Follow/Unfollow button** built with Vue.js + Axios (no page reload)
- Pivot table `profile_user` manages the many-to-many relationship

### 📧 Welcome Email
- Automated **welcome email** sent to every new registered user
- Built with **Laravel Mailable** using Markdown email templates
- Triggered automatically via **Model Boot Event** on user creation

### 🔐 Authorization (Laravel Policies)
- `ProfilePolicy` ensures **only the profile owner** can edit their profile
- Other users attempting to edit are blocked at the controller level
- `$this->authorize('update', $user->profile)` — clean, policy-based security

### ⚡ Performance Caching
- Post count, follower count, and following count are **cached for 30 seconds**
- Uses `Cache::remember()` to avoid repeated database queries on every profile visit
- Cache auto-refreshes when the 30-second window expires

---

## 🏗️ Architecture Highlights

### Many-to-Many Follow Relationship

```
users ────────────────── profile_user (pivot) ────────────── profiles
  id                          user_id                            id
  name                        profile_id                         user_id
  email                                                          title
  username                                                       image
```

```php
// User follows a Profile (many-to-many)
public function following()
{
    return $this->belongsToMany(Profile::class);
}

// Toggle follow/unfollow in one line
auth()->user()->following()->toggle($user->profile);
```

### Auto Profile Creation on Registration

```php
// User Model — boot() fires automatically when a new user is created
protected static function boot()
{
    parent::boot();

    static::created(function ($user) {
        // 1. Auto-create profile
        $user->profile()->create(['title' => $user->username]);

        // 2. Auto-send welcome email
        Mail::to($user->email)->send(new NewUserWelcomeMail());
    });
}
```

### Vue.js Follow Button (Reactive — No Page Reload)

```vue
<template>
    <button @click="followUser" v-text="buttonText"></button>
</template>

<script>
methods: {
    followUser() {
        axios.post('/follow/' + this.userId)
            .then(() => { this.status = !this.status; });
    }
},
computed: {
    buttonText() {
        return this.status ? 'Unfollow' : 'Follow';
    }
}
</script>
```

### Profile Stats with Caching

```php
// Cached for 30 seconds — avoids DB query on every page load
$postCount = Cache::remember('count.posts.' . $user->id, now()->addSeconds(30), function () use ($user) {
    return $user->posts->count();
});
```

### News Feed — Only from Followed Users

```php
// Fetch user IDs that the current user follows
$users = auth()->user()->following()->pluck('profiles.user_id');

// Get only posts from those users, paginated
$posts = Post::whereIn('user_id', $users)->with('user')->latest()->paginate(5);
```

---

## 🗄️ Database Schema

| Table | Description |
|---|---|
| `users` | User accounts (name, email, username, password) |
| `profiles` | User profiles (title, bio, URL, image) |
| `posts` | Photo posts (caption, image path, user_id) |
| `profile_user` | Pivot table — follow relationships (many-to-many) |
| `password_resets` | Password reset tokens |

---

## 📦 Laravel Packages Used

| Package | Purpose |
|---|---|
| `laravel/framework ^6.2` | Core framework |
| `intervention/image` | Resize and crop uploaded images |
| `laravel/telescope` | Debug panel for requests, queries, and mail |
| `laravel/ui` | Authentication scaffolding |
| `fideloper/proxy` | Trusted proxy support |

---

## ⚙️ Installation

### Requirements
- PHP >= 7.2
- Composer
- MySQL
- Node.js & NPM

### Steps

**1. Clone the repository**
```bash
git clone https://github.com/KoKoAungStar/InstagramClone.git
cd InstagramClone
```

**2. Install dependencies**
```bash
composer install
npm install && npm run dev
```

**3. Set up environment**
```bash
cp .env.example .env
php artisan key:generate
```

**4. Configure database in `.env`**
```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=instagram_clone
DB_USERNAME=root
DB_PASSWORD=your_password
```

**5. Configure email (for welcome mail)**
```env
MAIL_DRIVER=smtp
MAIL_HOST=smtp.mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=your_mailtrap_username
MAIL_PASSWORD=your_mailtrap_password
```

**6. Run migrations and storage link**
```bash
php artisan migrate
php artisan storage:link
```

**7. Start the server**
```bash
php artisan serve
```

Visit `http://localhost:8000`

---

## 📁 Project Structure

```
app/
├── Http/Controllers/
│   ├── PostsController.php       # Feed, create, store, show posts
│   ├── ProfilesController.php    # View, edit, update profile + caching
│   └── FollowsController.php     # Toggle follow/unfollow
├── Models/
│   ├── User.php                  # hasOne Profile, hasMany Posts, boot()
│   ├── Profile.php               # belongsToMany Users (followers)
│   └── Post.php                  # belongsTo User
├── Mail/
│   └── NewUserWelcomeMail.php    # Welcome email Mailable
└── Policies/
    └── ProfilePolicy.php         # Auth: only owner can edit profile

resources/
└── js/components/
    └── FollowButton.vue          # Vue.js reactive follow button

database/migrations/
├── create_users_table
├── create_profiles_table
├── create_posts_table
└── creates_profile_user_pivot_table   ← many-to-many follow system
```

---

## 🔐 Security Features

| Feature | Implementation |
|---|---|
| Authentication required | `middleware('auth')` on all protected routes |
| Profile edit protection | `ProfilePolicy` — only owner can edit |
| CSRF protection | Laravel built-in CSRF token on all forms |
| Password hashing | Laravel `bcrypt` via default Auth |
| Unauthorized follow redirect | Axios catches 401 → redirects to `/login` |

---

## 💡 Laravel Concepts Demonstrated

This project covers the following Laravel concepts that are directly applicable to enterprise web development in Japan:

- **Eloquent ORM** — hasOne, hasMany, belongsTo, belongsToMany
- **Many-to-Many Pivot Table** — follow system
- **Model Boot Events** — auto-create profile + send email on registration
- **Laravel Policies** — role-based authorization
- **Cache::remember** — performance optimization
- **Laravel Mailable** — automated email with Markdown template
- **Intervention/Image** — server-side image processing
- **Vue.js + Axios** — reactive component without full page reload
- **Route Model Binding** — clean controller injection
- **Pagination** — `paginate(5)` on feed

---

## 👨‍💻 Developer

**Ko Ko Aung**
- GitHub: [KoKoAungStar](https://github.com/KoKoAungStar)
- LinkedIn: [ko-ko-aung-dev](https://linkedin.com/in/ko-ko-aung-dev)

---

## 📄 License

This project is open source and available under the [MIT License](LICENSE).
