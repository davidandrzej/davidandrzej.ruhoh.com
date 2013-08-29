## Technical blog notes

Some "note to self" reminders about the way I've used ruhoh to set up this blog.


#### Custom "About" page

* put page fragment in `_root/about.html` (I used a simple Bootstrap `container` class)
* add `navigation > about` to `data.yml`


#### S3-hosted images

* from AWS console, enable public http serving for your S3 bucket
* add `images : https://s3.amazonaws.com/mypublicbucketname` to `data.yml`
* link to images with `<img src="{{ data.images }}/my-image-file.jpg"></img>`


#### RSS 

* in `config.yml` flip `_root > rss > enable` to `true`
* in theme `default.html`, add `href="{{urls.base_path}}/rss.xml">` somewhere


#### mathjax

* put [this html](https://gist.github.com/davidandrzej/6381171) in `widgets/mathjax/default.html` (original source: [this Github discussion](https://github.com/ruhoh/ruhoh.rb/issues/46#issuecomment-6988308))
* add `{{{ widgets.mathjax }}}` in the theme `default.html` file
* note that underscores will not work properly unless you wrap in a `<div>` block (see discussion linked above)
* also see [discussion](https://github.com/ruhoh/ruhoh.rb/issues/209#issuecomment-23464200)
  around my misunderstanding of this feature

