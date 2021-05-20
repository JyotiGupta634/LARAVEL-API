This is Demo app for Laravel API
Sending verification after user registration. 
before start open localhost/phpmyadmin on your 
browser and update your user table. Add otp_code 
and is_verified columns. set default value of
 is_verified class to 0.
 
## Step 1
 Setting up email. befor setting email driver you have to create
  app password for your mail account.
  you can visit this link( https://support.google.com/mail/answer/185833?hl=en-GB ) to learn how to create app password. After you generate app password open your `.env` file and go to the email section as shown below
````
MAIL_MAILER=smtp
MAIL_HOST=smtp.gmail.com
MAIL_PORT=587
MAIL_USERNAME=youremail@gmail.com
MAIL_PASSWORD= dasdasdasdasd // this is your app password.
MAIL_ENCRYPTION=tls
MAIL_FROM_ADDRESS=youremail@gmail.com
MAIL_FROM_NAME="${APP_NAME}" //your app name. Default is laravel. you can change app name editing .env file. 
````
## Step 2 
Generate a mail class. To send email we will generate mailable class. Run the given below command in your project root
````
php artisan make:mail VerificationMail
````
## Step 3
the new class VerificationMail.php will be generated under app/Mail. This class will have a build method which defines which view file to send as email.
````
<?php

namespace App\Mail;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Mail\Mailable;
use Illuminate\Queue\SerializesModels;

class VerificationMail extends Mailable
{
    use Queueable, SerializesModels;

    public $user;
    public $otp;
    /**
     * Create a new message instance.
     *
     * @return void
     */
    public function __construct($user,$otp)
    {
        $this->user = $user;
        $this->otp= $otp;
    }

    /**
     * Build the message.
     *
     * @return $this
     */
    public function build()
    {
        return $this->view('register');
    }
}

````
## Step 4
After mailable class you have to generate view file for email design(This is not mendatoy you can send email using mailable class too). 
Create a new file register.blade.php under resources/views and paste following code into it.
````

<!DOCTYPE html>
<html>
<head>
    <title>Email Verification </title>
</head>

<body>
<h2>Welcome to the site {{  $user['name'] }}</h2>
<br/>
 Please Verify your email: YourVerification code is: <b> {{ $otp }} </b>
</body>

</html>

````
you can customize this email view as you like.
## Step 5 Create otp() Method.
Create new Otp function inside your UserController.php
````
public function otp(){
        $otp = rand(100000,999999);
        return $otp;
    }
````

## Step 6 Sending Email
Since now we have generated the VerificationMail class. We can modify our register method of `UserController` to send emails.
inorder to do that open your Usercontroller.php and modify regster method as shown below.

````
//Dont forget to use class given below.
use App\Mail\VerificationMail;
use Illuminate\Support\Facades\Mail;
......
public function register(Request $request){
        $model=new User();
        $model->name= $request->name;
        $model->email= $request->email;
        $model->password=Hash::make($request->password);
        $otp=$this->otp();
        $model->otp_code=$otp;
        if($model->save()){
            Mail::to($model->email)->send(new VerificationMail($model,$otp));
            return [
                'message'=>"user Created, Please Check your email to verify your email.",
                'user'=>$model->email
            ];
        }else{
            return ['message'=>"User not Created"];
        }
}
````
Finally ! Send Email on User Registration should work just fine.

#...




Follow the given steps to setup login API in your mobile application using laravel.
## Step 1 
Install Laravel Sanctum.
go to your project directory and open command prompt or terminal and type following code.
~~~~
composer require laravel/sanctum
~~~~
## Step 2
publish config and migration files 
~~~~
php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"
~~~~
## Step 3 
Run Database migration. if you have alredy created the user table drop the table and run the migration.
````
php artisan migrate
````
## Step 4 
Add the Sanctum's middleware. Go to Kernel.php inside your App\Http Folder and add 
````
namespace App\Http;
use Illuminate\Foundation\Http\Kernel as HttpKernel;
use Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful;
````
Again scroll down and add following code in The application's route middleware groups section inside api as shown below.
````
EnsureFrontendRequestsAreStateful::class,
````
for Eg.
````
   /**
     * The application's route middleware groups.
     *
     * @var array
     */
    protected $middlewareGroups = [
        'web' => [
            \App\Http\Middleware\EncryptCookies::class,
            \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
            \Illuminate\Session\Middleware\StartSession::class,
            // \Illuminate\Session\Middleware\AuthenticateSession::class,
            \Illuminate\View\Middleware\ShareErrorsFromSession::class,
            \App\Http\Middleware\VerifyCsrfToken::class,
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
        ],

        'api' => [
            EnsureFrontendRequestsAreStateful::class,
            'throttle:api',
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
        ],
    ];
````
## Step 5
To use token based authentication add folloeing code inside User model. if you donot have User model create using command prompt.
````
use Laravel\Sanctum\HasApiTokens;
````
also add HasApiTokens inside User class as shown bwllow.
````
use Laravel\Sanctum\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens, Notifiable;
.....
}
````
## Step 6
Create Usercontroller using command prompt and add the following code inside user controller. 
````
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Models\User;
use Illuminate\Support\Facades\Hash;

class UserController extends Controller
{
    function login(Request $request)
    {
        $user= User::where('email', $request->email)->first();
        // print_r($data);
        if (!$user || !Hash::check($request->password, $user->password)) {
            return response([
                'message' => ['These credentials do not match our records.']
            ], 404);
        }

        $token = $user->createToken('my-app-token')->plainTextToken;

        $response = [
            'user' => $user,
            'token' => $token
        ];

        return response($response, 201);
    }
}
````
to test the code we can add dummy data to the user table and to add data you can add seeder for user model ` php artisan make:seeder UsersSeeder` 
and insert the record to the user table.
go to databases\seeders folder and open the seeder you just created add add the following code'
````
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Hash;
class UsersTableSeeder extends Seeder
{
    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        DB::table('users')->insert([
            'name' => 'Ram Prasad',
            'email' => 'ram@gmail.com',
            'password' => Hash::make('12345678')
        ]);
        //
    }
}
````
the Above code will create new record inside users table after you execute `php artisan db:seed --class=UsersSeeder
`
## add Route to the api.php
 ````
 Route::post("login",[UserController::class,'login']);
 ````
 ````
 Route::group(['middleware' => 'auth:sanctum'], function(){
    //you can add your secure urls here.
    });
   ````
   I will teach you how to test our login API. After successful attempt we will create API for User Register module. 
   



 
 
 


