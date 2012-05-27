EngineerInk Blog
================

* Platform: Octopress/Jekyll
* URL: http://technology.customink.com


Installation
------------

The EngineerInk Blog uses [Octopress](http://octopress.org/) and is hosted on github pages.

### Dependencies

* Ruby 1.9.3
* Bundler

### Quick Start

```bash
git clone git@github.com:customink/customink.github.com.git blog
cd blog
bundle install
```

You might be prompted about the `rvmrc` file. Type `yes`.


Usage
-----

* **NEVER push to master**
* Update the blog the same way you would update code
* Posts in are `./source/_posts`
* Branch off `master` or work locally for longer running articles
* Always work off the `source` branch

### Creating a new Post

 1. Use the built-in Rake task to generate the necessary files:

  ```bash
  rake new_post["Taking Over the World: One custom T-shirt at a time"]
  ```

 2. Open the generated markdown file
 3. Add metadata to the YAML front matter ('author' should correspond to a key in _config.yml):

 ```yml
 author: tien-nguyen
 categories: [Evil, Muwahaha, Kittens]
 published: false # working draft, will not be published on generate
 ```

 4. Write content
 5. Generate & Preview

 ```bash
 rake preview # Watches and mounts a webserver at http://0.0.0.0:4000
 ```

### Publishing

```bash
rake generate && rake deploy
```

In a few moments, see your new article published on [the blog](http://technology.customink.com)!


Resources
---------

* [Octopress Docs](http://octopress.org/docs)
* [Markdown Docs](http://daringfireball.net/projects/markdown/)
* [Jekyll Docs](https://github.com/mojombo/jekyll/wiki/template-data)
* [Liquid Docs](https://github.com/Shopify/liquid/wiki/Liquid-for-Designers)