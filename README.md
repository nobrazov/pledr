[![Build Status](https://travis-ci.org/jmacdotorg/Pledr.svg?branch=master)](https://travis-ci.org/jmacdotorg/Pledr)
# Pledr 

Pledr is meant to be an ultralight blogging platform for Markdown fans that plays well with (but does not require) Dropbox.

It allows you to compose and maintain blog posts as easily as adding and modifying Markdown files in a single folder. Pledr creates an entirely static website based on the content of this one folder, automatically updating the site whenever this content changes.

Optional and experimental Pledr features add support for IndieWeb technologies, such as sending webmentions.

## Purpose

Pledr allows a blogger to publish new posts to their blog simply by adding Markdown files to a designated blog-source directory. By mixing in Dropbox, this directory can live on their local machine. They can also update posts by updating said files, and unpublish posts by deleting or moving files from that same folder.

The generated website comprises a single directory containing only static files. These include one "permalink" HTML page for every post, a recent-posts front page, a single archive page (in the manner of [Daring Fireball](http://daringfireball.net/archive)), and syndication documents in Atom and [JSON Feed](http://jsonfeed.org) formats. All these are constructed from simple, customizable templates.

That's it! That's all Pledr does.

If you have the time and inclination, you may watch [a 20-minute presentation about my reasons for creating Pledr](http://fogknife.com/2015-06-09-my-yapcna-2015-talk-about-blogging.html).

## Project status

Pledr is released and stable. It still has plenty of room for improvement, and I welcome community feedback and patch proposals, but it will continue to do what it does now, in more or less the same fashion, for the foreseeable future.

I reserve the right to add new features to it, but I will do my best not to break existing functionality when doing so. If you'd like to stay informed of Pledr changes, consider joining [the Pledr-announce mailing list](http://lists.jmac.org/listinfo/Pledr-announce).

## Setup

### Installation

**First, make sure you have the `cpanm` program on your machine.** It is likely
available as "cpanminus" in your favorite package manager. (Or install it from
source, through the instructions at [http://cpanmin.us](http://cpanmin.us).)

Then, run this command:

    cpanm Pledr

If you run into issues due to failing dependencies, you can try one of these instead:

    cpanm --notest Pledr
    # or
    cpanm --force Pledr

Alternately, you can run these commands under \`sudo\` to install Pledr at the system level.

If everything installed as it should, then you should have the `Pledrall` and `Pledrwatcher` programs in your command path.

**To install Pledr from source**, set the current working directory to the same directory containing this README file, make sure you have `cpanm` as described above, and then do this:

    cpanm --installdeps .
    perl Makefile.PL
    make
    make install
    

### Configuration

1. Run `Pledrall --init` to create a new directory called `Pledr/` in your current working directory, populated with all the special files and directories that Pledr requires. This includes a sample config file in `Pledr/Pledr.conf`.

    If you'd like the directory named something else or located somewhere else, you can provide it as an argument, e.g. `Pledrall --init=/some/other/location`. See the `Pledrall` man page for full details.

    Here's the purpose of the subdirectories that `Pledrall --init` creates:

    - `source`: This will hold your blog's Markdown-based source files.
    - `templates`: Holds your blog's templates. `Pledrall --init` will place a full complement of sample templates in this directory for you.
    - `docroot`: Will hold your blog's actual docroot, ready for serving up by the webserver software of your choice.
    - `conf`: Holds your blog's configuration files. `Pledrall --init` will place an example `Pledr.conf` file in this directory for you.
    - `db`: Will contain metadata about your blog's posts.
    - `run`: Holds PID files and such.
    - `log`: The `Pledrwatcher` program will write logs here.

    You can freely add other files or directories in this directory if you wish (a `drafts` folder, perhaps?). Pledr will happily ignore them.

2. Edit the `Pledr.conf` file in the directory that you created in the previous step to best suit your needs. The file itself is extensively commented and self-documenting.

    You can optionally move or copy the file to `.Pledr` in your home directory. If you do, that new copy of the file becomes the default configuration file that Pledr's command-line programs will refer to.

3. As noted above, the `templates` directory that you created in the first step of this process contains sample templates that you can customize as much as you'd like. They are rendered using [Template Toolkit](http://www.template-toolkit.org). You can't change these template files' names, but you can add new sub-template files that the main temlates will invoke via [the \[% INCLUDE %\] directive](http://www.template-toolkit.org/docs/manual/Directives.html#section_INCLUDE), and so on.
4. Configure the webserver of your choice such that it treats the `docroot` subdirectory (which you created as part of the first step) as your new blog's own docroot.

    When running in its basic mode, Pledr does not provide a webserver; it simply generates static HTML & XML files, ready for some other process to serve up.

## Usage

### Running Pledr

Pledr includes two command-line programs:

- **Pledrall** creates a new website in Pledr's docroot directory, based on the contents of its source and templates directories. (It also provides a few other "one-off" utility functions, including the `--init` feature referred to above.)

    Run this program (with no arguments) to initially populate your blog's docroot, and at any other time you wish to manually regenerate your blog's served files.

- **Pledrwatcher** runs a daemon that monitors the Dropbox-synced source directory for changes, republishing files as necessary.

    **_This is where the magic happens._** While both _Pledrwatcher_ and Dropbox's own daemon process run on your webserver's machine, any changes you make to your blog's source directory will instantly update your blog's published static files as appropriate.

    Launch Pledrwatcher through this command:

        Pledrwatcher start

    It also accepts the verbs `stop`, `restart`, and `status`, as well as [all the command-line options listed in the App::Daemon documentation](https://metacpan.org/pod/App::Daemon#Command-Line-Options).

### Composing posts

To start writing a new blog post, just create a new Markdown file somewhere _outside of your blog's source directory_. (Recall that any change to the source directory instantly republishes the blog, something you won't want to do with a half-written entry on top.) You can name this file whatever you like, so long as the filename ends in either `.markdown` or `.md`.

You must also give your post a title, sometime before you're ready to publish it. You define the title simply by having the first line of your entry say `title: [whatever]`, followed by two newlines, followed in turn by the body of your post.

For example, a valid, ready-to-publish source file could be called `today.markdown`, and it could contain this, in full:

    title: My day today
    
    I had a pretty good day today. 
    
    I hung out at [the coffee shop](http://empireteaandcoffee.com). Then I went home.
    
    Well, that's all for now. Bye bye.

### Publishing posts

To publish a post, simply move it to Pledr's source directory. (Take care not to overwrite an older post's source file that may have the same name.)

Pledr will, once it notices the new file, give the file a timestamp recording the date and time of its publication. This timestamp will appear in its own line, after the title line.

Normally, Pledr will set the publication time to the moment that you added the file to the source directory. Pledr recognizes two exceptions to this rule:

- If you manually give your post a `time:` timestamp, and it's in W3C date-time format, then Pledr will leave that timestamp alone.
- If you leave the timestamp out, _and_ include in your post's filename a date of yesterday or earlier (e.g. `1994-06-10-i-like-ace-of-base.md`), then Pledr will set the post's timestamp to midnight (in the local time zone) of that date. This allows you to batch-backdate many posts at once -- useful, perhaps, for populating a new blog with existing writing.

(Note that Pledr assumes you use a text editor smart enough to see that the source file has both moved and had additional lines added to it from an external process, and to react to this in a graceful fashion.)

Once it has prepared the source file, Pledr will update the blog. It will create a new HTML file for the new entry, and add a link to it from the `archive.html` page. It will also appear in the recent-posts sidebar of every other entry, as well as the Atom and JSON Feed documents (unless you decided to manually backdate the entry by specifying your own date attribute within the file).

### Updating or deleting posts

To update a blog post, just edit its source Markdown file, right in the source directory. Any changes you make will immediately update your published blog as appropriate

To unpublish a blog post, simply move it out of the synced source directory -- or just delete it.

## Using Pledr with Dropbox

Pledr loves Dropbox! (Indeed it had Dropbox affinity in mind from the beginning of its design.)

To have Pledr work with Dropbox, just place its working directory (the one containing the source, docroot, and template subdirectories) somewhere in your synced Dropbox folder, and specify the local path to this folder (from your webserver's point of view) in your Pledr config file.

Now, you can create, update, and delete blog posts just by moving and editing files, _no matter what computer you're using_, so long as it has access to that Dropbox folder.

In this way you could, for example, compose and edit blog posts via Markdown in your favorite text editor while sitting by the fire with your laptop in the back of your favorite coffee shop, publishing updates to your blog by hitting _File → Save_ in your text editor, and not directly interacting with your webserver (or, indeed, with the Pledr software itself) in any way.

## Advanced use

### Customizing templates

For a brief guide to the template files and how to customize them for your blog, please [see the Pledr wiki on GitHub](https://github.com/jmacdotorg/Pledr/wiki/Pledr-template-guide).

### Tags

If you define a list of comma-separated tags under a post attribute named `tags`, then Pledr will add the post to those linked from a file named `tags/[tag].html`, relative to the blog's docroot. It will also link to that page form `tags/index.html`.

For example, this attribute would assign three tags to its post:

    tags: Media, Books I like, 📚

The default Pledr templates will display links to tag-pages where appropriate. Tag pages get their shape from the template named `tags.tt`.

_Mind your capitalization with tags!_ If faced with inconsistent capitalization within a single tag, e.g. one post claims "boston" for a tag and other one claims "Boston", then Pledr will prefer the first tag containing capital letters to one that contains none, and it will retroactively apply to across all relevant posts.

### User-defined attributes

You can add any attributes you'd like to your posts, and then refer to them from your templates via a hash named `attributes` attached to every post object. For example, if a post's metadata looks like this:

    title: Example of user-defined attributes
    byline: Sam Handwich

Then you can refer to `post.attributes.byline` to fetch that value from within the `post.tt` template file, even though "byline" is not an attribute that Pledr otherwise recognizes. (If a template refers to an attribute key that a post's source file does not define, it will simply return a blank value.)

### Social-media metatags

By defining some extra attributes in both your blog's configuration file, you
can direct Pledr to add [Open Graph](http://ogp.me) and [Twitter Card](https://dev.twitter.com/cards/overview)-enabling metadata tags to each of your posts. This will allow services like Facebook, Twitter, and Slack to present attractive little summaries of your blogposts when displaying links to them.

These blog configuration options (all optional) are:

- **twitter\_id**: If present, then Pledr will try to attach Twitter Card metadata to each post, associated with the given Twitter username. (No leading '@', please. Yes, I know. It confuses the YAML parser. Sorry...)
- **image**: If present, Pledr will use this URL as the location of a default image to use in the metadata for any post that doesn't define its own _image_ attribute.

    If _not_ present, Pledr will _not_ generate any social-media metadata for any post lacking an _image_ attribute.

- **image\_alt**: A textual description of the image referenced by the `image` attribute. (Equivalent in usage to the "alt" attribute in an HTML `<img>` tag.) Pledr will just leave this blank, if you don't define it yourself.

To see examples of all the above, please see the file `conf/Pledr_example.conf`.

Once you've configured your blog as described above, you can add these attributes to any post:

- **description**: A very brief summary of this post.

    If not defined, then Pledr will try to use the first paragraph of your post's text (after stripping out any markup) as the post's description.

- **image**: The URL of an image to associate with this post within social-media links. (This could refer to image that also appears in your post by way of an HTML `<img>` tag, but it doesn't have to.) 

    If not defined, then Pledr will instead use the blog's _image_ configuration directive. If _that_ is also undefined, then Pledr will not generate any social-media metadata for this post.

- **image\_alt**: A textual description of the image referenced by the `image` attribute. (Equivalent in usage to the "alt" attribute in an HTML `<img>` tag.) Pledr will just leave this blank, if you don't define it yourself.

### MultiMarkdown

Pledr supports [MultiMarkdown](https://fletcherpenney.net/multimarkdown/) syntax out of the box! Go ahead and put MultiMarkdown tables and stuff into your posts. it'll just work.
