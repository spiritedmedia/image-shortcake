# Image-shortcake #
**Contributors:** fusionengineering
**Tags:** shortcodes, images
**Requires at least:** 3.0.1
**Tested up to:** 4.2
**Stable tag:** 0.1
**License:** GPLv2 or later
**License URI:** http://www.gnu.org/licenses/gpl-2.0.html

Image Shortcake adds a shortcode for images, so that themes can template and
filter images displayed in posts. It is designed to work with the UI provided
by the [Shortcake (Shortcode UI)](https://github.com/fusioneng/Shortcake)
plugin,

## Description ##

When images are inserted into posts from the media library or media uploader,
only the html of the `<img>` tag and the link around it (if any) are preserved.
This means that themes which want to change the way images are marked up in
content don't have an easy way of doing this.

Image Shortcake is an attempt to solve this problem, by saving images in post
content as _shortcodes_ rather than HTML. The output of shortcodes can be
easily filtered in themes, plugins and templates, and since the original
attachment data is preseved as attributes on the shortcode, it becomes much
easier for modify the way images are marked up in themes.

For best results, use this with the [Shortcake (Shortcode
UI)](https://github.com/fusioneng/Shortcake) plugin. Shortcake offers an
easy to use interface to manage shortcodes in post content.

What could you use this for? Well, at [Fusion](http://fusion.net) we use this
shortcode for:

* **Responsive Images**. By filtering the output of the `[img]` shortcode
  image tag, we're able to insert the `srcset` attribute, so that all of
  the images on our site are served responsively to browsers that support
  that.

* **Inline sharing buttons**. We've added share links to each of the
  images on our site. Because these are inserted through a filter on
  a shortcode and not in the post content, it's easy to modify them on the
  fly. And having this logic in template files rather in on-page javascript
  that runs after page load makes it quicker for users.

* **Photo credits**. We've added "credit" as an image meta field, and we
  use a filter on 'img_shortcode_output_after_linkify' to display it on all
  images.

See the [Installation](#Installation) section for more ideas and tips for
custom image templates.

## Installation ##

### Customizing Output ###

The whole point of using shortcodes rather than HTML tags for images is
that you can customize the markup of images in your theme. This plugin
offers three primary hooks to modify the output:

* `img_shortcode_output_img_tag`: Filters the output of the <img> tag
  before wrapping it in link or caption
* `img_shortcode_output_after_linkify`: Filters the output of the <img>
  tag after wrapping in link
* `img_shortcode_output_after_captionify`: Filters the output of the <img>
  tag after wrapping in link and attaching caption

You can, of course, disregard the markup generated by this plugin
altogether and use a template part for images if you want. This example
adds EXIF data below all images containing those fields, in all post content:

in the theme's `functions.php`:

	```
	add_filter( 'img_shortcode_output_img_tag', 'load_image_template', 10, 2 );

	function load_image_template( $_, $attributes ) {
		ob_start();
		get_template_part( 'inline-image' );
		return ob_get_clean();
	}
	```

in a template file called `inline-image.php`:

	```
	<?php

	$attachment = $attributes['attachment'];
	$size       = $attributes['size'];
	$class      = $attributes['classes'];
	$align      = $attributes['align']
	$alt        = $attributes['alt']
	$linkto     = $attributes['linkto']

	$attachment_meta = wp_get_attachment_metadata( intval( $attachment ) );
	$exif_data = ( $attachment_meta['image_meta']['camera'];

	echo wp_get_attachment_image( $attachment, $size, null,
		array(
			'class' => "$class $align attachment-$size",
			'alt'   => $alt,
		)
	);

	if ( is_array( $exif_data ) ) {
		echo '<ul class="image-meta">';
		foreach ( $exif_data as $field => $value ) {
			echo '<li>' . $field . ': ' . $value . '</li>';
		}
		echo '</ul>';
	}
	```


### Data Migration ###

The plugin comes with two [WP-CLI](http://wp-cli.org) commands to migrate
images in your existing content into the `[img]` shortcode format used by
this plugin. _Note: if it isn't clear, this is an early release -- use
at your own risk, and make sure you've backed up your posts before
migrating!_

`wp image-shortcake migrate <ids> [--dry-run]`

This command searches the post content of the posts specified in `<ids>`,
and replaces any `<img>` tags or `[caption]` shortcodes with `[img]`
shortcodes. Currently it only catches images added through the media
library; custom img tags will not be converted.

If you add the `--dry-run` flag, no replacements will actually be
performed, just a summary report of the changes which would have been
made.

`wp image-shortcake revert <ids> [--dry-run]`

This command finds all `[img]` shortcodes in the content of any of the
posts specified in `<ids>`, and replaces them with the markup that would
be generated by those shortcodes. Note that this runs any filters in your
theme, so that if you have filtered the output of the shortcodes at any
output, those filters will be reflected in the coverted post content.


## Screenshots ##

### Shortcode UI form as accessed from Insert Media > Insert Post Element ###
![Shortcode UI form as accessed from Insert Media > Insert Post
Element](http://s.wordpress.org/extend/plugins/image-shortcake/screenshot-1.png)

This is the Shortcode UI form as accessed from Insert Media > Insert Post
Element.  (Note that you can also insert images as usual, by inserting them in
the Media Library - they will be transparently converted into shortcodes behind
the scenes for you.)

### Image preview renders in the editor just as usual ###

![Image preview in editor](http://s.wordpress.org/extend/plugins/image-shortcake/screenshot-2.png)

Image preview renders in the editor just as it normally would. The Shortcake
plugin's edit/delete buttons are available to modify the shortcode through the
provided UI.


## Changelog ##

### 0.1 ###
Initial release (May 1, 2015)

