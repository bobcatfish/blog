# Christie's blog

This blog was created using [Hugo](http://gohugo.io).

View my blog at [bobcatfish.com](http://bobcatfish.com)!

## Updating

Generate Hugo static content to [`public`](./public) with the `hugo` command:

```bash
hugo
```
_You must set 'draft: false` on any new posts._

Copy the files from `public` to AWS with:

```bash
scp -i ~/.ssh/aws.pem -r public/* ec2-user@$BLOG_HOST:/var/www/html/
```
