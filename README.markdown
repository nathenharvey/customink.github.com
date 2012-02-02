CustomInk Technology Blog
=========================

## Installation

The CustomInk Technology Blog uses [Octopress](http://octopress.org/) which requires Ruby 1.9.2.

Install Ruby 1.9.2
    rvm install 1.9.2 && rvm use 1.9.2


Clone the repo
    git clone git@github.com:customink/blog.git


Install Gems
    bundle install


## Usage

Create a new post
    rake new_post["'Clever' Code: I will rub your nose in it"]

Open the generated markdown file
      
Add post categories to the YAML front matter  
    categories: [Expressive Code, Metaprogramming, Badness]

Write Stuff

Generate & Preview
    rake generate   # Generates posts and pages into the public directory
    rake watch      # Watches source/ and sass/ for changes and regenerates
    rake preview    # Watches, and mounts a webserver at http://localhost:4000

Publish
    TBD


## Resources
[Octopress Docs](http://octopress.org/docs)
[Markdown Docs](http://daringfireball.net/projects/markdown/)

