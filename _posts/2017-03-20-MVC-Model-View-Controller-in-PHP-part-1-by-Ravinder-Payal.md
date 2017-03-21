---
layout: post
title: MVC (Model View Controller) in PHP part 1 by Ravinder Payal
author: ravinder_payal
page_id: blog1
---
Hello friends, in this series we are going to create Model View Controller in PHP using classes and methods. For a large application, a model view controller is really helpful for code structuring, and separating data (model), output (view), and logics (controller) layers. Many times View and Controller layers are combined together when further processing of output provided by logic layer is not required, or require a single function call like [`json_encode()`](http://php.net/manual/en/function.json-encode.php), as in case of APIs service where data is returned in the form of JSON or XML.
Along with MVC there is also a router layer, on top of MVC layers, which decides which logic to be called, and in turn logics decides what to call and do.

A basic diagram of service using MVC architecture looks like this:-
	
![Diagram showing flow of data in MVC structure](/blog-images/2017-03-20-01.png)
 
So, our first task is creating a layer which exposes our MVC to outside world. Here we go:-
```php
<?php

/*
 * 
 * (c) Ravinder Payal <mail@ravinderpayal.com>
 * 
 */

/**
 * Description of index
 *
 * @author Ravinder Payal
 */
define("OUT_MODE","GUI_MODE");
ini_set('xdebug.max_nesting_level', 1000);
/****Script to initiate the System**********/
function badReq($f="0"){
    header("HTTP/1.1 400 BAD REQUEST");
    die("400–Bad Request -".$f);

}
function unauthrisedReq(){
    header("HTTP/1.1 401 Unauthorised Request");
    OUT_MODE=="GUI_MODE"?die("401–Unauthrised Request"):die("{\"out\":\"401–Unauthorised Request\",\"out_code\":".OUT::REQ_UNAUTHORISED."}");
}

$path=array("","main","main");
if(isset($_SERVER['PATH_INFO']) and preg_match('/[a-z 0-9~%.:_\-\/]+$/iu', $_SERVER['PATH_INFO'])) {
    if (!ctype_alnum(trim($_SERVER['PATH_INFO'], "/")) and !preg_match("[/]",$_SERVER['PATH_INFO'])) {
        badReq(__FILE__." : ".__LINE__);
    }
    else{
        $path= explode('/',rtrim($_SERVER['PATH_INFO'],"/") );
    }
}
else{
    badReq(__FILE__." : ".__LINE__);
}
define("public_dir","public/");
define("private_dir","private/");
//isset($_GET["json"])? define("OUT_MODE","JSON_MODE"): define("OUT_MODE","GUI_MODE");

require private_dir."load.php";
if(isset($path[2])){
    $method=$path[2];
    $param = array_slice($path, 3);
    $controller=load($path[1]);
    if(method_exists($controller,$method))
        call_user_func_array(array($controller,$method),$param);
    else{
        $controller->output->error("404");
    }
}
else{
    load($path[1])->main();
}
```

In this block of code:- 

```php
$path=array("","main","main");
if(isset($_SERVER['PATH_INFO']) and preg_match('/[a-z 0-9~%.:_\-\/]+$/iu', $_SERVER['PATH_INFO'])) {
    if (!ctype_alnum(trim($_SERVER['PATH_INFO'], "/")) and !preg_match("[/]",$_SERVER['PATH_INFO'])) {
        badReq(__FILE__." : ".__LINE__);
    }
    else{
        $path= explode('/',rtrim($_SERVER['PATH_INFO'],"/") );
    }
}
else{
    badReq(__FILE__." : ".__LINE__);
}
```

We are processing URL and [stripping of] information about controller name and its method’s name. In URL like http://www.example.com/index.php/hello/world, our router will consider “Hello” as controller name, and “World” as method name. Further sub directories are supplied as arguments/parameters to methods of controllers [(It will cause an error if more arguments are supplied than requested)]. Our regex expression `'/[a-z 0-9~%.:_\-\/]+$/iu'` separates all directory names (or strings separated by “/” or “\”) in URL, and return an 1 D Array containing separated strings. Now we will proceed to next block of code which, using strings, call contoller and it’s methods.

>Remember, the [$_SERVER[‘PATH_INFO’](http://php.net/manual/en/reserved.variables.server.php) returns the after part of `index.php/` in URL.

>We can use url rewrite to get rid of index.php in URL.

```php
define("public_dir","public/");
define("private_dir","private/");
//isset($_GET["json"])? define("OUT_MODE","JSON_MODE"): define("OUT_MODE","GUI_MODE");

require private_dir."load.php";
if(isset($path[2])){
    $method=$path[2];
    $param = array_slice($path, 3);
    $controller=load($path[1]);
    if(method_exists($controller,$method))
        call_user_func_array(array($controller,$method),$param);
    else{
       error404(); //Right now we will make it simple and use simple function, but it will be changed to below one from next part.
       //$controller->output->error("404");
    }	
}
else{
    load($path[1])->main();
}
```
Here, $path[0] is blank as there is nothing useful before first “/” in URL, and $path[1] contains name of controller class, and $path[2] contains name of method. Our first layer proceeds further after checking if method name is supplied or not. In case not supplied, it will call `main` method of controller class. It can be inferenced that method with name `main` (or you name it) is required in our MVC. If method name is supplied, it extacts array of arguments from our `$path` array containing strings containing all parts of url (after index.php) separated by “/” or “\”. Here we are using a function named “load” defined in another file named (“load.php”)[LINK]. In load function we are having some logics which loads file containing class and return the instantiated class. After loading the desired controller, we check if there is method with the supplied name exists or not. If not, we will call a function which shows a `HTTP/1.1 404` error.
>It can be inferenced that method with name `main` (or you name it) is required in our MVC.

>[`array_slice($path, 3)`](http://php.net/manual/en/function.array-slice.php) returns all elements of array, after slicing off first 3 elements.

>[`method_exists`](http://php.net/manual/en/function.method-exists.php) check if a class have a method with supplied name or not.

>There are two native functions in PHP named call_user_func_array and call_user_func, but we are using call_user_func_array as it allows calling a method of class with an array of N-number of arguments.
>call_user_func is used for calling functions.(see php.net)


Upto now, we have coded a router/public layer in PHP which exposes our controllers, but wait where are our controllers. Before we code our first controller class, we will have a look at `load.php` file.
```php
//Load github code here, from tutorial branch of repo
//this code for reference purpose only
<?php
/*
 * 
 * (c) Ravinder Payal <mail@ravinderpayal.com>
 * 
 * Loads our whole application into memory for execution
 *
 * @author Ravinder Payal<www.ravinderpayal.com>
 *
 */

require dirname(__FILE__)."/config.php";

/**
 * 
 * Main Application Loader
 * var  $controller:- The controller class which we use to initiate the whole module.
 * 
 */
function load($controller,$dependencies = array()){
    require(COREPATH."Application.php");
    require(COREPATH."Controller.php");
    require(COREPATH."Model.php");
    require(COREPATH."Helper.php");
    /**
    *
    *Thinking about lazy loading these three class, as i18n is not required in all applications. Mail me your suggestions.
    *
    */
    require(COREPATH."Language.php");
    require COREPATH .'Database.php';
    require COREPATH .'Output.php';
    if(file_exists(APPPATH."controllers/".$controller.".php")){
        require(APPPATH . "controllers/" . $controller . ".php");
        if(class_exists(ucfirst($controller))) {
            return new $controller();
        }
        else{
            badReq();
        }
    }
    else{
        badReq();
    }
}
function out($a){
    echo "<pre>$a</pre></br>";
}
```

Now, we come to a new file name config.php, which is very important to us as, very few naming is hard coded in our MVC, and all non-hardcoded strings are defined in config.php file.

```php
<?php
/**
 * 
 * @author Ravinder Payal <mail@ravinderpayal.com>
 * @since v20160515
 * @version vCONFIG20170208
 * 
 */

/**
 * Constant for telling the System that the Application is running in Debug Mode
 */
define("MODE_DEBUG",true);

/**
*System specific contants
*/
define("DEFAULT_LANG","hi");
define("BASEPATH",dirname(__FILE__)."/");
define("BASE_SERVER_DIR",dirname(__DIR__)."/");
define("APPPATH",BASEPATH."application/");
define("COREPATH",BASEPATH."systemcore/");
define("MODEL_DIR","models");
define("UPLOAD_DIR_PUB",BASE_SERVER_DIR."public/files/");
define("LIBRARY_DIR","lib/");
define("LANG_DIR","languages/");
define("SQL_HOST","127.0.0.1");
define("SQL_DB","SystemPHP");
define("SQL_USER","root");
define("SQL_PASSWORD","root");

/**
*Application specific constants
*/
define("SESSION_TOKEN","PHPsession");
define("SESSION_ID","PHPsessionUserID");
define("SESSION_ADMIN","PHPsessionAdminLoggedIn");
define("SESSION_SUBADMIN","PHPsessionSubAdminLoggedIn");
define("SESSION_PRIV_USER","PHPsessionPrivUserLoggedIn");
define("SESSION_USER","PHPsessionUserLoggedIn");
define("SESSION_PREFIX","system");
define("COOKIES_TOKEN","PHPcookie");
/**
*Application specific GlobalVar.//From PHP 7, we have functionality for contants arrays similar to define(...,...). So, we will move to that in upcoming release.
*/
class GlobalVar{
    static public $SG_session_account_level = array(1 => SESSION_ADMIN,2=>SESSION_SUBADMIN,3=>SESSION_PRIV_USER,4=>SESSION_USER );
}
require COREPATH."constants/OUT.php";
```

This file is well commented and self explanatory. Period.

Now, we will code our first controller.
>From next part onwards we will see major changes in our controller, and additional functionalities.
File location:- …/public/application/controllers/firstcont.php

```php
<?php
defined('BASEPATH') OR exit('No direct script access allowed');

class printcont{
    function __construct(){
    }
    public function main(){
        echo "{\"success\":true}";
    }
    public function somemethod($a,$b,$c=”optional”){
	echo $a,$b,$c;
    }
    public function anothermethod($a=”optional”){
	//myLogic($a);
    }
}
```
Download complete code used in this part of tutorial from [Github tutorial branch of SystemPHP repository](https://github.com/ravinderpayal/systemPHP/tree/tutorialPart1).

Comment your questions, and if you learned something, please give your 5 seconds and like our github repo for this tutorial series, and opensource version of complete SystemPHP (inspired from CodeIgniter®)
>Youtube video series coming soon.
