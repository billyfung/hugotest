Learning Go by porting my website to Hugo

Scaffolding
One of the first things I look at in a framework is scaffolding. Akin to visiting a new city, the infrastructure shows how everything is laid out and the ease of access. As a city grows, the infrastructure also gets more complex. With most problems, I approach from a first principles viewpoint, start as small as I can, and then add to the problem sample as requirements are found. 

Hugo starts with folders for archetypes, content, data, layouts, public, static, and themes. The configuration, or instructions for the scaffolding, lies within the `config.toml` file. 

One of the first things that I did was write a list of features that I'd like to have for the blog engine.
- code syntax formatting 
- math formatting and support
- colour coded code blocks if possible
- automated frontmatter creation for blog posts
- simple markdown structure 