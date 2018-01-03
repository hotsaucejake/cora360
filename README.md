# Cora

Chatbot Server for Corus360 

## Table of Contents
* [Core Concepts](#core-concepts)
   * [Hearing Messages](#hearing-messages)
      * [Basic Commands](#basic-commands)
      * [Command Parameters](#command-parameters)
      * [Matching Regular Expressions](#matching-regular-expressions)
      * [Command Groups](#command-groups)
         * [Drivers](#drivers)
         * [Middleware](#middleware)
         * [Recipients](#recipients)
      * [Fallbacks](#fallbacks)


## Core Concepts

### Hearing Messages

#### Basic Commands

The easiest way to listen for Cora commands is by "listening" for a specific keyword and a Closure, providing a very simple and expressive method of defining bot commands.

```php
$cora->hears('foo', function ($bot) {
    $bot->reply('Hello World');
});
// Process incoming message
$cora->listen();
```

In addition to the Closure you can also pass a class and method that will get called if the keyword matches.

```php
$cora->hears('foo', 'Some\Namespace\MyBotCommands@handleFoo');
// Process incoming message
$cora->listen();

class MyBotCommands {

    public function handleFoo($bot) {
        $bot->reply('Hello World');
    }

}
```

#### Command Parameters

Listening to basic keywords is fine, but sometimes you will need to capture parts of the information your Cora users are providing. For example, you may need to listen for a command that captures a user's name. You may do so by defining command parameters:

```php
$cora->hears('call me {name}', function ($bot, $name) {
    $bot->reply('Your name is: '.$name);
});
```

You may define as many command parameters as required by your command:

```php
$cora->hears('call me {name} the {adjective}', function ($bot, $name, $adjective) {
    //
});
```
Command parameters are always encased within `{}` braces and should consist of alphabetic characters only.

#### Matching Regular Expressions

If command parameters do not give you enough power and flexibility for Cora commands, you can also use more complex regular expressions in Cora commands. For example, you may want Cora to only listen for digits. You may do so by defining a regular expression in your command like this:

```php
$cora->hears('I want ([0-9]+)', function ($bot, $number) {
    $bot->reply('You will get: '.$number);
});
```

Just like the command parameters, each regular expression match group will be passed to the handling Closure.

#### Command Groups

Command groups allow you to share command attributes, such as middleware or channels, across a large number of commands without needing to define those attributes on each individual command. Shared attributes are specified in an array format as the first parameter to the `$cora->group` method.

##### Drivers

A common use-case for command groups is restricting the commands to a specific messaging service using the `driver` parameter in the group array:

```php
$cora->group(['driver' => SlackDriver::class], function($bot) {
    $bot->hears('keyword', function($bot) {
        // Only listens on Slack
    });
});
```

##### Middleware

You may also group your commands, to send them through custom middleware classes. These classes can either listen for different parts of the message or extend your message by sending it to a third party service. The most common use-case would be the use of a Natural Language Processor like wit.ai or api.ai.

```php
$cora->group(['middleware' => new MyCustomMiddleware()], function($bot) {
    $bot->hears('keyword', function($bot) {
        // Will be sent through the custom middleware object.
    });
});
```

##### Recipients

Command groups may also be used to restrict the commands to specific recipients, meaning they get restricted to the user sending the message to Cora.

```php
$cora->group(['recipient' => '1234567890'], function($bot) {
    $bot->hears('keyword', function($bot) {
        // Only listens when recipient '1234567890' is sending the message.
    });
});
```

##### Fallbacks

Cora allows you to create a fallback method, that gets called if none of your "hears" patterns matches. You may use this feature to give Cora users guidance on how to use her.

```php
$cora->fallback(function($bot) {
    $bot->reply('Sorry, I did not understand these commands. Here is a list of commands I understand: ...');
});
```
