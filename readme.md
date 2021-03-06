## Laravel PHP Framework - Jai Understanding Laravel prackges 

[![Latest Stable Version](https://poser.pugx.org/laravel/framework/version.png)](https://packagist.org/packages/laravel/framework) 

This is just for understanding how packages in Laravel works :

I have spend days looking for the appropriate guide - how to use controllers, routes and config and after doing some research my self I found few key things that have to implemented while building packages.

Lets get to business:

Install latest version of Laravel via composer if you like. ( You can still use Laravel with out composer so its up to you  ).

Once Laravel is installed give its storage folder respective permissions and try and run home page.

Now I suppose Everything is working fine right !!!!

Lets create a package  I am doing it via composer  ( using composer is better for developers).

Before we start  make sure that Laravel debug is set to true ( app/config/app.php  debug:true) by default Laravel is now shipped with debug off, trunning it on will help you to identify and google the error quickly or else you would be looking at "oops message " which doesn't help you .

Ok ! now 

Laravel packages are build from workbench ... there is a whole different concept to this ..anyways
 
 1) Go  to app/config/workbench.php add your name and email here  this is  Important as packages structure will be based on this.
 
 2) Use Command Line Interface to navigate to Laravel 4 root folder, and then run:

    php artisan workbench jai/newpackage --resources

--resources will create  controllers and other imp folder. 

Note that `jai` represents a vendor (company name, personal name etc.), and `cms` represents a package name.
You get a message  in CLI 'package workbench created ......'
Now there should be a folder created "workbench/jai/newpackage...... "  

## Package Setup

4) Now we need to link the Laravel app with  the package created  so 
  Open `/app/config/app.php` to add a Service Provider to the end of the providers array:

	'providers' => array(
		// --
		'Jai\Newpackage\NewpackageServiceProvider',
	),
	
   Make sure that first letters are in cap ( Jai and Newpackage) J and N are in capitals.

5) To create a main package class, generate the file named `Newpackage.php` inside a path `/workbench/jai/Newpackage/src/Jai/Newpackage/` with the following code inside:

Make sure that capitals are entered where necessary!

  <?php namespace Jai\Newpackage;

	class Newpackage {

		public static function hello(){
		return "What's up .....!";
		}

	}

6) Register the new class with the Laravel’s IoC Container by editing Package Service Provider file `/workbench/jai/newpackage/src/Jai/Newpackage/WalkthroughServiceProvider.php` and make sure that the register method looks like this:

	  public function register()
	  {
	    $this->app['newpackage'] = $this->app->share(function($app)
	    {
	      return new Newpackage;
	    });
	  }

**Note: If your service provider cannot be found, run the php artisan dump-autoload command from your application's root directory.
 This NOT equall to running  composer dump autoload!
**

## Package Facade Generation

Although generating a facade is not necessary, Facade allows you to do something like this:

	  echo Newpackage::hello();

7) Create a folder named `Facades` under following path `/workbench/jai/newpackage/src/Jai/Newpackage/`

8) Inside the `Facades` folder create a file named `Newpackage.php` with the following content:

	  <?php namespace Jai\Newpackage\Facades;
	
	  use Illuminate\Support\Facades\Facade;
	
	  class Newpackage extends Facade {
	
	    /**
	     * Get the registered name of the component.
	     *
	     * @return string
	     */
	    protected static function getFacadeAccessor() { return 'newpackage'; }
	
	  }

9) Add the following to the register method in '/workbench/jai/newpackage/src/Jai/Newpackage/WalkthroughServiceProvider.php' of your Service Provider file:

	  $this->app->booting(function()
	  {
	    $loader = \Illuminate\Foundation\AliasLoader::getInstance();
	    $loader->alias('Walkthrough', 'Orangehill\Walkthrough\Facades\Walkthrough');
	  });

  So now your registered  method will look like this 

	  public function register()
	  {
	    $this->app['newpackage'] = $this->app->share(function($app)
	        {
	          return new Newpackage;
	        });
	
	    $this->app->booting(function()
	    {
	      $loader = \Illuminate\Foundation\AliasLoader::getInstance();
	      $loader->alias('Newpackage', 'Jai\Newpackage\Facades\Newpackage');
	    });
	  }


This allows the facade to work without adding it to the Alias array in app/config/app.php.

## Browser Test

10) Edit your `/app/routes.php` file and add a route to test if the package works:

	  Route::get('/package', function(){
	    echo Newpackage::hello();
	  });

If all went well you should see the `What's up .....` output in your browser after visiting the test url.

This article was forked from https://github.com/orangehill/Laravel-Workbench-Walkthrough.

Now adding routes !

As you know we can use the main routes files, but in many cases you want your package to have a routes by itself, hence :

##Adding routes 

1) Navigate to /workbench/jai/newpackage/src/'  and create a file routes.php  like this  
 
	  Route::get('/packageroute', function(){
	     echo " package route is working ";
	  });

2) Now we need to include this route  on our package boot  so navigate to '/workbench/jai/newpackage/src/Jai/Newpackage/WalkthroughServiceProvider.php', edit this file and add this line in boot method :
  
	  include __DIR__ . '/../../routes.php';
	
	  now Boot method should look like this 
	
	  public function boot()
	    {
	      $this->package('orangehill/walkthrough');
	      include __DIR__ . '/../../routes.php';
	      
	    }

3) To test this, type in packageroute and it will show you "package route is working" .

##loading a view in package via package route.

1) This is pretty straight, but nevertheless you need to make some adjustments to the way you call them.
Create a view  in "/workbench/jai/newpackage/src/views/" folder namely "newpackageview.blade.php" :

	  <html>
	  <head>
	    <title> New package view</title>
	  </head>
	  <body>
	    <p> This is our package view</p>
	    {{ $route_name}}
	
	  </body>
	  </html>

2) From your  package route file  call this view like this :

	  Route::get('/packageview', function()
	  {
	    return View::make('newpackage::newpackageview')->with('route_name','This is called from packageview');
	  });

3) Now  in the browset when you type packageview you should see :

	This is our package view
	
	This is called from packageview

##  Using a controller 

1) Using a controller is not that straight as you think it is , althought we have a controller folder, we need to tell lavel to include that controller folder by editing composer.json file  which is /workbench/jai/newpackage folder and add the controllers folder:

	  "autoload": {
	        "classmap": [
	            "src/migrations"
	            "src/controllers"
	        ],
 
 2) Make sure you add service provider namespace to global class.To do that navigate to /workbench/jai/newpackage/src/Jai/Newpackage/WalkthroughServiceProvider.php file edit it and add this on to top :

	     <?php namespace Jai\Newpackage;
	
	    use Illuminate\Support\ServiceProvider;

 3) Now let's create a controller.Nnavigate to /workbench/jai/newpackage/src/controllers and  create a new file newpackageController.php :

	  <?php namespace Jai\Newpackage\Controllers;
	
	  class newpackageController extends \BaseController
	  {
	      public function index()
	      {
	          return \View::make('newpackage::newpackageview')->with('route_name','This is called from newpackage controller');
	      }
	  }


 Make sure you get in '\' this will tell it to look from source code for that method.

 4) Via command line navigate to Laravel folder a run this command "php artisan dump-autoload" you should be see the following:

  Generating optimised class loader..
  Running for workbench [jai\newpackage]....

 What will this do ? This now autoloads the controller folder. 
 How can I check this ? Go to /workbench/jai/newpackage/vendor/composer/autoload_classmap.php this should have a  map in return array, like this, already placed :
 
	" 'Jai\\Newpackage\\Controllers\\newpackageController' => $baseDir . '/src/controllers/newpackageController.php',"

  5) Now let's add a route  in "/workbench/jai/newpackage/routes.php" file.

	Route::get('/packagecontroller', 'Jai\Newpackage\Controllers\newpackageController@index');

  6) Now when you run "packagecontroller" in your url you will see this message in your browser

    This is our package view.

    This is called from newpackage controller.

 That is all for now.  I will add more stuff like:  accessing config file , models , helpers and many more ...
 Follow me for more here: https://github.com/jaiwalker/followers

 Jai.
 Walker.


### License

The Laravel framework is open-sourced software licensed under the [MIT license](http://opensource.org/licenses/MIT)
