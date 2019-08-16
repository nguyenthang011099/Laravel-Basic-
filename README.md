# Laravel 


## [1. Giới thiệu và cài đặt Laravel](https://laravel.com/docs/5.8/installation)
`composer create-project --prefer-dist laravel/laravel TenProject "5.*"`



## [2. Cấu trúc thư mục](https://laracasts.com/series/laravel-from-scratch-2018/episodes/9)

### Thư mục App
- Chứa `Controller`, `Model` và các thành phần cốt lõi khác của ứng dụng.
### Thư mục Config
- Chứa các file cấu hình cho hệ thống.
### Thư mục Database
- Giúp ta tạo CSDL.
- Có ba thành phần gồm `migrate`, `seed`, `factory`.
### Thư mục Resources
- Chứa view, các file tài nguyên chưa compile(SASS, Javascript), các file ngôn ngữ.
### Thư mục Routes
- Chứa các định nghĩa `Route`.
- Gồm các file: `web.php`, `api.php`, `console.php`, `channels.php`(từ 5.8).
### Thư mục Public
- Chứa file `index.php` là điểm đầu tiên cho tất cả các request đến ứng dụng của bạn và cấu hình autoloading. Thư mục còn chứa các tài nguyên như image, css, js.
### Thư mục Tests
- Chứa các file tests.
### file .env 
- Cài đặt liên kết tới DB cho hệ thống.

## 3. Luồng ứng dụng cơ bản

![](./image/luong-ung-dung.jpg)

## [4. Eloquent - Model](https://laracasts.com/series/laravel-from-scratch-2018/episodes/8)
- Model là một lớp dữ liệu có cấu trúc giống với bảng trong CSDL, dùng để tương tác với dữ liệu trong bảng.

### Tạo Model 

```
php artisan make:model TenModel
```
### Một số thuộc tính

```php
protected $table = 'table';
protected $primaryKey = 'primaryKey';
public $timestamps = false;
...
```

### Một số phương thức

```php
App\Model::all();
App\Model::find('primaryKey');
App\Model::destroy('primaryKey');
```

- Kết hợp với query builder để tạo câu truy vấn phù hợp.
![](./image/querybuilder.png)

### Liên kết tới các model khác
```php
namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Get the phone record associated with the user.
     */
    public function phone()
    {
        return $this->hasOne('App\Phone'); // ('Model', 'foreign_key', 'local_key')
    }
}
```

```php
$phone = User::find(1)->phone;
```

- One To One
Sử dụng ở model cha
```php
$this->hasOne()
```
Sử dụng ở model con
```php
$this->belongsTo()
```

- One To Many
```php
$this->hasMany()
```
```php
$this->belongsTo()
```

- Has One Through
```
users
    id - integer
    supplier_id - integer

suppliers
    id - integer

history
    id - integer
    user_id - integer
```

```php
class Supplier extends Model
{
    /**
     * Get the user's history.
     */
    public function userHistory()
    {
        return $this->hasOneThrough(
            'App\History',
            'App\User',
            'supplier_id', // Foreign key on users table...
            'user_id', // Foreign key on history table...
            'id', // Local key on suppliers table...
            'id' // Local key on users table...
        );
    }
}
```

- Has Many Through
```
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
```

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Country extends Model
{
    /**
     * Get all of the posts for the country.
     */
    public function posts()
    {
        return $this->hasManyThrough('App\Post', 'App\User');
    }
}
```

## [5. Service Container](https://laracasts.com/series/laravel-from-scratch-2018/episodes/21)

### Dependency Injection
- Dependency Injection có nghĩa là khi khởi tạo một object của class, chúng ta không cần phải khởi tạo những class liên quan (dependencies class) bên trong class đó, mà chúng sẽ được inject từ bên ngoài. Và thường gặp nhất là dạng Constructor injection , những class sẽ được inject khi khởi tạo object của class.

```php
<?php
class ReservationMailer{
    protected $mailer;
    public function __construct(Mailer $mailer){
      $this->mailer = $mailer;
    }

    public function sendReservationMail($user){
      return $this->mailer->mail($user->email, 'reservation.reserveComplete');
    }
}
?>
```

### Service Container
- Có một số vấn đề mà DI gặp phải: khi có nhiều dependencies class thì phải làm sao? Làm sao để giải quyết việc inject nối tiếp?
- Trong laravel hỗ trợ chúng ta giải quyết việc này thông qua Service Container.
- Vậy ta có thể hiểu nôm na Service Container như là tấm bản đò, một dịch vụ tổng đài để quản lý class dependency và thực hiện inject.

### Global Helper app()
```php
<?
class Alpha{
  public function __construct(){

  }
}

class Beta{
  public function __construct(){

  }
}

class Gamma{
  public function __construct(Alpha $alpha, Beta $beta){

  }
}

$gm = app(Gamma::class);
```

### Bind và Resolve
- `bind` để chỉ việc đăng ký một class hay interface với Container, và `resolve` để lấy ra instance từ Container.

```php
// Binding
\App::bind('computer', function() {
    $keyboard = new Keyboard();
    $monitor = new Monitor();
    $computer = new Computer($monitor, $keyboard);
    return $computer;
}

// Resolving.
\App::make('computer');
app('computer');
app()->make('computer');
app()['computer']
```

### Singleton binding, Instance binding, Interface binding
- **Singleton binding**: instance sẽ chỉ được resolve một lần, những lần gọi tiếp theo sẽ không tạo ra instance mới mà chỉ trả về instance đã được resolve từ trước

```php
app()->singleton('now', function() {return time();});
app('now') === app('now'); // true
```

- **Instance binding**: Cũng gần giống với Singleton Binding.
```php
$now = time();
app()->instance('now', $now);
$now === app('now'); // true
```

- **Interface binding**: 

## [6. Service providers](https://laracasts.com/series/laravel-from-scratch-2018/episodes/22)
- Service provider nói với Laravel `bind` các thành phần khác nhau vào `Service container` của laravel. Ta có thể sử dụng lệnh `php artisan make: provider ClientsServiceProvider` trên command để tự tạo ra một service provider. Nó sẽ cung cấp cho chúng ta 2 function là: `register()` và `boot()`.

## [7. Facade](https://laravel.com/docs/5.8/facades)
- Facade giup ta truy cập đến các hàm bên trong các service được khai báo trong `Service Container` bằng cách gọi các hàm static.
- Nó là những thứ chúng ta hay sử dụng như:
```php
use Illuminate\Support\Facades\Cache;

Route::get('/cache', function () {
    return Cache::get('key');
});
```
```
- Auth, 
- Route, 
- DB, 
- Session,
- View,
...
```

## [8. Events](https://laravel.com/docs/5.8/events)
- Trong đời thường cũng như trong quá trình hoạt động của một ứng dụng có rất nhiều event xảy ra. Ví dụ như trong ứng dụng web của chúng ta khi người dùng click lên một button là một sự kiện, khi người dùng thêm sản phẩm vào giỏ hàng là một sự kiện..v.v.. Đôi khi chúng ta cần xử lý các sự kiện này, và định nghĩa các tác vụ mà ứng dụng của chúng ta cần phản hồi lại khi một sự kiện xác định xảy ra. Để giúp chúng ta làm được việc này laravel cung cấp cho chúng ta cái gọi là Event.
- Laravel cung cấp cho chúng ta một cách thuận tiện để bạn có thể `đăng ký event` và `listener` cho event của bạn, bằng cách truy cập vào `app\Providers\EventServiceProvider`
    + Đăng ký event và listener
    + Sinh class event và listener:  `php artisan event:generate`
    + Định nghĩa sự kiện(trong thư mục `Events`)
    + Định nghĩa Listener(trong thư mục `Listener`), Queued Event Listeners, Handling Failed Jobs
    + Bắt sự kiện trong controller với `event()`

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTEyMTIyNTMxNSwtODQ4MDY4NjYxXX0=
-->
