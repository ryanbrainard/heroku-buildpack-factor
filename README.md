Heroku Buildpack: Factor
========================

A [Heroku Buildpack](http://devcenter.heroku.com/articles/buildpack) for [Factor](http://factorcode.org) applications.

Usage
-----
When Heroku receives your code, this buildpack downloads Factor and places your code in the `work` directory, so your repo's root should be that of the `work` directory. 

If your app is `USING` large vocabularies that could delay start up time, such as `furnace`, this buildpack allow you to create a pre-baked, cached image. To do so, add a `system.properties` file to the root of your project with a `factor.image.using` property specifying which vocabularies to include in the image. For example:

    $ cat system.properties
    factor.image.using=furnace

Example
-------
This is an example of creating a [Furnace](http://concatenative.org/wiki/view/Factor/Furnace) web app on Heroku using this buildpack. The code for this example is based on an [article from _Re: Factor_ about getting started with Furnace](http://re-factor.blogspot.com/2010/08/hello-web.html), which explains in detail how the app works.

_To get started, first follow the [Heroku Quickstart](https://devcenter.heroku.com/articles/quickstart). For "Step 4: Deploy An Application," follow these steps:_

1. Create an empty directory, change to, and initialize a Git repo:

        $ mkdir hello-factor
        $ cd hello-factor
        $ git init

2. Create a Heroku app using this buildpack. A randomly generated app name will be generated.

        $ heroku create --buildpack https://github.com/ryanbrainard/heroku-buildpack-factor.git

        Creating salty-depths-9179... done, stack is cedar
        BUILDPACK_URL=https://github.com/ryanbrainard/heroku-buildpack-factor.git
        http://salty-depths-9179.herokuapp.com/ | git@heroku.com:salty-depths-9179.git
        Git remote heroku added

3. Create a file at `webapps/hello/hello.factor` with the following contents. This is the code for the web application that will run on Heroku. Note that the server binds to the `PORT` environment variable. Heroku will automatically supply a value for `PORT` during start up of the [web dyno](https://devcenter.heroku.com/articles/dynos).

        USING: accessors furnace.actions http.server
        http.server.dispatchers http.server.responses io.servers kernel
        namespaces environment math.parser ;

        IN: webapps.hello

        TUPLE: hello < dispatcher ;

        : <hello-action> ( -- action )
            <page-action>
                [ "Hello, world!" "text/plain" <content> ] >>display ;

        : <hello> ( -- dispatcher )
            hello new-dispatcher
                <hello-action> "" add-responder ;

        : run-hello ( -- )
            <hello>
                main-responder set-global
            "PORT" os-env string>number httpd wait-for-server ;

        MAIN: run-hello

4. Create a file called `Procfile` with the following contents. The [`Procfile`](https://devcenter.heroku.com/articles/procfile) tells Heroku how to start your application.

        web: ./factor -run=webapps.hello

5. Create a file called `system.properties` in the root with the following contents. This is an optional file that will pre-bake in the `furnace` vocabularies into the image to speed start up times.

        factor.image.using=furnace

6. Add and commit everything to Git:

        $ git add .
        $ git commit -m init

        [master (root-commit) 867242e] init
         3 files changed, 25 insertions(+)
         create mode 100644 Procfile
         create mode 100644 system.properties
         create mode 100644 webapp/hello/hello.factor
   
7. Push to Heroku. The first push will take longer because the image is being prepared, but subsequent pushes should be faster.

        $ git push heroku master

        Counting objects: 7, done.
        Delta compression using up to 4 threads.
        Compressing objects: 100% (3/3), done.
        Writing objects: 100% (7/7), 758 bytes, done.
        Total 7 (delta 0), reused 0 (delta 0)

        -----> Heroku receiving push
        -----> Fetching custom buildpack... cloning with git...done
        -----> Factor app detected
        -----> Downloading Factor... done
        -----> Preparing image... done
        -----> Preparing work dir... done
        -----> Discovering process types
               Procfile declares types -> web
        -----> Compiled slug size is 46.0MB
        -----> Launching... done, v4
               http://salty-depths-9179.herokuapp.com deployed to Heroku

        To git@heroku.com:salty-depths-9179.git
         * [new branch]      master -> master    

8. View your running app:

        $ heroku open

