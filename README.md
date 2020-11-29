# Laravel API with Passport Auth Example

Just an example project to create an API with Laravel and Passport Authentication.

## Prerequisites

You need a working environment with:
* [Git](https://git-scm.com) - You can install it from https://git-scm.com/downloads.
* [MySQL](https://www.mysql.com) - You can install it from https://www.mysql.com/downloads/.
* [MySQL Workbench](https://www.mysql.com/products/workbench/) - You can install it from https://dev.mysql.com/downloads/workbench/.
* [Visual Studio Code](https://code.visualstudio.com/) - The Editor used in this project
* [Composer](https://getcomposer.org) - Install it from https://getcomposer.org/download/

## Getting Started (Just if you want to try this project)

Clone this project:

```
$ git clone https://github.com/tcrurav/laravel8APIAuthPassport.git
```

Open the created folder with your favourite editor/IDE. I have used Visual Studio Code for this project.

Make a copy of the file .env.example and name it as .env. 

After that edit the file with the data necessary to connect to your MySQL database:

```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=db_laravel_products
DB_USERNAME=YOUR_USER
DB_PASSWORD=YOUR_PASSWORD
```

Create the database db_laravel_products. I use MySQL Worbench.

To use mysql native authentication run the following script in MySQL Workbench (my case):

```
CREATE USER 'YOUR_USER'@'localhost' IDENTIFIED WITH mysql_native_password BY 'YOUR_PASSWORD';
GRANT ALL PRIVILEGES ON *.* TO 'YOUR_USER'@'localhost' WITH GRANT OPTION;

CREATE USER 'YOUR_USER'@'%' IDENTIFIED WITH mysql_native_password BY 'YOUR_PASSWORD';
GRANT ALL PRIVILEGES ON *.* TO 'YOUR_USER'@'%' WITH GRANT OPTION;
```

Now go to the directory which contains the cloned project:

```
cd laravel-products
```

Install all project packages:

```
composer install
```

Run the migrations to create all tables (check with MySQL Workbench that all tables were created):

```
php artisan migrate
```

Run the project:

```
php artisan serve
```

Use Postman to test this project as done in the following section.

## Testing the API

Use Postman to test the API.

### To register a user:

![alt text](https://github.com/tcrurav/laravel8APIAuthPassport/blob/master/screenshots/screenshot-1.png)

### To login with a user:

![alt text](https://github.com/tcrurav/laravel8APIAuthPassport/blob/master/screenshots/screenshot-2.png)

### To use the token to get all products: (At the beginning there are no products in DB)

It's important to notice that here and in the rest of steps I'm using the token recieved when the user logs in.

![alt text](https://github.com/tcrurav/laravel8APIAuthPassport/blob/master/screenshots/screenshot-3.png)

### To create a product:

![alt text](https://github.com/tcrurav/laravel8APIAuthPassport/blob/master/screenshots/screenshot-4.png)

### To use the token to get all products again: (now there's a product in DB)

![alt text](https://github.com/tcrurav/laravel8APIAuthPassport/blob/master/screenshots/screenshot-5.png)

### To use the token to update a product:

![alt text](https://github.com/tcrurav/laravel8APIAuthPassport/blob/master/screenshots/screenshot-6.png)

### To use the token to delete a product:

![alt text](https://github.com/tcrurav/laravel8APIAuthPassport/blob/master/screenshots/screenshot-7.png)


## Getting Started (If you want to create this project from the beginning)

You need an environment with all the items in the Prerequisites section of this document:
* Git.
* MySQL
* MySQL Workbench
* Visual Studio Code
* Composer

Install globally laravel:

```
composer global require laravel/installer
```

Create a new laravel project:

```
laravel new laravel-products
```

Now go to the directory which contains the created project:

```
cd laravel-products
```

Make a copy of the file .env.example and name it file .env. After that edit the file with the data to connect to your MySQL database:

```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=db_laravel_products
DB_USERNAME=YOUR_USER
DB_PASSWORD=YOUR_PASSWORD
```

Create the database db_laravel_products. I use MySQL Worbench.

To use mysql native authentication run the following script in MySQL Workbench (my case):

```
CREATE USER 'YOUR_USER'@'localhost' IDENTIFIED WITH mysql_native_password BY 'YOUR_PASSWORD';
GRANT ALL PRIVILEGES ON *.* TO 'YOUR_USER'@'localhost' WITH GRANT OPTION;

CREATE USER 'YOUR_USER'@'%' IDENTIFIED WITH mysql_native_password BY 'YOUR_PASSWORD';
GRANT ALL PRIVILEGES ON *.* TO 'YOUR_USER'@'%' WITH GRANT OPTION;
```

Install Passport package:

```
composer require laravel/passport
```

Open config/app.php and put the following code in the 'providers' array:

```
'providers' =>[
    Laravel\Passport\PassportServiceProvider::class,

    // There may be other providers here already
 ],
```

Make the migration:

```
php artisan migrate
```

Now, you need to install laravel 8 to generate passport encryption keys. This command will create the encryption keys needed to generate secure access tokens (Maybe you need the option --force):

```
php artisan passport:install
```

Navigate to App/Models directory and open User.php file. Then update the following code into User.php, so that 'HasApiTokens' is used instead of 'HasFactory':

```
use Laravel\Passport\HasApiTokens;
 
class User extends Authenticatable
{
    use Notifiable, HasApiTokens;
```

Next, Navigate to config/auth.php and open the auth.php file. Then Change the API driver. Put this code ‘driver’ => ‘passport’, in API:

```
'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],
        'api' => [ 
            'driver' => 'passport', 
            'provider' => 'users', 
        ],
    ],
```

open the terminal and run the following command to create a product model and migration file:

```
php artisan make:model Product -m
```

After that, navigate to the database/migrations directory and open the create_products_table.php file. Then update the following code into it:

```
Schema::create('products', function (Blueprint $table) {
            $table->bigIncrements('id');
            $table->string('name');
            $table->text('detail');
            $table->timestamps();
        });
```

Add a fillable property in Product.php file, so navigate to app/Models directory and open Product.php file and update the following code into it:

```
class Product extends Model
{
    protected $fillable = [
        'name', 'detail'
    ];
}
```

you need to do a migration using the below command. This command creates tables in the database:

```
php artisan migrate
```

In this step, create rest API auth and crud operation laravel 8 routes.

So, navigate to the routes directory and open api.php. Then update the following routes into the api.php file:

```
use App\Http\Controllers\API\PassportAuthController;
use App\Http\Controllers\API\ProductController;
 
Route::post('register', [PassportAuthController::class, 'register']);
Route::post('login', [PassportAuthController::class, 'login']);
  
Route::middleware('auth:api')->group(function () {
    Route::get('get-user', [PassportAuthController::class, 'userInfo']);
    Route::resource('products', ProductController::class);
});
```

Create PassportAuthController and ProductController. Use the below commands:

```
php artisan make:controller API/PassportAuthController
php artisan make:controller API/ProductController
```

Navigate to app/http/controllers/API directory and open the PassportAuthController.php file. And update the following methods into your PassportAuthController.php file:

```
<?php

namespace App\Http\Controllers\API;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;

class PassportAuthController extends Controller
{
  /**
   * Registration Req
   */
  public function register(Request $request)
  {
    $this->validate($request, [
      'name' => 'required|min:4',
      'email' => 'required|email',
      'password' => 'required|min:8',
    ]);

    $user = User::create([
      'name' => $request->name,
      'email' => $request->email,
      'password' => bcrypt($request->password)
    ]);

    $token = $user->createToken('Laravel8PassportAuth')->accessToken;

    return response()->json(['token' => $token], 200);
  }

  /**
   * Login Req
   */
  public function login(Request $request)
  {
    $data = [
      'email' => $request->email,
      'password' => $request->password
    ];

    if (auth()->attempt($data)) {
      $token = auth()->user()->createToken('Laravel8PassportAuth')->accessToken;
      return response()->json(['token' => $token], 200);
    } else {
      return response()->json(['error' => 'Unauthorised'], 401);
    }
  }

  public function userInfo()
  {

    $user = auth()->user();

    return response()->json(['user' => $user], 200);
  }
}
```

Next, Create rest API crud methods in ProductController.php. So navigate to app/http/controllers/API directory and open the ProductController.php file. And, update the following methods into your ProductController.php file:

```
<?php

namespace App\Http\Controllers\API;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;

use App\Models\Product;

use Validator;

class ProductController extends Controller
{

  /**
   * Display a listing of the resource.
   *
   * @return \Illuminate\Http\Response
   */
  public function index()
  {
    $products = Product::all();
    return response()->json([
      "success" => true,
      "message" => "Product List",
      "data" => $products
    ]);
  }

  /**
   * Store a newly created resource in storage.
   *
   * @param  \Illuminate\Http\Request  $request
   * @return \Illuminate\Http\Response
   */
  public function store(Request $request)
  {
    $input = $request->all();
    $validator = Validator::make($input, [
      'name' => 'required',
      'detail' => 'required'
    ]);
    if ($validator->fails()) {
      return $this->sendError('Validation Error.', $validator->errors());
    }
    $product = Product::create($input);
    return response()->json([
      "success" => true,
      "message" => "Product created successfully.",
      "data" => $product
    ]);
  }

  /**
   * Display the specified resource.
   *
   * @param  int  $id
   * @return \Illuminate\Http\Response
   */
  public function show($id)
  {
    $product = Product::find($id);
    if (is_null($product)) {
      return $this->sendError('Product not found.');
    }
    return response()->json([
      "success" => true,
      "message" => "Product retrieved successfully.",
      "data" => $product
    ]);
  }

  /**
   * Update the specified resource in storage.
   *
   * @param  \Illuminate\Http\Request  $request
   * @param  int  $id
   * @return \Illuminate\Http\Response
   */
  public function update(Request $request, Product $product)
  {
    $input = $request->all();
    $validator = Validator::make($input, [
      'name' => 'required',
      'detail' => 'required'
    ]);
    if ($validator->fails()) {
      return $this->sendError('Validation Error.', $validator->errors());
    }
    $product->name = $input['name'];
    $product->detail = $input['detail'];
    $product->save();
    return response()->json([
      "success" => true,
      "message" => "Product updated successfully.",
      "data" => $product
    ]);
  }

  /**
   * Remove the specified resource from storage.
   *
   * @param  int  $id
   * @return \Illuminate\Http\Response
   */
  public function destroy(Product $product)
  {
    $product->delete();
    return response()->json([
      "success" => true,
      "message" => "Product deleted successfully.",
      "data" => $product
    ]);
  }
}
```

Finally you can test your API with Postman the way it's explained in the former section.

## Built With

* [Git](https://git-scm.com) - You can install it from https://git-scm.com/downloads.
* [MySQL](https://www.mysql.com) - You can install it from https://www.mysql.com/downloads/.
* [MySQL Workbench](https://www.mysql.com/products/workbench/) - You can install it from https://dev.mysql.com/downloads/workbench/.
* [Visual Studio Code](https://code.visualstudio.com/) - The Editor used in this project
* [Composer](https://getcomposer.org) - Install it from https://getcomposer.org/download/

## Acknowledgments

* https://www.phpcodingstuff.com/blog/laravel-8-rest-api-crud-with-passport-auth-tutorial.html. Laravel 8 Rest API CRUD With Passport Auth Tutorial. The one I have followed to create this example project.
