# Key points for this evaluation

## Application Design

### The developer understood well about provided  specifications.

You need to read the given specifications *before* you start designing the applications/features. For example, The specs on this time says that:
```
This app should be worked as "Single Page Application".
```

But some guys build normal apps with page translations. Be careful about the specs and think about *what* and *why* they required so. Think about *why* is also very important because with this kind of thoughts, you can image outside of specs properly.


### The code is DRY

Wikipedia says:
```
In software engineering, don’t repeat yourself (DRY) is a principle of software development, aimed at reducing repetition of information of all kinds, especially useful in multi-tier architectures.
```
Non-DRY Application is really difficult to maintain. Because if same/similar logics are distributed in many places, modifying/updating the logic brings huge pain, and also bring tons of bugs ( e. g. if you forget modifying some part, logic inconsistency happens ).

Other Reference: [Software Principles to Teach Every New Recruit](http://techblog.chegg.com/2015/04/08/software-principles-to-teach-every-new-recruit/)

For example, some guys wrote almost all of logics in controller methods. That is not DRY. Because it happens that different endpoints have similar logics and features.

For example, one guy write the following logics ( calc when user get the chance to draw gacha free ) in many controllers.

```
now =	(new DateTime())->getTimestamp();
if(Auth::user()->time_free_normal==null){
    Auth::user()->time_free_normal=date('Y-m-d H:i:s');
    Auth::user()->save();
}
```

If requirement changes in the future to use different parameters to calc the time, 


### All module's role and granularity are well designed

Controllers are the place which you should do only the following things.

* Input Value Validations.
* Set of function(method) calls to execute business logics.
* Build a response.

Never write business logics on controllers to make the application DRY.
### All objects are well abstracted and reusability are properly kept.

Similar class need to be inherited from same base class to have same interface. ( Use `interface` or `mix-in` also if it needed ).

With that, code will be more organized and readability will be increased. And also, code will be bind to the well designed interface makes increase extendability because developers don't need to think about interface on each time he/she add new sub classes and also it will be easily plugged in to existing systems.
For example, there are 3 types of gachas. So define base abstract class should be defined. And the base gacha class may have "draw" method to draw gacha.

One guy defined `GachaService` and `BoxGachaService` and other gacha services inherit it. 

### Database table structure is well designed

One important point of data table design is reduce future table modification ( alter table ). 

For example, see the following definition:

```
$table->timestamp('n_gacha_free_ply_time')->nullable(); // normal gacha
$table->timestamp('e_gacha_free_ply_time')->nullable(); // expensive gacha
$table->timestamp('b_gacha_last_reset_time')->nullable(); // box gacha
```
He/she defined one column for each type of gacha. If we want to add new type of Gacha in the future, we need to do alter table.
You need to design tables which alter table is not required when new Gacha type coming. Because we can easily predict that we need to add new Gacha type in the future.
You need to make the app ready for easy predictable upgrade.


### Use OR mapper properly

Modern web framework provide OR Mapper. It's make database access easy and also increase code readability also.
And common OR mapper issue is N+1 problem or some other similar problem about inefficient queries.

Some senior developers prefer raw SQLs rather than OR mapper generated one, but only the really senior developers can do this, and we all do 



### Use Cache properly
Use on memory cache is good for performance because it reduces database access. You always think about performance.

You can cache data which is rarely changed. For example, master data ( meta data ) is good for caching.


### URL is well designed

* URL need to be descriptive
* Don't use verb ( use noun ). because verb can be represented by HTTP Methods ( GET/POST/PUT/DELETE.. )
* Never use upper case alphabet / camelCase. ( e. g. 'timeLine' )
* Don't use "array" "list". Use plural form instead.

For example, one guy uses `/get_list_items` for the endpoint to get item list, but `items` is better. Because HTTP GET method represents "get" and items include the concept ot 'list'.

And check general URL rules. There are many good articles on the web. ( Don't refer other pages URL itself. sometimes they are also bad )

Reference: [REST-ful URI design](http://blog.2partsmagic.com/restful-uri-design/)

### API is well designed

Use HTTP Status Code Properly. 200 means okay. and if new data created, use 201. and return 4XX and 5XX when error occurs.

For JSON data design, you can read [Google'S JSON guide](https://google-styleguide.googlecode.com/svn/trunk/jsoncstyleguide.xml) for better structure.

If you use JSONSchema for designing JSON response ( request ) data, it's really good for testing.

### All module follow the "Single Responsibility Principle"

One module ( class, function ) should have only one responsibility, means never implement 2 or more logics in one function, one class.

For example, one service class only focus on one target ( means GachaService focus on Gacha thing, not handle user authorization ).

### Object should use as "immutable"

From wikipedia:

```
In object-oriented and functional programming, an immutable object is an object whose state cannot be modified after it is created.
```

Some data type ( depends on the language ) are natively immutable. But on the other types of object, every function must not change internal data to avoid side effects.

For example:

```
$this->canDrawForFree($gacha)
```

No one think that this function changes $gacha. So you don't change internal data of `$gacha` in canDrawForFree function.

If function modified internal data of objects which it got through arguments, and also caller don't know that, some kind trouble will be happen.


## Coding

### It doesn't use Magic Number

From Wikipedia:
```
Unique values with unexplained meaning or multiple occurrences which could (preferably) be replaced with named constants
```

For example, if you decide to use 1 for type id of normal gacha type, you must define constants for it. Don't use 1 directly in the code like this:
```
if( $gachaType == 1 ){
  // bla bla bla
}
```
### Code is well formatted for readability

You can use code formatter like [js-beautify](https://www.npmjs.com/package/js-beautify).
But you never change format of the code which other developers wrote without discussion. And on team development, you need to discuss the basic rule for formatting ( like use space or tab for indent, and so on )

### Directory structure is well organized
Put same kind of files to the same directory. No need to create 2 directories for similar / same purpose.
One guy put config files to 2 different places. `/config` and `/app/conf`. It makes your code more confusing.


### Naming follows consistent naming rule.
To code readable, The code follow single naming rule.
For example:

```
sub merge_fast {
    my($dest, @srcs) = @_;
    for my $src (@srcs) {
        @{$dest}{keys(%$src)} = values(%$src);
    }
}

sub getDateStart {
    my ($time) = @_;
    $time = current_time() if (!$time);
    my @t = localtime($time);
    return Time::Local::timelocal(0, 0, 0, $t[3], $t[4], $t[5]);
}
```

Both Camel Case ( getDateStart ) and Snake Case ( merge_fast ) are used. That's too bad.
You should use only one of them. Each language have preferred case. Perl and Python prefer Snake Case and JavaScript, JSON and PHP prefer Camel Case. 
And also you should be aware the selection of terms. Such as "format", "get", "array", "list", "dictionary" and so on.


### Namespace is properly used

Some language have name space. You should use it.
For example, In PHP's PSR-4 rule, Name spacing should sync with directory structure.
But some developers code doesn't follow the rule, and as a result, PSR-4 autoloading doesn't work.

### All names are properly named.
Good name for functions, classes, methods, properties, variables, constants are:
* Descriptive, represents what it is. name should describe their own purpose well.
* Don't use uncommon abbreviation.

Good readable code doesn't need comments because good naming itself describe meaning of the code.

For abbreviation, You never use abbreviation. For example,

```
$table->timestamp('n_gacha_free_ply_time')->nullable(); // normal gacha
 $table->timestamp('e_gacha_free_ply_time')->nullable(); // expensive gacha
 $table->timestamp('b_gacha_last_reset_time')->nullable(); // box gacha
```
"n" for normal, "ply" for play is not common. Use "normal" and "play". Again, you never use abbreviation. Even if you think it's common, you don't use it because it is common in your country, it can not be common in other world.

Some developers use `Func` but it must be `Function`.

Only exception is standardized ones such as "JP", "JA" "VND" and "EN". because they are ISO standardized Country, Language, Currency Code. 


Variables, functions, classes, and all other names are descriptive ( means name describe their own purpose well )
All methods and functions are not too long ( less than 20 lines )
All classes and modules are not too long ( less than 100 lines )

Code is properly commented ( or enough readable without comment )
Input values are enough validated
Security is well considered
Follow de-facto coding and structure standard and rules of the language and the framework.


### All methods and functions should not be too long

Never write function longer than 20 lines. ( hopefully less than 10 lines ). If it becomes longer than 20 lines. Split it to 2 functions.

The benefits are:

* Shorter function is easy to read.
* Every function have it's own name. It means you can name the logic. So, if the granularity of the functions are small, readability of the code will become higher.

For example, if the gacha draw logics are packed into single method, you have to read step by step to understand what this function is doing.

```
sub draw {
	my($self, $type) = @_;
	
	# logics
}
```

However, if draw method was built with several function call, you can easily understand it's logic roughly.

```
sub draw {
	my($self, $type) = @_;
	
	my $user = $self->user_repository->get_auth_user();
	my $gacha = $self->gacha_repository->find($type);
	
	$price = $self->get_price($gacha, $user);
	if( !$user->can_pay($price) ){
		return null;
	}
	$item = $this->decide_item($user, $gacha);
	$this->add_item($user, $item);
	$this->spend_coin($price);
	return $item;
}
```

Of course, you also need to name them descriptive name.

### All classes and modules are not too long


Never write a class and module more than 100 lines. Longer class may have too much responsibility. Refactor it to split the class to 2 or more small classes.

Reason why is totally same with making small methods/functions.

### Code is properly commented ( or enough readable without comment )

Comment is important only when it's hard to understand what the code is doing. However, it doesn't mean you should write comment as much as possible.

For example:

```
# Get User
$user = $self->user_repository->find($user_id);
```

This kind of comment is not necessary because the code itself is enough descriptive.
The most readable code can easily understand without any comments because descriptive names of functions, classes, variables describe what the code is doing.
So when you need to write a comment, you always think the way to make it more descriptive without comment.

For example, 

```
$table->timestamp('n_gacha_free_ply_time')->nullable(); // normal gacha
 $table->timestamp('e_gacha_free_ply_time')->nullable(); // expensive gacha
 $table->timestamp('b_gacha_last_reset_time')->nullable(); // box gacha
```

With those comments, you can understand `n_` means normal, `e_` means expensive. But it's better to name those columns like `normal_gacha_play_time`, `expensive_gacha_play_time`. So you don't need comments.


### Input values are enough validated
You must not trust any input values from outside. You need to check input values are okay or not and return error with status code 400 to the client.

For example:

```
sub draw_gacha {
    my ($self, $c) = @_;
    my $gacha_id = $c->req->param('gacha_id');
    my $user_data = $c->session->get('user_data');
    die ('Invalid gacha id') unless ($gacha_id);
      :
```

This code check $gacha_id is given or not. But there are 2 problems.

* It doesn't check given gacha id is correct or not
* it will just die when $gacha_id is not given. need to return 400.

There are so many people try to cheat the game. You have to imagine what kind of attack the people will try and protect your game in advance.


### Security is well considered

Not only the input parameters, you need to be careful about other security issues also.
For example, there are so many HTTP response headers for security. 
`X-XSS-Protection` is one of them. You can see many others at following link.

Reference: [Guidelines for Setting Security Headers](https://www.veracode.com/blog/2014/03/guidelines-for-setting-security-headers)

And also, to protect XSRF(CSRF), you always use XSRF token. Some web framework ( e.g. Laravel 5 ) enable it by default.


### Follow de-facto coding and structure standard and rules of the language and the framework.
Every languages and web frameworks have their own rules ( or de-facto standard ) rules for coding, naming, structure and so on. You have to learn about it and must follow it. With following these kind of rules, your code becomes more readable.

Different language have different rules. For example, perl and python uses snake case (func_name) for naming, but PHP, JavaScript, JSON uses camel case (funcName). Many guys uses snake case function name in PHP but it should be camel case. Because for PHP, there are standard naming rule named PSR-1 and it says method name should be camel case.

Reference: [PSR](http://www.php-fig.org)

You need to research coding rules before start writing and discuss with other developers which rules you should use.

Many companies disclose their own coding rules and some of them become de-facto standard of the language/framework.

For example:

* [Google JSON Style Guide](https://google-styleguide.googlecode.com/svn/trunk/jsoncstyleguide.xml)
* [Google JavaScript Style Guide](http://google-styleguide.googlecode.com/svn/trunk/javascriptguide.xml)
* [AirBnB JavaScript Style Guide](https://github.com/airbnb/javascript)
* [NYTimes Objective-C Style Guide](https://github.com/NYTimes/objective-c-style-guide)

## Usage of existing information/code

### The app doesn't implement the general functions by itself which it should use third-party libraries.

There are so many useful libraries for various purpose. You should use common libraries instead of doing implementation by yourself.

* Well developed libraries will be shorten development time
* Common libraries makes your code readable. Because such a common libraries are well known and other developers also be able to understand the purpose soon.

For example, Facebook, Amazon AWS, and other big service provide not only HTTP API but also SDK to use it. Perl have many common de-facto standard libraries such as `List::Util`, `List::MoreUtils` and so on.

So you don't have to write similar functions. It was called "reinventing the wheel".
### The app doesn't use too much third-party library.
On the other hand, you should not use third party libraries without deep considerations.

Because sometimes bad libraries make many problems. For example:

* You will be stuck with a bug in a bad library.
* Framework update becomes difficult because of the libraries which stick to the particular version of the framework.

There are so many PHP library for Laravel4. Some of them are just Syntax Sugar and also no longer be updated. If you use this kind of library on Laravel4, it's become difficult to upgrade your application to Laravel5.

So you should use the following libraries.

* Official SDK libraries such as Facebook SDK, Amazon AWS SDK.
* De-facto standard libraries
* Well maintained libraries.

And you should not use

* Minor libraries which is not widely used.
* Libraries which was not updated more than 1 year.
* Libraries stick to the framework version.

Anyway, every time you want to use new library, you MUST research this library well to avoid future trouble. And sometimes there are many libraries which have same purpose. At that case, you MUST check all of them and choose the best one.

The factor you should check is:

* Most popular
* Bug Fix Speed is fastest
* Still actively upgrading


### The app do not bring the code from other project ( use Just copy & paste ) without any consideration.

Import classes, methods from other project ( or from the web such as Stack Overflow ) is really useful and shorten your development time.

But when you do that kind of copy & paste, you should not do "brainless copy & paste".

Check the following example.

```
# YYYY/MM/DD hh:mm:ss
# 形式の日付を受け取ってunixtimeを返す
sub str2time {
    my ($text) = @_;
    my ($y, $m, $d, $h, $i, $s) = ($text =~ /(\d{4})\/(\d{2})\/(\d{2}) (\d{2}):(\d{2}):(\d{2})/);
    return 0 if (!$y);
    return Time::Local::timelocal($s, $i, $h, $d, $m-1, $y-1900);
}
```

This is obviously copied function from other project because the developer of this app cannot read Japanese.

Proper copy & paste is totally okay but you should do the following:

* Modify Comments to suit for the project
* Write where you copied from ( URL or Project Name for further reference)
* adjust naming rules to your project

And if you copy from the web ( like stack overflow ), you should find more than 3 solutions and compare them, then import the best one ( and leave source URL as a comment ). Top listed one on the google may not be a best solution.

## Testing

### The app prepared test code ( unit test and smoke test )

Almost no one prepare test code for this time. That's too bad. You MUST write test code every time.

You should write unit tests for all modules. and also write smoke tests for all entry points.

### All objects can work independently to be able to test easier

Make the code more test friendly, you should make all module can work independently ( means not depends on other modules )
For example, DI ( Dependency Injection ) is good practice for test friendly code. Some developers who choose PHP/Laravel uses DI technic on their code, so why don't you write unit tests ?
