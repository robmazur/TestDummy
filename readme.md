# TestDummy

![image](https://dl.dropboxusercontent.com/u/774859/GitHub-Repos/testdummy/crashtestdummy.jpg)

TestDummy makes the process of preparing your database for integration tests as easy as possible. ...As easy as:

### Make a post model with dummy attributes.

```php
use Laracasts\TestDummy\Factory;

$post = Factory::build('Post');
```

### Build a song entity, along with any set relationships, and save it to the database.

```php
use Laracasts\TestDummy\Factory;

$song = Factory::create('Song');
```

### Create and persist three comment entities

```php
use Laracasts\TestDummy\Factory;

Factory::times(3)->create('Song');
```

### Create three posts, but override the default title

```php
use Laracasts\TestDummy\Factory;

Factory::times(3)->create('Post', ['title' => 'My Title']);
```

## Usage

### Step 1: Install

Pull this package in through Composer, per usual:

```js
"require-dev": {
    "laracasts/testdummy": "1.*"
}
```

### Step 2: Create a fixtures.yml file

TestDummy will fetch the default attributes for each of your entities from a `fixtures.yml` file that you place in your tests directory. If using Laravel, this file may be added anywhere under the `app/tests`
directory. Here's an example of a `Post` and an `Author`.

```yaml
Post:
  title: Hello World $string
  body: $text
  published_at: $date
  author_id:
    type: Author

Author:
    name: John Doe $integer
```

Take note that the object name should correspond to your full namespaced model. For instance, if you use a `Models` namespace, then do:

```yaml
Models\Post:
  title: Hello World $string
```

#### Dynamic Values

Also, notice that we can use a number of dynamic values here:

- `$string`: A simple placeholder word
- `$text`: A paragraph of dummy text
- `$date`: A date in the format of "Y-m-d H:i:s', suitable for database timestamps
- `$integer`: Any unique number

Here are some usage examples:

```yaml
Post:
    title: Some Post: Number $integer
    body: $text
    publish_date: $date
    keywords: $string $string $string
```

This might return something similar to:

```bash
array(4) {
  ["title"]=>
  string(19) "Some Post: Number 1"
  ["body"]=>
  string(255) "Laborum rerum saepe et et voluptatem rerum. Debitis reiciendis dolores perferendis fugit. Et impedit sit reprehenderit quisquam. Dolor enim et quia. Excepturi rerum esse rerum amet omnis modi. Sint molestiae consequatur dolore omnis soluta minima tempora."
  ["publish_date"]=>
  string(19) "2013-05-05 20:33:12"
  ["keywords"]=>
  string(30) "consequatur provident pariatur"
}
```

#### Relationships

Pay attention to how we reference relationship types:

```yaml
Post:
  title: Some Post: Number $integer
  author_id:
    type: Author
```

You need to let TestDummy know the type of its associated model. TestDummy will then automatically build and save that relationship as well.

> This means that, when you do, say, `Factory::create('Song')`, if one of your columns references a `album_id`, then TestDummy will save an `Album` record to the database, too. This
will continue recursively. So, if the `Album` has an `artist_id`, then an artist will also be created.

### Step 3: Setup

When testing against a database, it's recommended that you recreate your environment for each test. That way, you can protect yourself against false positives. A SQLite database (maybe even one in memory) is a good choice in these cases.

If using Laravel with a DB in memory, you might do:

```php
public function setUp()
{
    parent::setUp();

    Artisan::call('migrate');
}
```

Better yet, place this code in a parent class, which you can then extend.

### Step 4: Write Your Tests

You're all set to go now. Start testing! Here's some code to get you started. Assuming that you have a `Post` and `Comment` model created...

```php

use Laracasts\TestDummy\Factory;

$comment = Factory::create('Comment');
```

This will create and save both a `Comment`, as well as a `Post` record to the database.

Or, maybe you need to write a test to ensure that, if you have three songs with their respective lengths, when you call a `getTotalLength` method on the owning `Album` model, it will return the correct value. That's easy!

```php
// create three songs, and explicitly set the length
Factory::times(3)->create('Song', ['length' => 200]);

$album = Album::first(); // this will be created once automatically.

$this->assertEquals(600, $album->getTotalLength());
```

## Extras

For Laravel users, to save a bit of time, a helper `Laracasts\TestDummy\DbTestCase` class is included with this package. If you extend it,
before each test, your test database will be migrated, and all DB modifications will be filtered through a transaction, and then rolled back on `tearDown`. This will give you a speed boost, and ensure that all tests start with the same database structure.

```php

use Laracasts\TestDummy\DBTestCase;

class ExampleTest extends DBTestCase {

    /** @test */
    function it_does_something()
    {
        // before this test fires, the DB will be migrated.
        // DB transactions will be used automatically to sve time.
    }
}
```
