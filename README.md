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

* put this html in `widgets/mathjax/default.html` (original source: [this Github discussion](https://github.com/ruhoh/ruhoh.rb/issues/46#issuecomment-6988308))
```html
<!-- INSERT MATHJAX CONFIGURATION FROM MATH.STACKEXCHANGE -->
<script type="text/x-mathjax-config">
  MathJax.Hub.Config({
    jax: ["input/TeX", "output/HTML-CSS"],
    tex2jax: {
      inlineMath: [ ['$', '$'] ],
      displayMath: [ ['$$', '$$']],
      processEscapes: true,
      skipTags: ['script', 'noscript', 'style', 'textarea', 'pre', 'code']
    },
    messageStyle: "none",
    "HTML-CSS": { preferredFont: "TeX", availableFonts: ["STIX","TeX"] }
  });
</script>
<script type="text/javascript"
        src="https://c328740.ssl.cf1.rackcdn.com/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>
```
* add `{{{ widgets.mathjax }}}` in the theme `default.html` file
* note that underscores will not work properly unless you wrap in a `<div>` block (see discussion linked above)
* also see [discussion](https://github.com/ruhoh/ruhoh.rb/issues/209#issuecomment-23464200)
  around my misunderstanding of this feature

