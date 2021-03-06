# PHP library for the Wordpress REST API

Currently only supports GET requests.

---

**TABLE OF CONTENTS:**

- [Installation](#installation)
- [Authentication](#authentication)
  - [Basic Authentication](#basic-authentication)
  - [API Token](#api-token)
  - [OAuth](#oauth)
- [Usage](#usage)
  - [Posts](#posts)
  - [Pages](#pages)
  - [Taxonomies & Terms](#taxonomies-&-terms)
  - [Users](#users)
  - [Media](#media)
- [Custom Post Types](#custom-post-types)
- [Slightly more advanced stuff](#slightly-more-advanced-stuff)
  - [Endpoints](#endpoints)
  - [Guzzle](#guzzle)
  - [Custom Classes](#custom-classes)
- [Todo:](#todo)

---

## Installation

Install via composer:

```
composer require maneuver/channel
``` 

And include the autoloader:

```php
require('vendor/autoload.php');
```

Happy times. 🤙



## Authentication

### Basic Authentication

Make sure the [Basic Authentication](https://github.com/WP-API/Basic-Auth) plugin for Wordpress is installed and activated.  
_(should only be used for development purposes, as stated by the repository)_

```php
$channel = new \Maneuver\Channel([
  'uri' => 'http://example.com/wp-json/wp/v2/',
  'username' => 'your-username',
  'password' => 'your-password',
]);
```

### API Token

Make sure the [Rooftop API Authentication](https://github.com/davidmaneuver/rooftop-api-authentication) plugin is installed and activated.

```php
$channel = new \Maneuver\Channel([
  'uri' => 'http://example.com/wp-json/wp/v2/',
  'token' => 'your-token',
]);
```

### OAuth

_Currently not implemented._



## Usage

### Posts

Retrieve a list of all posts (where post_type = 'post'):

```php
$posts = $channel->getPosts();

echo count($posts);
```

Retrieve a post by ID:

```php
$post = $channel->getPost(1);

echo $post->excerpt;
```

Using Twig? Fear not:

```twig
<h1>{{ post.title }}</h1>
<p>{{ post.excerpt|raw }}</p>
```



### Pages

Retrieve a list of all pages:

```php
$pages = $channel->getPages();

foreach ($pages as $page) {
  echo $page->title;
}
```

Retrieve a page by ID:

```php
$page = $channel->getPage(1);

echo $page->content;
```



### Taxonomies & Terms

Retrieve all existing taxonomies:

```php
$taxonomies = $channel->getTaxonomies();
```

Retrieve one taxonomy by slug:

```php
$taxonomy = $channel->getTaxonomy('category'); // use singular taxonomy name

// Then you can retrieve its terms:
$terms = $taxonomy->terms();
```

Or retrieve the terms in one call using the 'get' method:

```php
$terms = $channel->get('categories'); // use plural taxonomy name
``` 


### Users

Get all users:

```php
$users = $channel->getUsers();

echo $users[0]->name;
```


### Media

Get all media:

```php
$media = $channel->getMedia();
```



## Custom Post Types

When you define a custom post type in your Wordpress installation, make sure you set the ```show_in_rest``` option to ```true```. This exposes an endpoint in the REST API to retrieve the posts. [Read the docs](https://codex.wordpress.org/Function_Reference/register_post_type)

```php
add_action('init', function(){

  register_post_type('product', [
    'labels' => [
      'name'          => 'Products',
      'singular_name' => 'Product',
    ],
    'public'        => true,
    'show_in_rest'  => true,
    'rest_base'     => 'products' // defaults to the post type slug, 'product' in this case
  ]);

});
```

Then use the general 'get' method:

```php
$products = $channel->get('products'); // Pass in the 'rest_base' of the custom post type.
```



## Slightly more advanced stuff

### Endpoints

You can actually call any endpoint using the 'get' method:

```php
$post_types = $channel->get('types');
$latest = $channel->get('posts?per_page=5');
```

Read more about all endpoints in the [REST API Handbook](https://developer.wordpress.org/rest-api/)

### Guzzle

You can pass in more requestOptions for Guzzle:

```php
$latest = $channel->get('posts?per_page=5', [
  'proxy' => 'tcp://localhost:8125',
]);
```

Read more about the Guzzle RequestOptions [here](http://docs.guzzlephp.org/en/latest/request-options.html).


### Custom Classes

Every call returns an object (or array of objects) extending the '\Maneuver\Models\Base' class. You can define your own classes if needed.

NOTE: Don't extend the '\Maneuver\Models\Base' class directly, you'll lose some functionality.

```php
class MyPost extends \Maneuver\Models\Post {
  public function fullTitle() {
    return 'Post: ' . $this->title;
  }
}

class MyPage extends \Maneuver\Models\Page {

}

$channel->setCustomClasses([
  // 'type' => 'ClassName'
  // eg: 'user' => 'MyUser'
  // or:
  'post'    => 'MyPost',
  'page'    => 'MyPage',
  'product' => 'MyPost', // custom post type
]);

$post = $channel->getPost(1);

echo $post->fullTitle();

echo get_class($post);
// => 'MyPost'
```

---

## Todo:

- More support for ACF fields
- Better support for images
- Add WP_Query-like parameters
- OAuth authentication
