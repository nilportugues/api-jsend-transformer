# JSend API Transformer

[![Build Status]
(https://travis-ci.org/nilportugues/jsend-transformer.svg)]
(https://travis-ci.org/nilportugues/jsend-transformer) [![Coverage Status]
(https://coveralls.io/repos/nilportugues/jsend-transformer/badge.svg?branch=master&service=github?)]
(https://coveralls.io/github/nilportugues/jsend-transformer?branch=master) [![Scrutinizer Code Quality]
(https://scrutinizer-ci.com/g/nilportugues/jsend-transformer/badges/quality-score.png?b=master)]
(https://scrutinizer-ci.com/g/nilportugues/jsend-transformer/?branch=master) [![SensioLabsInsight]
(https://insight.sensiolabs.com/projects/2ee8556c-d480-4f6f-a645-0ee18c271867/mini.png)]
(https://insight.sensiolabs.com/projects/2ee8556c-d480-4f6f-a645-0ee18c271867) [![Latest Stable Version]
(https://poser.pugx.org/nilportugues/jsend/v/stable)]
(https://packagist.org/packages/nilportugues/jsend) [![Total Downloads]
(https://poser.pugx.org/nilportugues/jsend/downloads)]
(https://packagist.org/packages/nilportugues/jsend) [![License]
(https://poser.pugx.org/nilportugues/jsend/license)]
(https://packagist.org/packages/nilportugues/jsend) 



## Installation

Use [Composer](https://getcomposer.org) to install the package:

```json
$ composer require nilportugues/jsend
```


## Usage
Given a PHP Object, and a series of mappings, the **JSend Transformer** will represent the given data as a JSON object.

For instance, given the following piece of code, defining a Blog Post and some comments:

```php
$post = new Post(
  new PostId(9),
  'Hello World',
  'Your first post',
  new User(
      new UserId(1),
      'Post Author'
  ),
  [
      new Comment(
          new CommentId(1000),
          'Have no fear, sers, your king is safe.',
          new User(new UserId(2), 'Barristan Selmy'),
          [
              'created_at' => (new DateTime('2015/07/18 12:13:00'))->format('c'),
              'accepted_at' => (new DateTime('2015/07/19 00:00:00'))->format('c'),
          ]
      ),
  ]
);
```

And a Mapping array for all the involved classes:

```php
use NilPortugues\Api\Mapping\Mapper;

$mappings = [
    [
        'class' => Post::class,
        'alias' => 'Message',
        'aliased_properties' => [
            'author' => 'author',
            'title' => 'headline',
            'content' => 'body',
        ],
        'hide_properties' => [

        ],
        'id_properties' => [
            'postId',
        ],
        'urls' => [
            'self' => 'http://example.com/posts/{postId}',
            'comments' => 'http://example.com/posts/{postId}/comments'
        ],
    ],
    [
        'class' => PostId::class,
        'alias' => '',
        'aliased_properties' => [],
        'hide_properties' => [],
        'id_properties' => [
            'postId',
        ],
        'urls' => [
            'self' => 'http://example.com/posts/{postId}',
        ],
    ],
    [
        'class' => User::class,
        'alias' => '',
        'aliased_properties' => [],
        'hide_properties' => [],
        'id_properties' => [
            'userId',
        ],
        'urls' => [
            'self' => 'http://example.com/users/{userId}',
            'friends' => 'http://example.com/users/{userId}/friends',
            'comments' => 'http://example.com/users/{userId}/comments',
        ],
    ],
    [
        'class' => UserId::class,
        'alias' => '',
        'aliased_properties' => [],
        'hide_properties' => [],
        'id_properties' => [
            'userId',
        ],
        'urls' => [
            'self' => 'http://example.com/users/{userId}',
            'friends' => 'http://example.com/users/{userId}/friends',
            'comments' => 'http://example.com/users/{userId}/comments',
        ],
    ],
    [
        'class' => Comment::class,
        'alias' => '',
        'aliased_properties' => [],
        'hide_properties' => [],
        'id_properties' => [
            'commentId',
        ],
        'urls' => [
            'self' => 'http://example.com/comments/{commentId}',
        ],
    ],
    [
        'class' => CommentId::class,
        'alias' => '',
        'aliased_properties' => [],
        'hide_properties' => [],
        'id_properties' => [
            'commentId',
        ],
        'urls' => [
            'self' => 'http://example.com/comments/{commentId}',
        ],
    ],
];

$mapper = new Mapper($mappings);
```

Calling the transformer will output a **valid JSON response** using the correct formatting:

```php
use NilPortugues\Api\JSend\JSendTransformer;
use NilPortugues\Api\JSend\Http\Message\Response;
use NilPortugues\Serializer\Serializer;

$transformer = new JSendTransformer($mapper);

//Output transformation
$serializer = new Serializer($transformer);
$serializer->setSelfUrl('http://example.com/posts/9');
$serializer->setNextUrl('http://example.com/posts/10');
$serializer->addMeta('author',[['name' => 'Nil Portugués Calderó', 'email' => 'contact@nilportugues.com']]);

$output = $serializer->serialize($post);

//PSR7 Response with headers and content.
$response = new Response($output);

header(
    sprintf(
        'HTTP/%s %s %s',
        $response->getProtocolVersion(),
        $response->getStatusCode(),
        $response->getReasonPhrase()
    )
);
foreach($response->getHeaders() as $header => $values) {
    header(sprintf("%s:%s\n", $header, implode(', ', $values)));
}

echo $response->getBody();
```

**Output:**


```
HTTP/1.1 200 OK
Cache-Control: private, max-age=0, must-revalidate
Content-type: application/json; charset=utf-8
```

```json
{
    "status": "success",
    "data": {
        "post_id": 9,
        "title": "Hello World",
        "content": "Your first post",
        "author": {
            "user_id": 1,
            "name": "Post Author"
        },
        "comments": [
            {
                "comment_id": 1000,
                "dates": {
                    "created_at": "2015-07-18T12:13:00+00:00",
                    "accepted_at": "2015-07-19T00:00:00+00:00"
                },
                "comment": "Have no fear, sers, your king is safe.",
                "user": {
                    "user_id": 2,
                    "name": "Barristan Selmy"
                }
            }
        ]
    },
    "links": {
        "self": {
            "href": "http://example.com/post/9"
        },
        "next": {
            "href": "http://example.com/post/10"
        }
    }
}
```

## Quality

To run the PHPUnit tests at the command line, go to the tests directory and issue phpunit.

This library attempts to comply with [PSR-1](http://www.php-fig.org/psr/psr-1/), [PSR-2](http://www.php-fig.org/psr/psr-2/), [PSR-4](http://www.php-fig.org/psr/psr-4/) and [PSR-7](http://www.php-fig.org/psr/psr-7/).

If you notice compliance oversights, please send a patch via [Pull Request](https://github.com/nilportugues/jsend-transformer/pulls).



## Contribute

Contributions to the package are always welcome!

* Report any bugs or issues you find on the [issue tracker](https://github.com/nilportugues/jsend-transformer/issues/new).
* You can grab the source code at the package's [Git repository](https://github.com/nilportugues/jsend-transformer).



## Support

Get in touch with me using one of the following means:

 - Emailing me at <contact@nilportugues.com>
 - Opening an [Issue](https://github.com/nilportugues/jsend-transformer/issues/new)
 - Using Gitter: [![Gitter](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/nilportugues/jsend-transformer?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge)



## Authors

* [Nil Portugués Calderó](http://nilportugues.com)
* [The Community Contributors](https://github.com/nilportugues/jsend-transformer/graphs/contributors)


## License
The code base is licensed under the [MIT license](LICENSE).
