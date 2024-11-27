# WP-Tweaks - Bundle of 'hacks' for WordPress and WooCommerce

This project contains several useful functions to optimize the behavior of WordPress and WooCommerce.

These code snippets should be inserted into the `functions.php` file, preferably in a child theme.

## WooCommerce
-----------------


### PHP Snippet: Add New Filter “Filter by featured status” on WooCommerce Products Admin Table
Filter by Featured @ WooCommerce Products Admin

```php
/**
 * @snippet       Filter by Featured @ WooCommerce Products Admin
 * @how-to        businessbloomer.com/woocommerce-customization
 * @author        Rodolfo Melogli, Business Bloomer
 * @compatible    WooCommerce 8
 * @community     https://businessbloomer.com/club/
 */
 
add_filter( 'woocommerce_products_admin_list_table_filters', 'bbloomer_featured_filter' );
 
function bbloomer_featured_filter( $filters ) {
   $filters['featured_choice'] = 'bbloomer_filter_by_featured';
   return $filters;
}
 
function bbloomer_filter_by_featured() {
   $current_featured_choice = isset( $_REQUEST['featured_choice'] ) ? wc_clean( wp_unslash( $_REQUEST['featured_choice'] ) ) : false;
   $output = '<select name="featured_choice" id="dropdown_featured_choice"><option value="">Filter by featured status</option>';
   $output .= '<option value="onlyfeatured" ';
   $output .= selected( 'onlyfeatured', $current_featured_choice, false );
   $output .= '>Featured Only</option>';
   $output .= '<option value="notfeatured" ';
   $output .= selected( 'notfeatured', $current_featured_choice, false );
   $output .= '>Not Featured</option>';
   $output .= '</select>';
   echo $output;
}
 
add_filter( 'parse_query', 'bbloomer_featured_products_query' );
 
function bbloomer_featured_products_query( $query ) {
    global $typenow;
    if ( $typenow == 'product' ) {
        if ( ! empty( $_GET['featured_choice'] ) ) {
            if ( $_GET['featured_choice'] == 'onlyfeatured' ) {
                $query->query_vars['tax_query'][] = array(
                    'taxonomy' => 'product_visibility',
                    'field' => 'slug',
                    'terms' => 'featured',
                );
            } elseif ( $_GET['featured_choice'] == 'notfeatured' ) {
                $query->query_vars['tax_query'][] = array(
                    'taxonomy' => 'product_visibility',
                    'field' => 'slug',
                    'terms' => 'featured',
                    'operator' => 'NOT IN',
                );
            }
        }
    }
    return $query;
}
```


### Sorting Products by Stock Quantity

Organizes products based on the quantity in stock, prioritizing available products when displaying listings in WooCommerce (store, categories, product tags).

```php
// Organizes products based on the quantity in stock, prioritizing available products when displaying listings in WooCommerce (store, categories, product tags).
class DWP_Orderby_Stock_Status {
    public function __construct(){
	if (in_array('woocommerce/woocommerce.php', apply_filters('active_plugins', get_option('active_plugins')))) {
	    add_filter('posts_clauses', array($this, 'order_by_stock_status'), 2000);
	}
    }

    public function order_by_stock_status($posts_clauses){
	global $wpdb;

	if (is_woocommerce() && (is_shop() || is_product_category() || is_product_tag())) {
	    $posts_clauses['join'] .= " INNER JOIN $wpdb->postmeta istockstatus ON ($wpdb->posts.ID = istockstatus.post_id) ";
	    $posts_clauses['orderby'] = " istockstatus.meta_value ASC, " . $posts_clauses['orderby'];
	    $posts_clauses['where'] = " AND istockstatus.meta_key = '_stock_status' AND istockstatus.meta_value <> '' " . $posts_clauses['where'];
	}

	return $posts_clauses;
    }
}

new DWP_Orderby_Stock_Status;
```

#### Alternative:
### 'Out of Stock' products to the end of the list

```php
// 'Out of Stock' products to the end of the list
if (in_array('woocommerce/woocommerce.php', apply_filters('active_plugins', get_option('active_plugins')))) {
    add_filter('posts_clauses', 'order_by_stock_status', 2000);
}

function order_by_stock_status($posts_clauses) {
    global $wpdb;
  
    if (is_woocommerce() && (is_shop() || is_product_category() || is_product_tag())) {
	$posts_clauses['join'] .= " INNER JOIN $wpdb->postmeta istockstatus ON ($wpdb->posts.ID = istockstatus.post_id) ";
	$posts_clauses['orderby'] = " istockstatus.meta_value ASC, " . $posts_clauses['orderby'];
	$posts_clauses['where'] = " AND istockstatus.meta_key = '_stock_status' AND istockstatus.meta_value <> '' " . $posts_clauses['where'];
    }
	return $posts_clauses;
}
```

### Shortcode to list all product categories
```php
// Shortcode to list all product categories
function display_product_categories() {
    $product_categories = get_terms( array(
	'taxonomy' => 'product_cat',
	'hide_empty' => false,
    ) );

    if ( ! empty( $product_categories ) && ! is_wp_error( $product_categories ) ) {
	$output = '<ul>';
	foreach ( $product_categories as $category ) {
	    $output .= '<li>' . $category->name . ' (ID: ' . $category->term_id . ')</li>';
	}
	$output .= '</ul>';
    } else {
	$output = 'Nenhuma categoria encontrada.';
    }

    return $output;
}

add_shortcode( 'product_categories_list', 'display_product_categories' );
```

### Return recent products when search has no results
```php
// Return recent products when search has no results
add_action( 'woocommerce_no_products_found', 'show_products_on_no_products_found', 20 );
function show_products_on_no_products_found() {
    echo '<h4 class="aligncenter quemsabe">' . __( 'Mas você pode gostar destes:', 'domain' ) . '</h4>';
    echo do_shortcode( '[recent_products per_page="4"]' );
}
```

### Permission for Store Managers to edit users
```php
// Permission for Store Managers to edit users
function wws_add_shop_manager_user_editing_capability() {
    $shop_manager = get_role( 'shop_manager' );
    $shop_manager->add_cap( 'edit_users' );
    $shop_manager->add_cap( 'edit_user' );
}
add_action( 'admin_init', 'wws_add_shop_manager_user_editing_capability');
```

### Removes Password Strength Check in Customer Registration
```php
// Removes Password Strength Check in Customer Registration
function iconic_remove_password_strength() {
    wp_dequeue_script( 'wc-password-strength-meter' );
}
add_action( 'wp_print_scripts', 'iconic_remove_password_strength', 10 );
```

### Rename order status 'Processing'
```php
// Rename order status 'Processing'
add_filter( 'wc_order_statuses', 'rename_completed_order_status' );
function rename_completed_order_status( $statuses ) {
   $statuses['wc-processing'] = 'Pedido Recebido';
   return $statuses;
}
```

## Wordpress
-----------------

### Remove ?ver= argument in url from styles/scripts
```php
// Remove ?ver= argument in url from styles/scripts
function remove_css_js_version( $src ) {
    if( strpos( $src, '?ver=' ) )
	$src = remove_query_arg( 'ver', $src );
    return $src;
}
add_filter( 'style_loader_src', 'remove_css_js_version', 9999 );
add_filter( 'script_loader_src', 'remove_css_js_version', 9999 );
```

### Remove WP version from Head and Feeds
```php
// Remove WP version from Head and Feeds
function wp_remove_version() {return '';}
add_filter('the_generator', 'wp_remove_version');
```

### Filter to remove Rank Math credits from the website's HTML body
```php
// Filter to remove Rank Math credits from the website's HTML body
add_filter( 'rank_math/frontend/remove_credit_notice', '__return_true' );
```

### Convert JPEG and PNG Images to WebP on Upload
This feature automatically converts JPEG and PNG images uploaded to WordPress to WebP format, optimizing website performance.
```php
// Convert JPEG and PNG Images to WebP on Upload
function convert_uploaded_images_to_webp($image_data) {
    if ($image_data['type'] != 'image/jpeg' && $image_data['type'] != 'image/png') {
	return $image_data;
    }

    if ($image_data['size'] > 5 * 1024 * 1024) {
	return new WP_Error('file_too_large', 'O tamanho do arquivo excede o limite permitido.');
    }

    $image_id = attachment_url_to_postid($image_data['url']);
    $metadata = wp_get_attachment_metadata($image_id);
    if ($metadata) {
	return $image_data;
    }

    $image = wp_get_image_editor(sanitize_text_field($image_data['file']));
    if (is_wp_error($image)) {
	return $image_data;
    }

    $size = $image->get_size();
    $dimensions = wp_constrain_dimensions($size['width'], $size['height'], 1920);
    $new_width = $dimensions[0];
    $new_height = $dimensions[1];
    $image->resize($new_width, $new_height);

    $filename = str_replace('.jpeg', '', $image_data['file']);
    $filename = str_replace('.jpg', '', $filename);
    $filename = $filename . '-new.webp';

    $image_data['caption'] = wp_strip_all_tags(esc_attr($image_data['caption']));
    $image_data['title'] = wp_strip_all_tags(esc_attr($image_data['title']));

    $saved = $image->save($filename, 'image/webp');
    if (is_wp_error($saved)) {
	return $image_data;
    }

    $original_filename = $image_data['file'];
    $new_filename = str_replace('.jpeg', '', $original_filename);
    $new_filename = str_replace('.jpg', '', $new_filename);
    $new_filename = $new_filename . '-new.webp';
    rename($original_filename, $new_filename);

    $image_data['file'] = $new_filename;
    $image_data['type'] = 'image/webp';

    return $image_data;
}

add_filter('wp_handle_upload', 'convert_uploaded_images_to_webp');
```

### SVG File Upload Support
This feature adds support for uploading SVG files to WordPress, allowing SVGs to be uploaded directly to the media library.

```php
// SVG File Upload Support
// This feature adds support for uploading SVG files to WordPress, allowing SVGs to be uploaded directly to the media library.
function add_svg_support( $mimes ) {
    $mimes['svg'] = 'image/svg+xml';
    return $mimes;
}

add_filter( 'upload_mimes', 'add_svg_support' );
```

### Integration with Google Tag Manager
Adds Google Tag Manager code to your website header for traffic monitoring and analytics collection.

```php
// Integration with Google Tag Manager
// Adds Google Tag Manager code to your website header for traffic monitoring and analytics collection.
function adsandmetas() {
    ?>
    <script async src="https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXXXXX"></script>
    <script>
	window.dataLayer = window.dataLayer || [];
	function gtag(){dataLayer.push(arguments);}
	gtag('js', new Date());
	gtag('config', 'G-XXXXXXXXXXX');
    </script>
    <?php
}

add_action( 'wp_head', 'adsandmetas' );
```

### Disabling RSS Feeds
This code disables all actions and filters related to RSS Feeds in WordPress, such as RDF, RSS, RSS2 and Atom feeds, preventing the site from generating feeds.

```php
// Disabling RSS Feeds
// This code disables all actions and filters related to RSS Feeds in WordPress, such as RDF, RSS, RSS2 and Atom feeds, preventing the site from generating feeds.

add_action( 'init', $af = function() {
    remove_action( 'do_feed_rdf', 'do_feed_rdf', 10 );
    remove_action( 'do_feed_rss', 'do_feed_rss', 10 );
    remove_action( 'do_feed_rss2', 'do_feed_rss2', 10 );
    remove_action( 'do_feed_atom', 'do_feed_atom', 10 );
    remove_action( 'wp_head', 'feed_links_extra', 3 );
    remove_action( 'wp_head', 'feed_links', 2 );
    remove_filter( 'the_content_feed', '_oembed_filter_feed_content' );
});
unset( $af );
```

### Where do I report bugs or rendering issues?

Just [open an issue][] on github, post your markdown code and describe the problem. You may also attach screenshots of the rendered HTML result to describe your problem.

[open an issue]: https://github.com/douglaswp/WP-Tweaks/issues

### How can I contribute to this library?

Check the [CONTRIBUTING.md](CONTRIBUTING.md) file for more info.


### Am I free to use this?

This library is open source and licensed under the [MIT License][]. This means that you can do whatever you want
with it as long as you mention my name and include the [license file][license]. Check the [license][] for details.

[MIT License]: http://opensource.org/licenses/MIT

[license]: https://github.com/douglaswp/WP-Tweaks/blob/master/LICENSE
