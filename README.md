# WordPress Ajax Filter Posts

## Description

A WordPress plugin to filter posts with taxonomies terms and load more posts via Ajax.
You can add posts and filters via a **shortcode** on any page.

```
[ajax_filter_posts post_type="recipe" tax="meal_type, food_type, diet_type"  posts_per_page="12"]
```

This plugins uses no dependencies, is translatable and WPML ready.

## Parameters

- **post_type**
  Post type to show. Default post.

- **tax**
  A comma seperated list of taxonomies to filter the post by. Default `post_term`.

- **post_per_page**
  Numbers of maximum posts to retreive at a time. Default 12.

- **orderby**
  Value to order the posts by. Supports `ID`, `author`, `title`, `name`, `type`, `date`, `modified`, `parent`, `rand`, `comment_count`, `relevance`, and `menu_order`.
  Does **not** support `meta_value`, `meta_value_num`, `post_name__in`, `post_parent__in` `post_parent__in` because additionals arguments needs to be set with these orderby values. You can [add your own query arguments via a filter hook](#query-arugments) if you need that support. Defaults to `date`.

  Check the [WordPress documentation on Query arguments](https://developer.wordpress.org/reference/classes/wp_query/#order-orderby-parameters) for more information.

- **order**
  Order the posts ascending or descendings. Support `ASC` (1, 2, 3; a, b, c) and `DESC` (3, 2, 1; c, b, a). Defaults to `DESC`.

- **multiselect**
  Allow one or more active filters per taxonomy. Defaults to `true`: allow more active filters

- **id**
  Usefull for custom styling or to target specific instances of the shortcode in the filter hooks. Default not set.

## Overwriting template files

To easily overwrite template files you can copy one or more of the files in the templates folder to your own theme. Create a folder `ajax-filter-posts` in the root of your theme directory and copy the files in that newly created folder. Keep in mind that you have to keep the folder structure intact. For example: If you want a custom version of `loop.php`, you copy the file to `<<your-public-folder>>/wp-content/themes/<<your-theme>>/ajax-filter-posts/partials`.

## Motivation

I build a lot of sites that needed a functionality like this and decided to create a plugin for it. Although there are a lot of plugins doing something like this, they usually add a lot of bloat and are not developer friendly. This plugin is for a developer easier to implement, easier to edit and keeps te codebase cleaner.

## Installation

Clone this repo to your plugins or mu-plugins folder. When you load it in your mu-plugins folder, you have to call the plugin via a file that is directly in the `mu-plugins` folder. See [this article](https://www.sitepoint.com/wordpress-mu-plugins/) for more information.

## Filters hooks
As a developer you can overwrite functionality with WordPress hooks

### Query arugments
With the filter `ajax_filter_posts_query_args` you can pass or alter query arguments to all post queries made by this plugin.

For example you can add an extra taxonomy query.

```php
/**
 * Add the diet term on all the queries made with the shortcode ajax_filter_posts
 *
 * @param array $query_args 			query arguments set by the plugin Ajax Filter posts
 * @param array $shortcode_attributes 	all shortcode attributes
 *
 * @return array a updated list of query arguments
 */
function my_site_set_additional_term_for_ajax_filter_posts($query_args, $shortcode_attributes) {

	// Only show posts with the term vegan in the diet taxonomy
	$diet_tax_query_args = [
		[
			'taxonomy' => 'diet',
			'field'    => 'slug',
			'terms'    => 'vegan',
		],
	];

	// If there are already tax queries args set, merge my query args with the set args
	if ( !empty( $query_args['tax_query'] ) ) {
		$prev_set_tax_args = $query_args['tax_query'];
		$query_args['tax_query'] = [
			// Set the relationship to AND: we want only post with my term and the set terms by the user
      		// Also see https://developer.wordpress.org/reference/classes/wp_query/#taxonomy-parameters
			'relation' => 'AND',
			$diet_tax_query_args,
			$prev_set_tax_args
		];
		return $query_args;
	}

	// If there are no tax queries args already set, just add it
	$query_args['tax_query'] = $diet_tax_query_args;
	return $query_args;
}
add_filter('ajax_filter_posts_query_args', 'my_site_set_additional_term_for_ajax_filter_posts', 10, 2);
```

## License

GNU GENERAL PUBLIC LICENSE
