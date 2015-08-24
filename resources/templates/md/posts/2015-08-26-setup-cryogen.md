{:title "How to setup your own blog on Github using Cryogen"
 :layout :post
 :tags ["github"]}

For quite sometime I have been looking for a nice way to setup my blog. I have looked at various options but wasn't happy with any of them.

My main criteria was:
- I should be able to edit it within Emacs/Another nice Markdown Editor.
- It should be markdown based so I don't face weird editor issues.
- Code highlighting should work for Clojure.
- It should work on mobile/desktop nicely.
- Tags should be available.
- RSS feeds should work.
- Preferably free and scales well.
- Have some control on the setup but it should just work.
- Should have an option to add comments.
- It would be awesome if I can hack it if I need to.
- I should be able to point it to my custom domain.
- It should be under version control.

And I think finally [Cryogen](http://cryogenweb.org/)/Github/Emacs/[MacDown](macdown.uranusjr.com) has nailed it for me.

Even though the documentation is fairly good, I wasn't clear how to setup blog and repo together. I'll just assume you want to setup it as [username.github.io](username.github.io).

Before you move on, make sure you have checked out the [actual documentation](http://cryogenweb.org/docs/getting-started.html) to get an idea of what we are going to do and if this setup is suitable for you or not.

- Go to github and create one repo called `cryogen-blog`.
- Open up commandline, switch to your projects directory and run the following:

```bash
$ lein new cryogen my-blog
```
- Run the server so that it compiles any new changes

```bash
$ lein ring server
```
- Open another terminal
- Switch to `my-blog`

```bash
$ cd my-blog
```
- Open resources/templates/config.edn and update the following values
    - :site-url `"http://username.github.io/"`
    - :blog-prefix `""`
    - :site-title `"your site title"`
    - :author `"@twitterhandle"`
    - :description `"your description"`
- Make sure .gitignore has `/resources/public/` in it.
- Initialize your cryogen-blog by running the following in `my-blog` directory

```bash
$ git init
$ git commit -m "first commit"
$ git remote add origin git@github.com:<YOUR-USERNAME>/cryogen-blog.git
$ git push -u origin master
```
- But this will only push a new repo, this does not create any blog.
- To create the blog, go to github again & create another repo with same name as your username, i.e. `username.github.io`
- Run

```bash
$ cd my-blog/resources/public
$ git init
$ git add README.md
$ git commit -m "first commit"
$ git remote add origin git@github.com:<YOUR-USERNAME>/<YOUR-USERNAME>.github.io.git
$ git push -u origin master
```
- And now check `YOUR-USERNAME.github.io`, you should see your blog.
- Check [this](http://cryogenweb.org/docs/writing-posts.html) for how to write posts.
- Then check `localhost:3000` to make sure it all looks good.
- To push new changes simply run the following (from `my-blog` directory)

```bash
$ git add . && git commit -am "WIP" && git push
$ cd resources/public &&  git add . && git commit -am "WIP" && git push && cd ../../
```
- You can make that as a command as well.
- Create the following snippet as a command deploy.sh

```bash
#!/bin/sh

echo "Deploying blog post"
git add . && git commit -am "WIP" && git push;
cd resources/public &&  git add . && git commit -am "WIP" && git push && cd ../../
open http://<YOUR-USERNAME>.github.io
```
- Make it executable

```bash
$ chmod +x deploy.sh
```

- Make changes and deploy!

```bash
$ ./deploy.sh
```


Thanks to [tangrammer's instructions here](https://github.com/tangrammer/cryogen-blog/blob/master/resources/templates/README.md#instructions-to-make-changes)
