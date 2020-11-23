# wordpress-to-markdown

This script uses the standard exported XML file from WordPress, and creates a folder/file structure that contains all of the blog posts, converted to markdown format. It will also download all of the images.

Instructions for exporting your information from WordPress [can be found here](http://en.support.wordpress.com/export/).

This is a upgraded version of the original project. I fixed a bunch of issues and edge cases. [You can read about it here](https://swizec.com/blog/how-to-export-a-large-wordpress-site-to-markdown/)


## Works on my box

I tested this on my machine and my blog. The blog is 14 years old with 1500+ posts and I hope that makes this script battle tested.

It seems to have worked for someone else too ✌️

[![](https://i.imgur.com/tajlNWE.png)](https://twitter.com/akhromieiev/status/1301526142898364417)

## Usage

This program will put the output into `/out` folder, and also all images will go to `/img`. Image urls are rewritten to `./img`, which is what most markdown static site generators enjoy.

Export your Wordpress to XML then:

```
    $ git clone https://github.com/Swizec/wordpress-to-markdown

    $ cd wordpress-to-markdown

    # download your wordpress xml (into the same folder, above)
    # change .xml filename on line 25 of convert.js

    $ yarn
    $ yarn convert

    # sip margaritas
```

## Technical Details

This uses [xml2js](https://github.com/Leonidas-from-XIV/node-xml2js) to parse the XML, and then uses a combination of Rehype, Remark, Prettier, and custom plugins to convert your blogs into prettified Markdown. Embeds become links so your site can run modern-style embedding.

## Conversions the script runs for you

wordpress-to-markdown performs a number of conversion for you that make stuff work better with modern static site builders. [Read my article for technical details on how this works](https://swizec.com/blog/how-to-export-a-large-wordpress-site-to-markdown/)

### Metadata into Frontmatter

I wanted to create full frontmatter without manual edits. That means:

```
    ---
    title: 'Always put side effects last'
    description: ""
    published: 2018-01-10
    redirect_from:
                - /blog/always-put-side-effects-last/swizec/8057
    categories: "Startups, Technical"
    hero: ./img/wp-content-uploads-2016-10-salesforce-tower-panorama-1024x358.jpg
    ---
```

Title from post title, description based on meta data, a publish date, keep old URL for redirects, combine categories and tags into categories, find a good hero/social image.

Data comes from digging around Wordpress exports and figuring out what fits.

### Edge case 1: Make your HTML parseable

Wordpress HTML is pretty good. Plop it in an HTML parser and, like, it won't choke ... but it won't parse correctly either.

We change double newlines to paragraph breaks. Wordpress doesn't wrap paragraphs in `<p></p>` tags

### Edge case 2: Bad code blocks

I wrote about fixing bad code blocks in my [You though computer science has no place in webdev? Here's a fun coding challenge](https://swizec.com/blog/you-though-computer-science-has-no-place-in-webdev-heres-a-fun-coding-challenge/) article.

Your challenge is that this isn't valid HTML:

```html
<pre lang="javascript">
class ReportSize extends React.Component {
  refCallback = element => {
    if (element) {
      this.props.getSize(element.getBoundingClientRect());
    }
  };

  render() {
    return (
      <div ref={this.refCallback} style={{ border: "1px solid red" }}>
        {faker.lorem.paragraphs(Math.random() * 10)}
      </div>
    );
  }
}
</pre>
```

JSX tags get parsed as HTML and break your code block. You want them to include a `<code></code>` tag as well. Otherwise Markdown stringifying doesn't work right.

Fixing this is tricky and this script does it for you.

### Edge case 3: Fixing embeds

Lots of ways to embed 3rd party content on a wordpress site. You can use plain old links pasted on their own line, shortcodes, and full HTML embeds.

Markdown site generators like to use plain links.

You want to change code like:

```html
<blockquote class="twitter-tweet">
  <p lang="en" dir="ltr">
    A script that converts Wordpress dumps into clean Markdown may have been the
    dumbest project I ever took on. Sooooo many edge cases 😅
    <a href="https://t.co/z8dPUMrBGk">pic.twitter.com/z8dPUMrBGk</a>
  </p>
  &mdash; Swizec Teller (@Swizec)
  <a
    href="https://twitter.com/Swizec/status/1298308910072307713?ref_src=twsrc%5Etfw"
    >August 25, 2020</a
  >
</blockquote>
<script
  async
  src="https://platform.twitter.com/widgets.js"
  charset="utf-8"
></script>
```

Into Markdown that's a link:

```markdown
https://twitter.com/Swizec/status/1298308910072307713
```

Site generator can take this and turn it into an embed. When it starts as a blockquote, you'll have trouble.

Script does this for you. If you find an unsupported service, PRs welcome :)

### Edge case 4: Fixing shortcodes

Shortcodes are a semi-standard system of snippets. Denoted by `[]` they give CMS users the ability to go beyond writing text.

These were popular on internet forums of the late 2000's. Wordpress supports them to this day. Don't know about others.

I wanted to get rid of most and preserve any embeds.

You can identify an embed because it's a closed shortcode prefixed with the name of a service followed by a link.

```html
[tweet https://twitter.com/Swizec/status/1298308910072307713]
```

The gnarly ones are Wordpress's almost-html shortcodes. Big issue on my site were the `[caption][/caption]` shortcodes.

We convert all that to standard markdown. ✌️

### Edge case 5: Underscores in links

This one was frustrating. Embed links can include underscores, like when you embed a tweet from `@_developit`.

Markdown stringification escapes underscores because it thinks they're emphasis and doesn't understand that some text nodes are link nodes despite not being links.

```markdown
https://twitter.com/_developit/status/1300154097170083842
```

That breaks your embed machinery so we fix it. 🤪

### License

The MIT License (MIT)

Copyright (c) 2013 Jason Young

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

