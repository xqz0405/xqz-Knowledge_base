---
tags:
  - Web前端
  - 综合领域
  - WordPress
  - CMS
date: 2026-05-14
status: 已完成
difficulty: 中等
---

# WordPress与CMS开发

## What — 是什么

> WordPress 是全球使用最广泛的开源内容管理系统（CMS），基于 PHP + MySQL 构建，从博客引擎发展为覆盖 43%+ 网站的通用建站平台，支持主题定制、插件扩展、REST API 无头架构等多种开发模式。

**核心概念：**

- **内容管理系统（CMS）**：提供后台管理界面，非技术用户可独立创建、编辑、发布内容，无需写代码
- **主题（Theme）**：控制网站外观和布局的模板系统，由 PHP 模板 + CSS + JS 组成，支持子主题继承定制
- **插件（Plugin）**：扩展 WordPress 功能的模块化组件，通过 Hook 机制注入逻辑，不改核心代码
- **Hook 机制**：WordPress 的扩展核心——Action（动作钩子）和 Filter（过滤器钩子），让插件能在特定时机执行代码
- **无头 WordPress（Headless）**：前端与后端解耦，WordPress 仅作内容管理后端，通过 REST API / GraphQL 为任意前端提供数据

**核心架构：**

- 技术栈：PHP 7.4+ / MySQL 5.7+ / MariaDB 10.3+ / Apache/Nginx
- 目录结构：`wp-content/`（用户内容）vs `wp-includes/` + `wp-admin/`（核心代码）
- 请求流程：URL → `.htaccess` 重写 → `index.php` → `wp-blog-header.php` → 数据库查询 → 主题模板渲染 → HTML 输出
- 数据模型：文章（post）、页面（page）、自定义文章类型（CPT）、分类法（taxonomy）、用户（user）、选项（option）

**关键特性：**

- 古腾堡区块编辑器（Gutenberg）：块级可视化编辑，基于 React
- REST API：开箱即用的完整 RESTful 接口，支持 JSON 格式 CRUD
- 多站点（Multisite）：一个 WordPress 安装管理多个子站点
- 自定义文章类型（CPT）+ 自定义字段（ACF）：灵活建模任意内容结构

## Why — 为什么

**适用场景：**

- 企业官网、品牌展示站——快速上线，主题丰富
- 博客与内容媒体——WordPress 本身就是博客引擎出身
- 电商网站——WooCommerce 插件提供完整电商功能
- 无头 CMS 后端——为 React/Vue/Next.js 前端提供内容 API
- 会员站与在线课程——MemberPress、LearnDash 等插件

**核心优势：全球最大的 CMS 生态**

WordPress 占据全球 43%+ 网站的市场份额，意味着：(1) 主题和插件市场极其丰富（6 万+免费插件）；(2) 开发者社区庞大，问题几乎都能找到解答；(3) 主机商普遍提供一键安装和优化；(4) SEO 生态成熟（Yoast SEO 等插件）。对于内容驱动的网站，WordPress 是成本最低、速度最快的方案。

**对比同类 CMS：**

| 维度 | WordPress | Strapi | Ghost | Drupal |
|------|-----------|--------|-------|--------|
| 定位 | 通用 CMS + 无头能力 | 纯无头 CMS | 博客/ Newsletter CMS | 企业级 CMS |
| 语言 | PHP | Node.js | Node.js | PHP |
| 前端自由度 | 传统主题 / REST API | 完全自由 | 有限（Handlebars） | 传统主题 / REST API |
| 插件生态 | 6 万+ 免费插件 | 少量插件 | 少量插件 | 4 万+ 模块 |
| 学习曲线 | 低（后台直观） | 中（需前端能力） | 低 | 高（概念复杂） |
| 内容建模 | CPT + ACF（灵活但手动） | 内容类型构建器（可视化） | 内置（简单） | 节点 + 字段（强大） |
| 无头模式 | REST API 原生支持 | 天生无头 | Admin API | REST API 原生支持 |
| 适用规模 | 小到大型站 | 中小型 | 小到中型 | 大型企业站 |

**优缺点：**

- ✅ 优点：
  - 生态极其成熟，主题/插件/教程海量
  - 后台管理界面友好，非技术人员可独立运营
  - REST API 支持无头架构，前后端解耦
  - SEO 优化方案完善（Yoast、Rank Math）
  - WooCommerce 电商生态完整
  - 全球主机商支持，部署运维成本低
- ❌ 缺点：
  - PHP 技术栈对前端开发者不友好
  - 插件质量参差不齐，安全漏洞频发
  - 传统主题开发体验差（PHP 混写 HTML）
  - 性能瓶颈：动态查询多，需大量缓存优化
  - 核心代码历史包袱重，架构不够现代

## How — 怎么用

### 1. 安装与项目结构

```bash
# 方式一：主机一键安装（最常见）
# cPanel / 宝塔面板 / WP Engine 都提供一键安装

# 方式二：手动安装
# 1. 下载 WordPress
curl -O https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz

# 2. 创建数据库
mysql -u root -p
CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'wp_user'@'localhost' IDENTIFIED BY 'secure_password';
GRANT ALL ON wordpress.* TO 'wp_user'@'localhost';
FLUSH PRIVILEGES;

# 3. 配置 wp-config.php
cp wordpress/wp-config-sample.php wordpress/wp-config.php
# 编辑 DB_NAME / DB_USER / DB_PASSWORD / DB_HOST
# 修改认证密钥：https://api.wordpress.org/secret-key/1.1/salt/

# 4. 配置 Nginx
```

```nginx
# /etc/nginx/conf.d/wordpress.conf
server {
    listen 80;
    server_name example.com;
    root /var/www/wordpress;
    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        fastcgi_pass unix:/run/php/php8.2-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    # 禁止访问敏感文件
    location ~ /(wp-config\.php|readme\.html|license\.txt) {
        deny all;
    }

    # 静态资源缓存
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff2)$ {
        expires 30d;
        add_header Cache-Control "public, immutable";
    }
}
```

**WordPress 目录结构：**

```
wordpress/
├── wp-admin/              # 后台管理界面（不修改）
├── wp-includes/           # 核心代码（不修改）
├── wp-content/            # 用户内容（所有定制在这）
│   ├── themes/            # 主题目录
│   │   ├── twentytwentyfour/
│   │   └── my-theme/      # 自定义主题
│   ├── plugins/           # 插件目录
│   │   ├── akismet/
│   │   └── my-plugin/     # 自定义插件
│   ├── uploads/           # 上传的媒体文件
│   └── mu-plugins/        # 必须启用插件（不可禁用）
├── wp-config.php          # 配置文件（数据库、密钥、调试）
├── wp-settings.php        # 引导文件
└── index.php              # 入口文件
```

### 2. 主题开发

**传统主题 vs 区块主题：**

WordPress 有两种主题开发模式：传统主题（PHP 模板 + 模板标签）和区块主题（FSE 全站编辑 + HTML 模板）。传统主题是主流，区块主题是未来方向。

**传统主题开发：**

```php
<?php
// style.css — 主题声明（必需）
/*
Theme Name: My Theme
Theme URI: https://example.com
Author: xqz
Description: 自定义 WordPress 主题
Version: 1.0
Requires at least: 6.0
Tested up to: 6.5
Requires PHP: 8.0
License: GPLv2
Text Domain: my-theme
*/
```

```php
<?php
// functions.php — 主题功能注册
// 这个文件在每次页面加载时执行，类似于主题的"入口文件"

// 注册样式和脚本
function my_theme_enqueue_assets() {
    wp_enqueue_style(
        'my-theme-style',
        get_template_directory_uri() . '/style.css',
        [],
        wp_get_theme()->get('Version')
    );

    wp_enqueue_script(
        'my-theme-script',
        get_template_directory_uri() . '/assets/js/main.js',
        [],
        wp_get_theme()->get('Version'),
        true  // 在 footer 加载
    );
}
add_action('wp_enqueue_scripts', 'my_theme_enqueue_assets');

// 注册导航菜单
register_nav_menus([
    'primary'   => '主导航菜单',
    'footer'    => '底部菜单',
]);

// 注册侧边栏
register_sidebar([
    'name'          => '文章侧边栏',
    'id'            => 'sidebar-post',
    'before_widget' => '<div class="widget %2$s">',
    'after_widget'  => '</div>',
    'before_title'  => '<h3 class="widget-title">',
    'after_title'   => '</h3>',
]);

// 启用主题功能
add_theme_support('title-tag');        // 动态 <title>
add_theme_support('post-thumbnails');  // 特色图片
add_theme_support('html5', [           // HTML5 标记
    'search-form', 'comment-form', 'gallery', 'caption',
]);
add_theme_support('custom-logo', [     # 自定义 Logo
    'height'      => 100,
    'width'       => 350,
    'flex-height' => true,
]);

// 自定义图片尺寸
add_image_size('card-thumbnail', 400, 250, true);
add_image_size('hero-banner', 1920, 600, true);
```

**模板层级（Template Hierarchy）：**

WordPress 有一套模板匹配优先级，决定不同页面使用哪个模板文件：

```
页面类型          模板匹配优先级（从高到低）
────────────────────────────────────────────────────────
首页              front-page.php → home.php → index.php
单篇文章          single-{post_type}.php → single.php → index.php
页面              page-{slug}.php → page-{id}.php → page.php → index.php
分类归档          category-{slug}.php → category-{id}.php → category.php → archive.php → index.php
标签归档          tag-{slug}.php → tag-{id}.php → tag.php → archive.php → index.php
搜索结果          search.php → index.php
404 页面          404.php → index.php
```

```php
<?php
// single.php — 单篇文章模板
get_header();  // 引入 header.php
?>

<main class="content">
    <?php if (have_posts()) : while (have_posts()) : the_post(); ?>
        <article class="post">
            <h1 class="post-title"><?php the_title(); ?></h1>

            <div class="post-meta">
                <time><?php echo get_the_date(); ?></time>
                <span><?php the_author(); ?></span>
                <?php the_category(', '); ?>
            </div>

            <?php if (has_post_thumbnail()) : ?>
                <div class="post-featured-image">
                    <?php the_post_thumbnail('hero-banner'); ?>
                </div>
            <?php endif; ?>

            <div class="post-content">
                <?php the_content(); ?>
            </div>

            <?php
            // 分页链接（文章内 <!--nextpage--> 分页）
            wp_link_pages([
                'before' => '<div class="page-links">',
                'after'  => '</div>',
            ]);
            ?>

            <?php the_tags('<div class="tags">', '', '</div>'); ?>
        </article>

        <nav class="post-navigation">
            <?php
            the_post_navigation([
                'prev_text' => '&larr; %title',
                'next_text' => '%title &rarr;',
            ]);
            ?>
        </nav>

        <?php comments_template(); ?>
    <?php endwhile; endif; ?>
</main>

<?php get_sidebar(); ?>
<?php get_footer(); ?>
```

**子主题（Child Theme）：**

```php
<?php
// 子主题 style.css
/*
Theme Name: My Theme Child
Template: my-theme  // 父主题目录名
Version: 1.0
*/
```

```php
<?php
// 子主题 functions.php
// 子主题的 functions.php 在父主题之前加载（不是覆盖！）
add_action('wp_enqueue_scripts', function() {
    // 先加载父主题样式
    wp_enqueue_style('parent-style', get_template_directory_uri() . '/style.css');
    // 再加载子主题样式（覆盖父主题）
    wp_enqueue_style('child-style', get_stylesheet_directory_uri() . '/style.css',
        ['parent-style'],
        wp_get_theme()->get('Version')
    );
});

// 覆盖父主题函数（通过优先级）
function my_theme_custom_excerpt_length($length) {
    return 30;  // 摘要字数改为 30
}
add_filter('excerpt_length', 'my_theme_custom_excerpt_length', 999);
```

### 3. Hook 机制

Hook 是 WordPress 扩展的基石——几乎所有定制都通过 Action 和 Filter 实现，无需修改核心代码。

**Action（动作钩子）：**

```php
<?php
// Action：在特定事件触发时执行代码，不返回值

// 在 <head> 中添加自定义 meta
add_action('wp_head', function() {
    if (is_single()) {
        echo '<meta property="og:type" content="article">' . "\n";
    }
});

// 在文章保存后执行操作
add_action('save_post', function($post_id, $post, $update) {
    // 避免自动保存触发
    if (defined('DOING_AUTOSAVE') && DOING_AUTOSAVE) return;
    // 避免修订版本触发
    if (wp_is_post_revision($post_id)) return;

    // 清除缓存
    wp_cache_delete('recent_posts', 'my_theme');
}, 10, 3);  // 优先级 10，接受 3 个参数

// 自定义 Action：插件间通信
do_action('my_theme_before_content', $post_id);  // 触发
add_action('my_theme_before_content', function($post_id) {
    // 其他插件可以在此注入逻辑
}, 10, 1);  // 监听
```

**Filter（过滤器钩子）：**

```php
<?php
// Filter：拦截和修改数据，必须返回值

// 修改文章内容中的图片，添加 lazy loading
add_filter('the_content', function($content) {
    return str_replace('<img ', '<img loading="lazy" ', $content);
});

// 修改摘要更多链接
add_filter('excerpt_more', function() {
    return '...<a href="' . get_permalink() . '">阅读全文</a>';
});

// 自定义 REST API 响应字段
add_filter('rest_prepare_post', function($response, $post, $request) {
    $response->data['featured_image_url'] = get_the_post_thumbnail_url($post->ID, 'full');
    $response->data['author_name'] = get_the_author_meta('display_name', $post->post_author);
    return $response;
}, 10, 3);

// 修改查询条件
add_filter('pre_get_posts', function($query) {
    if (!is_admin() && $query->is_main_query() && is_home()) {
        $query->set('posts_per_page', 12);
        $query->set('post_type', ['post', 'project']);
    }
    return $query;
});
```

**常用 Hook 速查：**

| Hook 类型 | 名称 | 触发时机 |
|-----------|------|----------|
| Action | `wp_enqueue_scripts` | 加载前端样式/脚本 |
| Action | `admin_enqueue_scripts` | 加载后台样式/脚本 |
| Action | `init` | WordPress 初始化完成 |
| Action | `wp_head` | 输出 `</head>` 前 |
| Action | `wp_footer` | 输出 `</body>` 前 |
| Action | `save_post` | 文章保存后 |
| Action | `wp_login` | 用户登录后 |
| Filter | `the_content` | 文章内容输出前 |
| Filter | `the_title` | 文章标题输出前 |
| Filter | `excerpt_length` | 摘要字数 |
| Filter | `rest_prepare_post` | REST API 文章响应 |
| Filter | `nav_menu_item_title` | 菜单项标题 |

### 4. 自定义文章类型（CPT）与 ACF

```php
<?php
// 注册自定义文章类型
add_action('init', function() {
    register_post_type('project', [
        'labels' => [
            'name'               => '项目',
            'singular_name'      => '项目',
            'add_new_item'       => '新建项目',
            'edit_item'          => '编辑项目',
            'all_items'          => '所有项目',
            'search_items'       => '搜索项目',
        ],
        'public'       => true,
        'has_archive'  => true,
        'menu_icon'    => 'dashicons-portfolio',
        'supports'     => ['title', 'editor', 'thumbnail', 'excerpt', 'custom-fields'],
        'rewrite'      => ['slug' => 'projects'],
        'show_in_rest' => true,  // 启用 REST API 支持（Gutenberg 需要）
    ]);

    // 注册自定义分类法
    register_taxonomy('project_category', 'project', [
        'labels' => [
            'name'          => '项目分类',
            'singular_name' => '分类',
        ],
        'hierarchical'      => true,   // 类似分类（true）vs 类似标签（false）
        'public'            => true,
        'show_in_rest'      => true,
        'rewrite'           => ['slug' => 'project-category'],
    ]);
});
```

**ACF（Advanced Custom Fields）— 自定义字段：**

ACF 是 WordPress 最常用的插件之一，为文章、页面、CPT 添加自定义字段，替代原生的自定义字段界面。

```php
<?php
// 方式一：PHP 注册字段组（推荐，版本控制友好）
if (function_exists('acf_add_local_field_group')) {
    acf_add_local_field_group([
        'key' => 'group_project',
        'title' => '项目信息',
        'fields' => [
            [
                'key'   => 'field_project_url',
                'label' => '项目链接',
                'name'  => 'project_url',
                'type'  => 'url',
            ],
            [
                'key'   => 'field_project_tech',
                'label' => '技术栈',
                'name'  => 'project_tech',
                'type'  => 'taxonomy',
                'taxonomy' => 'post_tag',
                'field_type' => 'multi_select',
            ],
            [
                'key'   => 'field_project_gallery',
                'label' => '项目截图',
                'name'  => 'project_gallery',
                'type'  => 'gallery',
                'return_format' => 'url',
            ],
        ],
        'location' => [
            [
                [
                    'param'    => 'post_type',
                    'operator' => '==',
                    'value'    => 'project',
                ],
            ],
        ],
    ]);
}
```

```php
<?php
// 在模板中使用 ACF 字段
$project_url   = get_field('project_url');
$project_tech  = get_field('project_tech');
$gallery       = get_field('project_gallery');

if ($project_url) {
    echo '<a href="' . esc_url($project_url) . '" target="_blank">访问项目</a>';
}

if ($gallery) {
    echo '<div class="gallery">';
    foreach ($gallery as $image_url) {
        echo '<img src="' . esc_url($image_url) . '" loading="lazy">';
    }
    echo '</div>';
}
```

### 5. 插件开发

```php
<?php
/**
 * Plugin Name: My Custom Plugin
 * Plugin URI: https://example.com
 * Description: 自定义功能插件示例
 * Version: 1.0.0
 * Author: xqz
 * License: GPLv2
 * Text Domain: my-plugin
 */

// 安全检查：防止直接访问
defined('ABSPATH') || exit;

// 插件常量
define('MY_PLUGIN_VERSION', '1.0.0');
define('MY_PLUGIN_PATH', plugin_dir_path(__FILE__));
define('MY_PLUGIN_URL', plugin_dir_url(__FILE__));

// 注册短代码
add_shortcode('recent_posts', function($atts) {
    $atts = shortcode_atts([
        'count'  => 5,
        'category' => '',
    ], $atts, 'recent_posts');

    $args = [
        'post_type'      => 'post',
        'posts_per_page' => intval($atts['count']),
        'post_status'    => 'publish',
    ];

    if ($atts['category']) {
        $args['category_name'] = sanitize_text_field($atts['category']);
    }

    $query = new WP_Query($args);

    if (!$query->have_posts()) return '<p>暂无文章</p>';

    $output = '<ul class="recent-posts">';
    while ($query->have_posts()) {
        $query->the_post();
        $output .= sprintf(
            '<li><a href="%s">%s</a> <time>%s</time></li>',
            get_permalink(),
            get_the_title(),
            get_the_date()
        );
    }
    $output .= '</ul>';

    wp_reset_postdata();  // 恢复全局 $post
    return $output;
});
// 使用：[recent_posts count="3" category="tech"]

// 自定义 REST API 端点
add_action('rest_api_init', function() {
    register_rest_route('my-plugin/v1', '/stats', [
        'methods'             => 'GET',
        'callback'            => 'my_plugin_get_stats',
        'permission_callback' => function() {
            return current_user_can('read');
        },
    ]);
});

function my_plugin_get_stats($request) {
    return new WP_REST_Response([
        'total_posts'  => wp_count_posts()->publish,
        'total_users'  => count_users()['total_users'],
        'total_comments' => wp_count_comments()->approved,
    ], 200);
}
```

### 6. REST API 与无头 WordPress

WordPress 原生提供完整的 REST API，是无头架构的数据基础。

**API 端点概览：**

```
GET  /wp-json/wp/v2/posts          # 文章列表
GET  /wp-json/wp/v2/posts/{id}     # 单篇文章
POST /wp-json/wp/v2/posts          # 创建文章（需认证）
PUT  /wp-json/wp/v2/posts/{id}     # 更新文章
DEL  /wp-json/wp/v2/posts/{id}     # 删除文章
GET  /wp-json/wp/v2/pages          # 页面列表
GET  /wp-json/wp/v2/categories     # 分类列表
GET  /wp-json/wp/v2/tags           # 标签列表
GET  /wp-json/wp/v2/media          # 媒体列表
GET  /wp-json/wp/v2/users          # 用户列表
GET  /wp-json/wp/v2/comments       # 评论列表
```

**前端消费 WordPress API：**

```ts
// lib/wordpress.ts — Next.js 无头 WordPress 示例
const API_URL = process.env.WP_API_URL!;

interface WPPost {
  id: number;
  title: { rendered: string };
  content: { rendered: string };
  excerpt: { rendered: string };
  date: string;
  slug: string;
  _embedded?: {
    'wp:featuredmedia'?: [{ source_url: string; alt_text: string }];
    'wp:term'?: [[{ id: number; name: string; slug: string }]];
  };
}

async function fetchAPI(path: string, options?: RequestInit) {
  const res = await fetch(`${API_URL}/wp-json${path}`, {
    ...options,
    headers: {
      'Content-Type': 'application/json',
      ...options?.headers,
    },
    next: { revalidate: 3600 },  // ISR: 每小时重新验证
  });

  if (!res.ok) throw new Error(`WordPress API error: ${res.status}`);
  return res.json();
}

// 获取文章列表
export async function getPosts(page = 1, perPage = 10): Promise<{
  posts: WPPost[];
  totalPages: number;
}> {
  const data = await fetchAPI(
    `/wp/v2/posts?page=${page}&per_page=${perPage}&_embed`
  );

  // 注意：fetchAPI 返回的是数组，总页数在响应头中
  // Next.js 中需单独获取
  return {
    posts: data,
    totalPages: 1, // 从 WP-Total 响应头获取
  };
}

// 获取单篇文章
export async function getPostBySlug(slug: string): Promise<WPPost> {
  const posts = await fetchAPI(`/wp/v2/posts?slug=${slug}&_embed`);
  return posts[0];
}
```

```tsx
// app/blog/[slug]/page.tsx — Next.js 页面组件
import { getPostBySlug } from '@/lib/wordpress';
import { notFound } from 'next/navigation';

export default async function BlogPost({ params }: { params: { slug: string } }) {
  const post = await getPostBySlug(params.slug);

  if (!post) notFound();

  const featuredImage = post._embedded?.['wp:featuredmedia']?.[0];
  const categories = post._embedded?.['wp:term']?.[0] || [];

  return (
    <article>
      {featuredImage && (
        <img src={featuredImage.source_url} alt={featuredImage.alt_text} />
      )}
      <h1 dangerouslySetInnerHTML={{ __html: post.title.rendered }} />
      <div className="meta">
        <time>{new Date(post.date).toLocaleDateString('zh-CN')}</time>
        {categories.map(cat => (
          <span key={cat.id}>{cat.name}</span>
        ))}
      </div>
      <div dangerouslySetInnerHTML={{ __html: post.content.rendered }} />
    </article>
  );
}
```

**认证方式：**

```ts
// 应用密码认证（WordPress 5.6+ 原生支持）
// 后台 → 用户 → 编辑 → 应用密码 → 生成新密码

const authHeader = 'Basic ' + Buffer.from('username:xxxx xxxx xxxx xxxx').toString('base64');

// 创建文章
await fetch(`${API_URL}/wp-json/wp/v2/posts`, {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': authHeader,
  },
  body: JSON.stringify({
    title: '新文章标题',
    content: '文章内容...',
    status: 'draft',
    categories: [1, 2],
    tags: [3, 4],
  }),
});
```

### 7. 性能优化

WordPress 性能问题主要来自：数据库查询多、PHP 动态渲染慢、插件加载重。优化需多管齐下。

**缓存策略：**

```php
<?php
// wp-config.php — 启用对象缓存和页面缓存

// Redis 对象缓存（安装 Redis Object Cache 插件）
define('WP_REDIS_HOST', '127.0.0.1');
define('WP_REDIS_PORT', 6379);
define('WP_REDIS_DATABASE', 0);

// 禁用不需要的功能减少查询
define('WP_POST_REVISIONS', 5);        // 限制修订版本数
define('EMPTY_TRASH_DAYS', 30);         // 回收站 30 天清空
define('DISALLOW_FILE_EDIT', true);     // 禁止后台编辑主题/插件
define('WP_MEMORY_LIMIT', '256M');      // PHP 内存限制
```

```php
<?php
// 对象缓存 Transients API
function my_theme_get_recent_posts() {
    $cache_key = 'recent_posts_' . get_locale();
    $cached = get_transient($cache_key);

    if ($cached !== false) return $cached;

    $posts = get_posts([
        'numberposts'  => 10,
        'post_status'  => 'publish',
    ]);

    set_transient($cache_key, $posts, HOUR_IN_SECONDS);  // 缓存 1 小时
    return $posts;
}

// 文章更新时清除缓存
add_action('save_post', function($post_id) {
    if (wp_is_post_revision($post_id)) return;
    delete_transient('recent_posts_' . get_locale());
});
```

**前端优化：**

```php
<?php
// functions.php — 前端性能优化

// 移除 WordPress 默认加载的不必要资源
add_action('wp_enqueue_scripts', function() {
    // 移除 Emoji 脚本
    remove_action('wp_head', 'print_emoji_detection_script', 7);
    remove_action('wp_print_styles', 'print_emoji_styles');

    // 移除 WordPress 版本号
    remove_action('wp_head', 'wp_generator');

    // 移除 REST API 链接（无头模式下不需要）
    remove_action('wp_head', 'rest_output_link_wp_head');

    // 移除 jQuery Migrate
    wp_deregister_script('jquery');
}, 1);

// 预加载关键资源
add_action('wp_head', function() {
    echo '<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>' . "\n";
    echo '<link rel="dns-prefetch" href="//cdn.example.com">' . "\n";
}, 1);

// 延迟加载非关键脚本
add_filter('script_loader_tag', function($tag, $handle) {
    $defer_handles = ['my-theme-script', 'comment-reply'];
    if (in_array($handle, $defer_handles)) {
        return str_replace(' src', ' defer src', $tag);
    }
    return $tag;
}, 10, 2);
```

**静态化与 CDN：**

| 方案 | 原理 | 适用场景 |
|------|------|----------|
| WP Super Cache | 生成静态 HTML 文件，Nginx 直接返回 | 访问量大的内容站 |
| Redis Object Cache | 缓存数据库查询结果 | 动态内容较多 |
| Cloudflare CDN | 边缘缓存 + DDoS 防护 | 全球用户 |
| Static Site Generator | WP → 静态 HTML 导出 | 安全要求高 |
| Varnish | 反向代理缓存 | 服务器级缓存 |

### 8. 安全加固

WordPress 是最常被攻击的 CMS，安全加固必不可少。

```php
<?php
// wp-config.php — 安全配置

// 修改数据库表前缀（安装时设置，默认 wp_ 易被 SQL 注入）
$table_prefix = 'xqz_wp_';

// 强制 SSL 后台
define('FORCE_SSL_ADMIN', true);

// 禁止文件编辑
define('DISALLOW_FILE_EDIT', true);

// 禁止文件修改
define('DISALLOW_FILE_MODS', true);

// 自动更新
define('WP_AUTO_UPDATE_CORE', true);       // 核心自动更新
add_filter('auto_update_plugin', '__return_true');  // 插件自动更新
add_filter('auto_update_theme', '__return_true');   // 主题自动更新
```

```php
<?php
// functions.php — 安全加固

// 移除 WordPress 版本号（防止攻击者针对已知漏洞）
add_filter('the_generator', '__return_empty_string');

// 移除登录错误提示（防止枚举用户名）
add_filter('login_errors', function() {
    return '登录信息错误，请重试。';
});

// 限制登录尝试
add_action('wp_login_failed', function($username) {
    $ip = $_SERVER['REMOTE_ADDR'];
    $attempts = (int) get_transient('login_attempts_' . $ip);

    if ($attempts >= 5) {
        wp_die('登录尝试过多，请 30 分钟后重试。');
    }

    set_transient('login_attempts_' . $ip, $attempts + 1, 30 * MINUTE_IN_SECONDS);
});

// 登录成功时清除计数
add_action('wp_login', function($username) {
    delete_transient('login_attempts_' . $_SERVER['REMOTE_ADDR']);
});

// REST API 安全：非登录用户禁用用户端点
add_filter('rest_endpoints', function($endpoints) {
    if (!is_user_logged_in()) {
        unset($endpoints['/wp/v2/users']);
        unset($endpoints['/wp/v2/users/(?P<id>[\d]+)']);
    }
    return $endpoints;
});
```

**安全检查清单：**

| 项目 | 措施 |
|------|------|
| 代码注入 | 禁止后台编辑文件，所有输出 `esc_html()` / `esc_attr()` / `esc_url()` |
| SQL 注入 | 使用 `$wpdb->prepare()` 而非字符串拼接 |
| XSS | 输出时转义，`the_content()` 安全但自定义字段需手动转义 |
| CSRF | 表单使用 `wp_nonce_field()` + `wp_verify_nonce()` |
| 暴力破解 | 限制登录尝试 + 验证码 + 双因素认证 |
| 文件上传 | 限制上传类型和大小，上传目录禁止执行 PHP |
| 信息泄露 | 移除版本号、移除 README、移除错误详情 |

---

## 常见问题

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 白屏死机（WSOD） | PHP 致命错误但 `WP_DEBUG` 关闭 | `wp-config.php` 启用 `define('WP_DEBUG', true)` 查看错误 |
| 文章更新后前端没变化 | 页面缓存或 CDN 缓存未清除 | 清除缓存插件缓存 + CDN 缓存 + 浏览器缓存 |
| REST API 返回 401 | 认证信息缺失或应用密码过期 | 检查 Authorization 头，重新生成应用密码 |
| ACF 字段在 REST API 中不显示 | ACF 默认不暴露到 REST | 安装 ACF to REST API 插件或在字段组设置中启用 REST |
| 上传大文件失败 | PHP `upload_max_filesize` 限制 | 修改 `php.ini` 的 `upload_max_filesize` 和 `post_max_size` |
| 插件冲突导致崩溃 | 多个插件 Hook 同一位置冲突 | 逐个禁用插件定位冲突源，使用健康检查插件 |
| 数据库连接错误 | `wp-config.php` 数据库配置错误或数据库服务未启动 | 检查 DB_NAME/DB_USER/DB_PASSWORD/DB_HOST，确认 MySQL 运行 |
| 固定链接 404 | Nginx/Apache 未配置 URL 重写 | 检查 `.htaccess` 或 Nginx `try_files` 配置 |

---

## 面试题

**Q1: WordPress 的 Hook 机制是什么？Action 和 Filter 有什么区别？**

> Hook 机制是 WordPress 的扩展核心，允许开发者在不修改核心代码的情况下注入自定义逻辑。Action（动作钩子）在特定事件触发时执行代码，不返回值，如 `save_post` 在文章保存后触发；Filter（过滤器钩子）拦截和修改数据，必须返回修改后的值，如 `the_content` 在文章内容输出前拦截修改。区别：(1) Action 执行副作用（发邮件、清缓存），Filter 修改数据（改内容、改查询）；(2) Filter 回调必须有返回值，Action 不需要；(3) Filter 链式传递数据（上一个 Filter 的输出是下一个的输入），Action 顺序执行互不影响。

**Q2: 什么是无头 WordPress（Headless WordPress）？与传统 WordPress 有什么区别？**

> 无头 WordPress 将 WordPress 仅作为内容管理后端，通过 REST API 或 GraphQL 为独立的前端应用（Next.js、Nuxt.js、Astro 等）提供数据。传统 WordPress 中，主题模板直接在服务端渲染 HTML 输出；无头模式中，WordPress 只管内容 CRUD，前端完全独立。优势：(1) 前端技术栈自由（React/Vue/Svelte）；(2) 性能更好（SSG/ISR vs PHP 动态渲染）；(3) 安全性更高（WordPress 不直接暴露在前端）；(4) 更好的开发体验（前端用现代工具链）。劣势：(1) 需要额外开发和维护前端；(2) 预览/实时编辑体验不如传统模式；(3) 部署架构更复杂。

**Q3: WordPress 模板层级（Template Hierarchy）的工作原理是什么？**

> WordPress 根据请求的 URL 判断页面类型（首页、单篇文章、分类归档等），然后按优先级从高到低查找匹配的模板文件。例如访问单篇文章，WordPress 依次查找 `single-{post_type}.php` → `single.php` → `index.php`，找到第一个存在的文件就使用它渲染。这套层级让开发者可以精确控制不同页面的模板——用 `single-book.php` 定制书籍类型的文章页，用 `category-tech.php` 定制技术分类的归档页，而通用情况用 `single.php` 或 `index.php` 兜底。子主题中的模板文件优先级高于父主题，这是子主题覆盖父主题模板的机制。

**Q4: 如何优化 WordPress 站点的性能？**

> 多层面优化：(1) **缓存层**——Redis 对象缓存减少数据库查询，WP Super Cache 生成静态 HTML，Varnish 反向代理缓存，CDN 边缘缓存；(2) **数据库优化**——限制修订版本数、定期清理过期 transient、优化 SQL 查询（避免 `get_posts` 在循环中嵌套）、添加数据库索引；(3) **前端优化**——移除不必要的默认资源（Emoji、jQuery Migrate）、图片懒加载、CSS/JS 压缩合并、关键 CSS 内联、字体预加载；(4) **服务器优化**——PHP 8.x + OPcache、Nginx 静态资源缓存头、HTTP/2 推送；(5) **架构优化**——无头模式 + SSG/ISR 消除 PHP 渲染开销、分离静态资源到 CDN、读写分离。核心思路：WordPress 性能瓶颈在数据库查询和 PHP 动态渲染，缓存和静态化是最有效的手段。

**Q5: ACF 插件的作用是什么？为什么比 WordPress 原生自定义字段更好？**

> ACF（Advanced Custom Fields）为 WordPress 提供可视化的自定义字段管理。原生自定义字段只有文本输入框，无验证、无关联、界面简陋。ACF 提供 30+ 字段类型（图片、文件、关联、 repeater、flexible content、gallery 等），每种类型有专用 UI；支持字段组（Field Group）按条件（文章类型、页面模板、分类等）显示；支持字段验证和默认值；提供 PHP 注册 API 支持版本控制。Repeater 字段解决了一对多关系（如团队成员列表），Flexible Content 字段解决了页面模块化（如页面编辑器），Gallery 字段解决了图片集管理。ACF 的 `get_field()` / `the_field()` API 简洁好用，原生需要 `get_post_meta()` 且返回原始值需自行处理。

**Q6: WordPress REST API 的认证方式有哪些？各自适用什么场景？**

> (1) **Cookie 认证**——WordPress 默认方式，登录用户在后台操作时自动带 Cookie，仅适用于同域 WordPress 主题内的 AJAX 请求；(2) **应用密码（Application Passwords）**——WordPress 5.6+ 原生支持，在用户设置中生成，通过 HTTP Basic Auth 传递，适用于服务端到服务端的 API 调用（如 CI/CD 发布文章）；(3) **JWT 认证**——通过 JWT Authentication 插件实现，适用于 SPA/移动端应用，前端获取 token 后在请求头携带 `Authorization: Bearer {token}`；(4) **OAuth 1.0a**——通过 WP REST API - OAuth 1.0a 插件实现，适用于第三方应用集成（如 IFTTT、Zapier）。推荐：服务端调用用应用密码，前端 SPA 用 JWT，第三方集成用 OAuth。

**Q7: 如何安全地开发 WordPress 插件？需要防范哪些安全风险？**

> (1) **SQL 注入**——使用 `$wpdb->prepare()` 参数化查询，禁止字符串拼接 SQL；(2) **XSS**——输出时转义：`esc_html()`（HTML 文本）、`esc_attr()`（HTML 属性）、`esc_url()`（URL）、`wp_kses()`（允许部分 HTML）；(3) **CSRF**——表单使用 `wp_nonce_field()` 生成令牌，处理时 `wp_verify_nonce()` 验证；(4) **文件包含**——不使用用户输入作为 `include`/`require` 的路径；(5) **权限检查**——使用 `current_user_can()` 检查用户权限，REST API 端点设置 `permission_callback`；(6) **数据验证**——输入时用 `sanitize_text_field()` / `intval()` 清理，输出时转义；(7) **直接访问防护**——文件开头加 `defined('ABSPATH') || exit;`；(8) **文件上传**——检查文件类型（`wp_check_filetype()`）、限制大小、上传目录禁止执行 PHP。

**Q8: WordPress 的自定义文章类型（CPT）解决了什么问题？如何注册和使用？**

> CPT 解决了 WordPress 只有"文章"和"页面"两种内容类型的局限。现实网站需要各种内容结构：产品、项目、课程、案例、评价等，每种类型有不同的字段和展示方式。通过 `register_post_type()` 注册 CPT，可以定义独立的标签、URL 结构、后台图标、支持的功能（特色图片、摘要、评论等）、是否显示在 REST API 中。配合 `register_taxonomy()` 注册自定义分类法，可以建立完整的自定义内容体系。CPT 与 ACF 组合是 WordPress 内容建模的标准方案：CPT 定义内容类型框架，ACF 添加具体字段。注册时必须挂载到 `init` Action 上，`show_in_rest => true` 是开启 REST API 和 Gutenberg 编辑器支持的关键参数。

---

**相关链接：** [[Astro与内容站框架]] [[Next.js与SSR]] [[CI与CD与部署]] [[Web安全XSS与CSRF]] [[前端性能优化]]
