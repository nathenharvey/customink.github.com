EngineerInk Blog
=========================

## Installation

The EngineerInk Blog uses [Octopress](http://octopress.org/) which requires Ruby 1.9.2.

1\. Install Ruby 1.9.2

    rvm install 1.9.2 && rvm use 1.9.2


2\. Clone the repo

    git clone git@github.com:customink/blog.git


3\. Install Gems

    bundle install


## Usage

1\. Note:

* Update the blog in the same way you would update code
* Merge into master completed posts ready to be published in the next push
* Branch off of master or work locally for long running articles

2\. Create a new post

    rake new_post["Taking Over the World: One custom T-shirt at a time"]

3\. Open the generated markdown file
      
4\. Add post categories to the YAML front matter  

    author: Tien Nguyen
    categories: [Evil, Muwahaha, Kittens]
    published: false # working draft, will not be published on generate

5\. Write Stuff

6\. Generate & Preview

    rake generate   # Generates posts and pages into the public directory
    rake watch      # Watches source/ and sass/ for changes and regenerates
    rake preview    # Watches, and mounts a webserver at http://localhost:4000

7\. Publish

    TBD


## Resources
* [Octopress Docs](http://octopress.org/docs)
* [Markdown Docs](http://daringfireball.net/projects/markdown/)

