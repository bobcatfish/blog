---
title: "Blog! Hugo, cloudflare, ec2"
date: 2018-03-04
draft: false
---

Since I have lots of opinions and nowhere to put them, I decided to make a blog!

![A blogging cat](/blogger.png)

## Where to host it

[I've posted on Medium before](https://medium.com/@bobcatwilson/) but I really like the feeling of blogs that folks make themselves, and I particularly like [Julia Evans's blog](https://jvns.ca) so I thought trying to copy her would be a good start!

The first thing I learned is that Julia is Canadian, like moi, which I didn't realize.

The next thing I noticed is that [her posts are all in markdown](https://github.com/jvns/jvns.ca/tree/master/content/post) which is appealing because I already like markdown and it's simpler than trying to setup something like wordpress and dealing with accounts and all of that.

So all I need is something that statically serves markdown files with some nice rendering.

## Golang

I've been trying to find excuses to use Golang a lot more lately, especially at work but also during [Advent of Code 2017](https://github.com/bobcatfish/adventofcode#advent-of-code) (I really need to post the rest of my solutions up there - next year I swear I'll post as soon as I solve a puzzle; this year I was the bad, sinful kind of lazy where I hadn't setup vim correctly for Golang and so my files were all formatted badly 😅😅😅). So why not make this blog in Golang too?

Looks like [3 years ago someone on reddit got the same idea](https://www.reddit.com/r/golang/comments/2lphue/making_a_blog/) and people recommended:

* [`net/http`](https://golang.org/pkg/net/http/) (of course they did, "this is Go", let's build everything from scratch XD)
* [Gorilla](http://www.gorillatoolkit.org/)

[The golang blog itself](https://github.com/golang/blog) has its source online. Poking through I stumbled on [`appengine.go`](https://github.com/golang/blog/blob/master/blog/appengine.go), which since I work on AppEngine at Google, I found kind of neat. And from the commit messages (I ❤️ commit messages p.s.) it's clear they are hosting the blog on AppEngine. I visited the website to see if there was anything in the source it served to indicate its AppEngine-y-ness, and there wasn't, but I noticed they had a post where they mentioned [Hugo](https://gohugo.io/) is now ["the most popular open-source static website engine"](https://blog.golang.org/8years) which sounds kind of like what I'm looking for.

## Hugo

[Hugo's main page](https://gohugo.io/) has a [quickstart](https://gohugo.io/getting-started/quick-start/) - I am in love already. Nothing like some careful attention to documentation to get me excited about a project!

And [the Hugo source is on GitHub](https://github.com/gohugoio/hugo)! I wonder if their trend for beautiful docs extends to their commit messages? Poking through their commits, I would say generally yes!

![A documenting gopher](/doc_gopher.png)

They win at documentation, I'll give this a shot.

### Getting started

Following along with [the quickstart](https://gohugo.io/getting-started/quick-start/), right off the bat I'm a bit weirded out that it wants me to `brew install hugo`. My first reaction was "wut" and my second reaction was "oh god but I'm using [Arch](https://www.archlinux.org/)."

Fortunately they link to [alternate installation methods](https://gohugo.io/getting-started/installing). I skimmed straight to the examples with `go get`: [fetch from GitHub](https://gohugo.io/getting-started/installing#fetch-from-github).

_A brief interlude about one of my pet peeves:_ I had to inspect the source to find the name of the anchor for "Fetch from GitHub"! It's a title, and there's a link to it that exists, but there is no link on the page itself! What a pita.

Looks like Hugo uses something called [magefiles](http://github.com/magefile/mage), which have automatically won me over because if there's something I like even more than documentation, it's gophers:

![The magefile mascot](/magefile.png)

"Makefiles are hard to read and hard to write." Well I agree with the sentiment, I think magic is cool, and you've got a sweet Gopher. Sold.

The Hugo quick start itself is really clean and simple. I like how the steps are punctuated with little videos showing the commands in action (though I had to reload the page to get them to work). They were recorded with [`asciinema`](https://asciinema.org/) which looks super handy. I love services that are really focused like that and do one thing really well: video hosting, but only for terminal sessions. Nifty!

And guess what the Hugo walkthrough uses as its example use case? A blog full of markdown files! Perfect!

After becoming overwhelmed and slightly irritated at the number of themes availabe (I JUST WANT MARKDOWN DAMMIT) I picked [Beautiful Hugo](https://themes.gohugo.io/beautifulhugo/).

_It was at this point I started to become terrified of losing everything I'd typed in this file so far before pushing it to github and I backed it up about 30 times. Okay once._

I added my first blog post, I ran `hugo server -D` and eagerly loaded `localhost:1313` in my browser and saw... nothing. I had to run with `--debug` to realize the blog post was being served at `/posts`. I found it helpful to review [the docs on the directory layout](https://gohugo.io/getting-started/directory-structure/) to understand what I was working with, and when looking at the [template lookup order](https://gohugo.io/templates/lookup-order/) I finally realized that I had put my post into a directory called `posts` instead of `post`! It turned out the theme came with an awareness of the type `post` but not of the type `posts`.

Taking a look at [beautiful hugo's example `config.toml`](https://github.com/halogenica/beautifulhugo/blob/master/exampleSite/config.toml) showed me lots of slick things I could enable, such as reading time and links to social media!

## Webserver

Hugo is all about static content generation and [not at all about hosting and deploying that content](https://gohugo.io/hosting-and-deployment/). I quickly ruled out the options I found the most appealing:

* Nanobox - "Using Nanobox deploy also means you'll use it to develop your application" - Uh, no thank you
* Firebase - Cool, more Google stuff, but then also, well I really wanted to use Amazon, where I already have my other sweet website ([sugardrill.com](http://sugardrill.com)). Maybe I'll come back to this later.
* GitHub - GitHub pages are pretty neat, but I kind of want to use ec2 for no great reason.

Well they're just static files so maybe I can get away with handbombing them into Apache.

## The Cloud

Now here is a mystery: I have 0 running instance in EC2. How is [my beautiful website](http://sugardrill.com) running? Has it been cached on Cloudflare all this time? Do I wish I had taken notes the last time I did this? Yes.

![Cat in the clouds](/cloud_cat.png)

Maybe [Cloudflare](https://www.cloudflare.com/) has some answers. Well it looks like my domain name is pointing straight at an IP, and when I look at that IP, I see the cat butt, so it looks like the website is still up, but where is it if it's not on EC2?

I did an `nslookup` on the IP and it looks like it's at `us-west-2.compute.amazonaws.com`. Oh! `us-west-2`! I was looking at `us-east-1`. And there it is, the little instance that could, running with no redundancy and no automation or protection of any kinda, faithfully serving up cat butts.

I will build here. What could go wrong?

Fortunately my `.histfile` goes back forever so I could easily find my aws key and remember how I was logging into that host. _Note to self: Must make sure to save those .ssh keys when I redo my Arch setup._ After watching the command hang forever I realized the security groups had some previous public IP of mine hardcoded into them so I updated it to my current IP. So elegant. I also added a rule to allow access to :80 from anywhere.

I created another `t2.micro` (don't want to overload the machine with the cat butts, it has a lot to do).

## Apache

Apache was my first webserver. Year ago I read an entire dull O'Reilly book about it. (All I retained was that it's funny that there's an `APR_HOOK_LAST` and an `APR_HOOK_REALLY_LAST`). There's probably something cooler and simpler these days, but setting up an Apache webserver is pretty easy so I'll use it for now. If I come back to this to add some deployment automation maybe I'll branch out.

On the server:

```bash
sudo yum install httpd
```

Locally:

```bash
hugo
scp -i ~/.ssh/aws.pem -r public/* ec2-user@$BLOG_HOST:/var/www/html/
```

On the server:
```bash
sudo /etc/init.d/httpd restart
```

## bobcatfish.com

I really wanted to finally use the domain name I've been hoarding for the last ~8 years, [bobcatfish.com](http://bobcatfish.com).

Time to get it pointing at my new blog, doing something useful for the first time in 8 years since [@sometorin](http://twitter.com/sometorin) and I stalemated each other by getting each other's email address as domains (`bobcatfish.com`, `torinsandall.com`) in a mutually assured destruction pact. He quickly lost interest and now I own both. Your time is coming soon, `torinsandall.com`. Maybe dog butts.

Looks like `bobcatfish.com` goes back to my days of using 1&1 internet.com. Yay. Now I use ghandi.com because it is way cooler and doesn't leak my personal information around on the internet. Did I mention it's cooler? Maybe one day I can transfer providers.

I guess I'll use cloudflare again, hopefully they'll let me have two domians for free. So I need to point my 1&1 domain at cloudflare. I logged into cloudflare, clicked "add site" and it was pretty easy from there though 1&1 made it kind of hard to see if I'd updated the nameservers correctly.

After waiting a while I realized it was working but cloudflare had copied the configuration from 1&1 and was still pointing at their awful default site, so I updated the A records to point at my ec2 instance and removed some records (ftp, AAAA - I have no ipv6 IPs on my instances apparently, MX).

## Problems

Several minor things went wrong from there.

### Ugly hugo

The "beautiful" hugo layout wasn't quite looking beautiful.

This problem went away on its own the next day, once the DNS changes had propogated, but it remained on cloudflare's free (god bless their hearts) `https` endpoint:

![banner at the top of the page overlaps; https warning about insecure scripts](/https.png)

_I took this screenshot using [`scrot`](https://www.howtoforge.com/tutorial/how-to-take-screenshots-in-linux-with-scrot/), in spite of / especially because ofthe name sounding to me like "scrotum"._

Viewing the `https` endpoint gives me a big hint: the chromium warning that the page is loading insecure scripts. I updated the `BaseUrl` in my `config.toml` to include `https` and the formatting was fixed! I [configured cloudflare to always redirect to https](https://support.cloudflare.com/hc/en-us/articles/200170536-How-do-I-redirect-all-visitors-to-HTTPS-SSL-) and that was the end of that.

### defaultsite

After everything was working, `bobcatfish.com` kept redirecting me to `bobcatfish.com/defaultsite`. I couldn't find anything in my Apache configuration that explained it but eventually I stumbled on [this google product forum post](https://productforums.google.com/forum/#!topic/apps/v9YNjxKRuKc) and several blog posts that had very complicated solutions to what turned out to be a simple problem: I just had to clear my cached files in Chrome and the redirection stopped.

### No image

Initially the images did not load.

I looked up [the hugo docs on static files](https://gohugo.io/content-management/static-files/) but the docs were focused on how to configure the directory you want to use, and not on how to access the content from markdown. [This hugo support post](https://discourse.gohugo.io/t/solved-how-to-insert-image-in-my-post/1473) finally gave me the answer. After misreading the solution several times, the answer turned out to be that anything in `static` is copied to the root of your site, and you can access it with an absolute path, e.g.:

```md
![my alt text](/my_image.png)
```

## Hugo hot reloading

`hugo server` is super slick. It helped me iterate on editing and layout issues. The hot reloading made this particularly smooth - the page at localhost automatically reloaded with changes as I made them, including discovering and syncing images as I added them.

Using `hugo server --bind=$MY_LOCAL_IP --baseURL=$MY_LOCAL_IP` also made it easy for me to force `@sometorin` to give me feedback before I uploaded the final version.)

## And then it worked!

In the end I didn't actually have to/get to use any Go (story of my life). Next time. But the blog works, and [I've got the source up in Github](https://github.com/bobcatfish/blog). Woot!

![A 'deal with it' cat](/deal_with_it_cat.png)
