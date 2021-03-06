Sitemap for Laravel 4/5
=======================

[![Build Status](https://travis-ci.org/dwightwatson/sitemap.png?branch=master)](https://travis-ci.org/dwightwatson/sitemap)

Sitemap is a package built specifically for Laravel 4/5 that will help you generate XML sitemaps for Google. Based on [laravel-sitemap](https://github.com/RoumenDamianoff/laravel-sitemap) this package operates in a slightly different way to better fit the needs of our project. A facade is used to access the sitemap class and we have added the ability to produce sitemap indexes as well as sitemaps. Furthermore, it's tested.

Read more about sitemaps and how to use them efficiently on [Google Webmaster Tools](https://support.google.com/webmasters/answer/156184?hl=en).

## Installation

Simply pop the correct version constraint in your `composer.json` file and run `composer update` (however your Composer is installed).

	// Laravel 5.0
    "watson/sitemap": "2.0.*"

    // Laravel 4.*
    "watson/sitemap": "1.1.*"

Now, add the service provider to your `app/config/app.php` file.

    'Watson\Sitemap\SitemapServiceProvider'

And finally add the alias to the facade, also in `app/config/app.php`.

    'Sitemap' => 'Watson\Sitemap\Facades\Sitemap'

## Usage

### Creating sitemap indexes
If you have a large number of links (50,000+) you will want to break your sitemaps out into seperate sitemaps with a sitemap index linking them all. You add sitemap indexes using `Sitemap::addSitemap($loc, $lastmod)` and then you return the sitemap index with `Sitemap::renderSitemapIndex()`. The `$lastmod` variable will be parsed by [Carbon](https://github.com/briannesbitt/Carbon) and then converted to the right format for a sitemap.

Here is an example controller that produces a sitemap index.

```
class SitemapsController extends BaseController
{
	public function index()
	{
		// Get a general sitemap.
		Sitemap::addSitemap('/sitemaps/general');

		// You can use the route helpers too.
		Sitemap::addSitemap(URL::route('sitemaps.posts'));
		Sitemap::addSitemap(route('sitemaps.users'));

		// Return the sitemap to the client.
		return Sitemap::renderSitemapIndex();
	}
}
```

Simply route to this as you usually would, `Route::get('sitemap', 'SitemapsController@index')` for Laravel 4 or you can use the route helper methods in Laravel 5 like `get('sitemap', 'SitemapsController@index')`.

### Creating sitemaps
Similarly to sitemap indexes, you just add tags for each item in your sitemap using `Sitemap::addTag($loc, $lastmod, $changefreq, $priority)`. You can return the sitemap with `Sitemap::renderSitemap()`. Again, the `$lastmod` variable will be parsed by [Carbon](https://github.com/briannesbitt/Carbon) and convered to the right format for the sitemap.

If you'd like to just get the raw XML, simply call `Sitemap::xml()`.

Here is an example controller that produces a sitemap for blog psots.

```
class SitemapsController extends BaseController
{
	pulblic function posts()
	{
		$posts = Post::all();

		foreach ($posts as $post)
		{
			Sitemap::addTag(route('posts.show', $post->id), $post->created_at, 'daily', '0.8');
		}

		return Sitemap::renderSitemap();
	}
}
```

**If you're pulling a lot of records from your database you might want to consider restricting the amount of data you're getting by using the `select()` method. You can also use the `chunk()` method to only load a certain number of records at any one time as to not run out of memory.**

## Configuration

To publish the configuration file for the sitemap package, simply run this Artisan command:

    php artisan config:publish watson/sitemap

Then take a look in `app/config/packages/watson/sitemap/config.php` to see what is available.

### Caching

By default, caching is disabled. If you would likd to enable caching, simply set `cache_enabled` in the configuration file to `true`. You can then specify how long you would like your views to be cached for. Keep in mind that when enabled, caching will affect each and every request made to the sitemap package.
