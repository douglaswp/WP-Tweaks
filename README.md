
# WP-Tweaks - Conjunto 'hacks' para WordPress e WooCommerce

Este projeto contém diversas funções úteis para otimizar o comportamento do WordPress e WooCommerce.

Esses trechos de código devem ser inseridos no arquivo `functions.php` preferencialmente em um tema filho.

## WooCommerce

### Ordenação de Produtos pela Quantidade de Estoque

Esta classe organiza os produtos com base na quantidade em estoque, priorizando produtos disponíveis na exibição das listagens no WooCommerce (loja, categorias, tags de produtos).


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

	new DWP_Orderby_Stock_Status;# Conjunto te Plugin Personalizado para WordPress
	#

## Wordpress



### Conversão de Imagens JPEG e PNG para WebP no Upload

Essa função converte automaticamente imagens JPEG e PNG carregadas no WordPress para o formato WebP, otimizando o desempenho do site.

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

### Suporte para Upload de Arquivos SVG

Esta função adiciona suporte ao upload de arquivos SVG no WordPress, permitindo que SVGs sejam carregados diretamente na biblioteca de mídia.

	function add_svg_support( $mimes ) {
	    $mimes['svg'] = 'image/svg+xml';
	    return $mimes;
	}

	add_filter( 'upload_mimes', 'add_svg_support' );

### Integração com Google Tag Manager

Adiciona o código do Google Tag Manager no cabeçalho do site para monitoramento de tráfego e coleta de dados analíticos.

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


### Desativação de Feeds RSS

Este código desativa todas as ações e filtros relacionados aos Feeds RSS no WordPress, como os feeds RDF, RSS, RSS2 e Atom, impedindo que o site gere feeds.

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


## Contribuição

Caso tenha sugestões ou melhorias, sinta-se à vontade para enviar pull requests. Agradecemos pela sua contribuição!

## Licença

Este código é distribuído sob a licença MIT.
