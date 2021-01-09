---
title: "About this site (a tutorial)"
date: 2020-12-17
draft: false
tags: ["tutorial", "hugo", "golang", "aws"]
---
I started this website in 2018, because I wanted to collect all my content in one single place. I decided to built it with simple tools, so I could understand how everything works, and fix problems if needed. It's a hobby like how some people like to build their own kitchens. Lots of tutorials are available on building personal websites, but I still want to write my own, because when I got started, I could never find one that explained every step needed to build a complete website. The problem was not that I couldn't find information about how to perform certain technical steps. The problem was that I wasn't sure which steps were relevant. So I will document everything I have done for this site, and I hope that it could be helpful for those who want to hand-build their own sites.

### Static website generation with [Hugo](https://gohugo.io)
The easiest way to make a website is to use a static website generator. The way it works is that you provide a folder of documents written in [Markdown](https://www.markdownguide.org), and the generator makes a website for you, based on the theme that you choose. The generated website is just a bunch of HTML, css, and javascript files. There are no content management systems and no databases. For the simple website that I have in mind, this is  more than enough. But of course, it means that it's harder to include fancy features such as webstores, discussion forums, and other forms of interactivity. For that kind of thing, you are probably better off with dynamic solutions such as WordPress or SquareSpace.

There are many static website generators to choose from. In fact, the number is growing so fast that it's hard to keep track of them. I picked [Hugo](https://gohugo.io) because it's easy to understand and easy to customize. It's written in the [Go programming language](https://golang.org), which tends to encourage minimalist design. One feature that I found very attractive is that although Hugo doesn't use a content management system, it is very flexible in content organization. In addition to the traditional organization tools such as tags and categories, you can also define your own taxonomy, which can be used to implement database-like features. 

Simple websites are easy to build with Hugo. The only tricky part is that Hugo doesn't have a GUI or web-based interface, so you have to interact with it using the unix shell. As long as you are comfortable with the command line, it takes very little time and effort to get something basic running. The most important thing is to pick a theme. I picked a minimalist theme called [Pickles](https://themes.gohugo.io/hugo_theme_pickles/) and never look back, but there are many [options](https://themes.gohugo.io). 

The Pickles theme, like many other Hugo themes, supports the display of LaTeX equations using the [KaTex](https://katex.org) fast rendering engine. The instructions, however, are hidden in the theme's demo page ([here](https://themes.gohugo.io//theme/hugo_theme_pickles/post/math-typesetting/)) so you have to look carefully.

The only thing that I find lacking in Hugo so far is the presentation of images and image galleries. I use a package called [Hugo Easy Gallery](https://github.com/liwenyip/hugo-easy-gallery), but I haven't customize it enough to make images look nice. This requires some more work.

### Tweaking the Hugo theme to support two blogging formats
I wanted a website that has two sections: a [regular blog](https://hhyu.org) for essays, and a [microblog](https://hhyu.org/micro) for informal notes. Many people like to have a section of tweet-like short content. But for me, microblog entries are not necessarily shorter. Rather, they are different from regular blog entries because they don't have titles. If I can come up with a title for something I write, it means that it's an article written to communicate an idea to an audience. It's displayed with the title and a short blorb, which invite the readers to dig further. But if I can't come up with a title, it means that it's unorganized thought, written mostly for myself. It's displayed without a title, without tags, and in entirety. It's intended to be read as a person journal or a [commonplace book](https://en.wikipedia.org/wiki/Commonplace_book). 

Most Hugo themes don't have a microblog section, so I had to modify the Pickles theme. This requires understanding how Hugo organizes its content, and how the template system works. Hugo's template system is derived from the template library of the Go programming language. They can embed programming logic, which make them very powerful, but it also means there is a learning curve. I couldn't find a tutorial that explained the general idea to me, until I found a book called _Build Websites with Hugo_ by Brian Hogan. It's a little book that explains how to design a Hugo theme from scratch. Once I understood the logic behind the Hugo template system, it was surprisingly easy to tweak the Pickles theme to support the microblog format.

In short, a Hugo website can have multiple "sections", which correspond to directories in the `content/` directory. I have a `posts/` directory for the regular blog, and a `micro/` directory for the microblog. When the website is generated by Hugo, content in the former directory is displayed under `hhyu.org/posts`, whereas content in the latter is displayed under `hhyu.org/micro`. All I had to do was to give these two sections different HTML templates. It is accomplished by adding a new template file called `section.html` under `themes/pickles_modified/layouts/_default/`. This is what's in the file:
``` go
{{ if eq .Section "posts" }}
    {{ .Render "list" }}
{{ else if eq .Section "micro" }}
    {{ .Render "list_micro" }}
{{ end }}
```
It's easy to understand. If the reader requests `hhyu.org/posts`, use the `list.html` template; but if the reader requests `hhyu.org/micro`, use the `list_micro.html` template instead. `list.html` is part of the Pickles theme. I just had to modify it, and save it as `list_micro.html`.

The first time I tried to implement this idea in 2008, I decided not to modify the Pickles theme directly. Instead, I overrode some of Pickles' behaviors by saving my own templates under the `layout/` directory of my website's source code. After a while, I realized that it was easier for me to create my own version of the Pickles theme. This was done by _forking_ Pickles source code on GitHub. My forked version of the theme can be found [here](https://github.com/hsinhaoyu/hugo_theme_pickles) on GitHub.

### Hosting static websites on AWS [S3](https://aws.amazon.com/s3/)

The next question: How do we show the Hugo generated website to other people? Hosting HTML files is one of the most basic web services, which can be done in numerous ways. I decided to use S3, the cloud object storage service of AWS. It's an overkill for my needs, but it's cheap, widely used, and well integrated with AWS' other services.

Hosting on S3 is simple: you first create a storage space called a "bucket", upload the Hugo-generated files into the bucket, and then make the bucket accessible to everybody. You can follow [this document](https://docs.aws.amazon.com/AmazonS3/latest/dev/WebsiteHosting.html).

Since I need to upload to S3 every time I update my website, it's worthy to do it from the command line instead of the web-based AWS console. For that, I downloaded the [AWS command line interface](https://aws.amazon.com/cli/). After configuring it with my login information and the bucket's location, I can synchronize Hugo's generated website with the S3 bucket using a simple `sync` command.

If you are new to AWS, this might be a good time to familiarize yourself with AWS's [IAM](https://aws.amazon.com/iam/) (Identity and Access Management) service. In short, when you create an account on AWS, that account is the root user account, which allows you unlimited control of your AWS services. It's good practice to create an additional IAM "role" and log in as an IAM user. It's safer than logging in as the root user.

### Getting a custom domain name
The problem with hosting on S3 was that the website had to be accessed via a web address (i.e., URL) provided by the S3 server. I wanted it to be something simpler and personal. For that, I bought my domain name `hhyu.org` from [hover.com](https://www.hover.com) - one of the more popular registrars, but there are many options to choose from. One noteworthy point is that although I said that I "bought" my domain name, I don't actually own it. It's more like a lease, so I have to renew my domain name every year.

Owning a domain name sounded like a big deal to me in th beginning. I didn't look into building my own site for many years, because I thought that only professional web developers deal with registrars. But in reality, it's very simple. I logged on to my registrar, and told it to forward all "http://www.hhyu.org" traffic to the address assigned to my bucket by AWS. That's it. An interesting thing that I didn't know when I started was that http://www.hhyu.org and http://hhyu.org are two separate addresses, so I had to tell hover.com to forward both to AWS. I thought that `hhyu.org` was an abbreviated form, which is translated to the full name `www.hhyu.org` by the browser.

### Directing traffic to my domain with [AWS Route 53](https://aws.amazon.com/route53/)
With forwarding, people could visit my website with my domain name `http://hhyu.org`, but the address displayed on their browser was still the AWS address. That seemed wrong to me. I don't want my readers to feel that they started at `hhyu.org` but were transported to a different place. In other words, what I wanted was "address masking" rather than "forwarding". 

There are different ways to achieve this goal, but since I use AWS S3 for hosting, I decided to use another AWS service called Route 53. The process is a little technical, involving editing something called DNS records, but it was just a matter of filling up some forms. I followed the instructions of [this tutorial](https://aaronshaver.medium.com/setup-a-custom-domain-on-aws-using-s3-static-website-hosting-and-route-53-1b30eda0d4c1). What got me stuck for a while was I didn't understand the logic behind it. The basic idea is this: DNS (Domain Name Service) is the mechanism for translating domain names such as `hhyu.org` to the server on the Internet. My registrar's DNS only does forwarding, but I can tell it to use Route 53 instead. Once I allowed Route 53 to provide the DNS service, it can make it look like my website and my domain name are one and the same.

### Securing content distribution with [AWS CloudFront](https://aws.amazon.com/cloudfront/)
The S3 hosting service is convenient, but it serves the web pages with the HTML protocol. It's an unencrypted protocol that can easily be exploited. To increase the security of my site, I'd like to use the HTTPS protocol instead. Within the AWS ecosystem, this can be done with CloudFront. I followed this [tutorial](https://www.davidbaumgold.com/tutorials/host-static-site-aws-s3-cloudfront/). There are two major steps: First, I had to create a "distribution" of my bucket. This way, the content of my site is send to AWS servers all over the world, to reduce load time. Second, I needed an ACM security certificate to distribute the distribution using the encrypted HTTPS protocol. Getting a certificate sounded like a difficult process, but it really was just a matter of sending a request on AWS. It was granted in less than an hour. One problem that I encountered was that I didn't know that I had to switch my region to North Virginia when I requested a certificate. 

Big distribution networks like CloudFront move slowly, which means that after I update the content of the  bucket, it can take several hours to a full day for CloudFront to update my distribution. This isn't a big issue for simple websites like mine, but it is possible to _invalidate_ the distribution, to force CloudFront to update it. Invalidation can be initiated with CloudFront's web-based console, or with the `create-invalidation` command in the AWS commandline tool.

### Building a deployment pipeline with [GitHub Actions](https://github.com/features/actions)
At this stage, I have to manually use Hugo to generate the website, and sync the new website to S3 from the command line. This process is not very tedious, but I still wanted to automate it. For a casual blogger like me, reducing the friction in blogging is important to keep me going. Hugo has its own [deployment mechanism](https://gohugo.io/hosting-and-deployment/), but I decided to use GitHub as the mechanism for deployment. The rationale will become more obvious in the next section.


[GitHub](https://www.github.com) is a cloud service for programmers to share their source code. To maintain a copy of my website's source code on GitHub, I use a version-control system called [git](https://en.wikipedia.org/wiki/Git) to track the changes made to my website. Once I am happy with a new edition of my website (which is typically after I add a new post), I can "commit" the edits and then "push" the source code to GitHub. Git is a very complicated program that takes time to learn, but the advantage is enormous: First, I always have the source code of my website backed-up in a  GitHub _repository_. Second, git maintains the entire history of my website, so I can checkout past editions of the website easily if needed. Third, thank to the branching feature of `git`, it's safe to tinker. If I want to add a new feature to my site, I can create a parallel universe of the site called a "branch", and experiment in it as much as I want without affecting my website. When the new feature is ready, it can be "merged" into the main branch of the source code. If the new feature doesn't work as planned, nobody will see it. And fourth, `GitHub` opens the door to automation.

The key to building a deployment pipeline around GitHub is a software engineering concept called Continuous Integration (CI). What it means is that once a new version of a piece of software is finalized, an automated process should check for bugs, and prepare it for deployment. For our purpose, it means that that once the source code of a new version of the website is sync'ed to GitHub, a script will automatically regenerate the website with Hugo, and deploy it to AWS. [Travis-CI](https://travis-ci.com) is a popular third-party service that can integrate with GitHub, but I decided to try GitHub's own automation tool called Actions. Because all the necessary actions have already been developed by the GitHub community, it was very easy for me to glue them together. [This](https://github.com/hsinhaoyu/hhyu_site/blob/master/.github/workflows/main.yml) is the script that I am using. It resides in the `.github/workflows` directory under my site's root directory. The script generates the website, syncs with S3, invalidates the Cloudfront distribution, and sends me a Slack message on to tell me that it's all done. I can openly share this script with you, because all authentication information is encrypted in GitHub as [repository secrets](https://docs.github.com/en/free-pro-team@latest/actions/reference/encrypted-secrets), so they don't have to be revealed in the script.

With this setup, after writing a new blog post, all I have to do is to sync with GitHub. GitHub Actions takes care of the rest. 

### Posting on the go with a mobile GitHub client
I don't have a lot of opportunities to blog from my laptop (such is the life of a dad with a young child), so I really needed a way to blog from my phone. The problem was that Hugo doesn't run on phones. For a long time, I thought that there was no way to solve this problem... until I found [this article](http://evanbrown.io/post/hugo-on-the-go/). Since Hugo is now running on GitHub rather than my laptop, all I have to do is to use a GitHub client for the phone to post to my website. I use a program called [Working Copy](https://workingcopyapp.com/) for the iOS. It's a little expensive, but it's a very well developed app. I can create a new Markdown file in the `content/` directory of my website's repository, commit the changes, and push to GitHub, and then the website will be regenerated and deployed. 

Note that since I still use my laptop to update my website, it's important to make sure GitHub Actions uses the same version of Hugo that I use on my laptop.

### Improving the blogging experience with [Micropub](https://indieweb.org/Micropub)

The solution above does work, but GitHub is, after all, a programmers' tool rather a blogging tool. The interface is a bit unnatural for blogging. For a better experience, I turned to [Micropub](https://indieweb.org/Micropub) - a W3C recommended open standard for posting entries to websites. Since Micropub is an open protocol, a variety of Micropub client programs have been developed to post to any website that accepts the Micropub protocol. I decided to use [Quill](https://quill.p3k.io), a popular web-based client, although [other options](https://indieweb.org/Micropub/Clients) are available. Below are the steps that I took for my site to work with Micropub clients. 

I think it's worthy to point out here that Micropub is part of a larger movement called [IndieWeb](https://indieweb.org). IndieWeb is an alternative approach to social networking. In contrast to corporate-owned services such as Facebook or Twitter, IndieWeb websites are owned by individuals, and they interact with each other using open protocols. It is an exciting movement that deserves a more in-depth discussion (you can start [here](https://www.smashingmagazine.com/2020/08/autonomy-online-indieweb/)), but for the purpose for this tutorial, I will concentrate on blogging with Micropub. The IndieWeb community has developed many components that can work with static websites, so it's a great source of inspiration for me. I plan to dive deeper into IndieWeb in the future.

### Configuring an [IndieAuth](https://indieweb.org/IndieAuth) endpoint for authentication 
To blog from a Micropub client program such as Quill, the first issue was to tell the program who I was. I thought I had to create an account on Quill, but the IndieWeb world has its own way of doings things. When I got started, I thought that IndieWeb's protocol for establishing identity, called [IndieAuth](https://indieweb.org/IndieAuth), was confusing, but it makes lots of sense after I got used to it.

The central philosophy is that my identity on the web should belong to me. Web services should come to my site for all information about me. I shouldn't need to create different accounts for different social networks, and I shouldn't need to use their methods to authenticate myself. This means that to log in to an IndieWeb-based service, the only information that I need to provide is the address of my website. From my website, the service can extract the address of an _authorization endpoint_. The service then negotiates with my authorization endpoint to establish my identity.

In theory, this authorization endpoint should be a server program that I write myself to implements _my_ policies. For example, if somebody uses my website to log in to Quill, my authorization endpoint might ask for a password, or even a retinal scan. It can be anything that I consider sufficient to identify myself. But of course, I didn't actually have to implement one myself, because the community has done the hard work. For example, [IndieAuth.com](https://indieauth.com) is a popular authorization endpoint. The way it works is that I can ask IndieAuth.com to go to one of the web services that I already use to authenticate me. Since I use GitHub a lot, I asked IndieAuth.com to delegate authorization to GitHub. If I can convince GitHub that I am the person who I claim to be, IndieAuth.com will also report to any service that I am that guy. Note that IndieAuth.com is one particular implementation of the IndieAuth protocol. The protocol itself is more general.

[This](https://indieauth.com/setup) is the instructions for setting up for IndieAuth.com. I also found [this tutorial](https://www.amitgawande.com/2018/02/10/204300.html) useful. First, on my site where I link to my GitHub account, I added the `rel=me` attribute to the `<a>` tag to confirm to GitHub that `https://hhyu.org` is my site. For the theme that I am using, I had to edit the `layouts/partials/links.html` template, so that in the generated HTML, the link to GitHub looks like this:
```html
 <a href=https://github.com/hsinhaoyu target=_blank rel=me>
```

Second, I added the followings to the `<head>` section of my site. For the theme that I use, I had to create the `layouts/partials/head_custom.html` template, but it depends on how the theme is implemented. 

```html
<link rel="authorization_endpoint" href="https://indieauth.com/auth">
<link rel="token_endpoint" href="https://tokens.indieauth.com/token">
<link rel="micropub" href="https://xxxx.execute-api.ap-southeast-2.amazonaws.com/Prod/">
```

The first tag declares that I opt to use IndieAuth.com to be my authorization endpoint. The second tag declares a second authentication-related service that will be explained in a later section. With these two tags, I can already log on to a number of IndieWeb services without creating separate accounts. For example, I can use the [IndieWeb wiki](https://indieweb.org) to participate in community discussions, and [IndieBookClub](https://indiebookclub.biz) to keep track of the books that I am reading. That's pretty cool, but it's not enough for Micropub clients such as Quill, because the Micropub endpoint, declared in the third tag, had not been implemented yet. We'll deal with that in the following sections.

A side note: so far, I am identified to the IndieWeb world as `https://hhyu.org`, the URL of my site. For social networking, that seems a bit too impersonal. It would be better if I could associate it with my name and perhaps other personal information, so that they can be extracted and displayed by IndieWeb services. The mechanism is called [`h-card`](https://indieweb.org/h-card), which allows a wide range of personal information to be tagged. My h-card is minimal. I edited the footer template (`layouts/partials/footer.html`) so that my name is in the h-card:

```html
<a class="h-card" href="https://hhyu.org">Hsin-Hao Yu</a>
```
[indiewebify](https://indiewebify.me) is a very useful tool for (among many things) checking if the `h-card` information is tagged correctly. 

### Customizing a Micropub server for [AWS Lambda](https://aws.amazon.com/lambda/) 
We took a detour to IndieAuth. Let's return to the problem at hand. The goal is to create a pipeline such that after I finish writing a new blog post using a Micropub client, it sends the new post to a Micropub endpoint (specified by the `<link rel="micropub"...>` tag discussed previously). The server program at the endpoint commits the post to the GitHub repository, and GitHub Actions will generate the website, and deploy it to AWS. 

I had no prior experience in programming or setting up servers, so implementing this idea was a little intimidating to me. Luckily, the IndieWeb community has developed several Microbpub servers, so I didn't have to write one from scratch. I started with Cole Lyman's [gozette](https://github.com/Colelyman/gozette) because the source code (written in Go) is concise and easy to understand. In addition, it's designed to be deployed on [AWS Lambda](https://aws.amazon.com/lambda/) - a "serverless application service" which takes away some of the hurdles associated with server programming. In this setup, AWS handles the interface to the web. All we have to do is provide a function (which is called a "lambda" in programming) to interpret the incoming requests and acts on them. More concretely, when AWS receives a request to post from my Micropub client, it sends the post to `gozette`, which commits it to GitHub. `gozette` merely functions as a bridge between AWS's web gateway and GitHub. This is why the code is so lightweight.

I forked `gozette`'s repository on GitHub so I could have a [personalized copy](https://github.com/hsinhaoyu/gozette-hhyu). I  customized the code slightly for my particular setup:

1. The file `validation.go` is all about authentication with IndieAuth.com. I assigned the constant `indieAuthMe` to the URL of my site `https://hhyu.org`.
2. The file `github.go` interfaces with GitHub. I assigned variables `sourceOwner`, `authorName`, `authorEmail` and `sourceRepo` my own information. The most important one is `sourceRepo`. It is the name of the GitHub repository of my website.
3. The original `gozette` commits a new post to the `site/content/micro/` directory in the GitHub repository, but I needed it to be committed to `content/micro/`. This can be changed by modifying the `WriteHugoPost()` function in `post.go`.
4. Since `gozette` will modify my GitHub repository, it needs a key (called a _token_) to communicate with GitHub. I went to `Personal Settings` of my GitHub profile, and used `Developer settings > Personal access tokens` to generate a token. It's a string of random numbers and letters. I added the section below to `template.yaml`. When `gozette` is running on AWS Lambda, this is how it convinces GitHub that it is acting on my behalf.
```yaml
Environment: 
    Variables:
        GIT_API_TOKEN: <insert token here>
```

There is how to work with my version of `gozette`: 

1. Clone the [repository](https://github.com/hsinhaoyu/gozette-hhyu)  under `~/go/src/`. The Go compiler does care about where the source is located, so I recommend that you don't put it anywhere else.
2. Download the dependent libraries using the `go get` command. You have to issue these command under `~/go/src/gozette-hhyu/`.
```bash
go get github.com/speps/go-hashids
go get golang.org/x/oauth2
go get github.com/aws/aws-lambda-go/events
go get github.com/aws/aws-lambda-go/lambda
go get github.com/google/go-github/github
```
3. Modify the source code if needed.
4. Compile the code. Under `~/go/src/gozette-hhyu/`, issue the following commands:
```bash
go mod init gozette/main
go install ./gozette
```
If you don't see any error message, `gozette` is ready to run on AWS Lambda.

### Testing the Micropub server locally with [SAM](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-reference.html#serverless-sam-cli)

Before I uploaded `gozette` to AWS Lambda, I wanted to make sure that it ran correctly on my own laptop. The good news is that AWS provides a tool called [SAM](https://aws.amazon.com/blogs/aws/new-aws-sam-local-beta-build-and-test-serverless-applications-locally/) for this purpose. When I first started to try SAM, I was a little intimidated by its complexity. I decided to first try a simple "Hello World" example. In `~/go/src/`, I asked SAM to create a demo project in Go with
```bash 
sam init --runtime go1.x --name sam-go
```
I followed the instructions in README.md, and made sure that I knew enough to deploy a simple function to AWS Lambda.

Now go back to `~/go/src/gozette-hhyu/`. We can start the server locally with

```bash
sam build
sam local start-api
```
As soon as the server is started, a local IP address (such as `http://127.0.0.1:3000/`) is provided by SAM. That is the address of the endpoint. 

I needed a way to talk to it. Instead of using an actual Micropub client, I found it easier to use a utility program called [`curl`](https://curl.se).

To make it work, we need to know more about the authentication process. As discussed above, I can log in to a Micropub client (e.g., Quill) using the authorization endpoint IndieAuth.com. The question is: how does the Micropub endpoint (e.g., `gozette`) know that the client is actually acting on my behave? What happens if another person tries to post to my Micropub endpoint? The answer is that after successful authentication, the client passes an "access token" to the Micropub endpoint. Importantly, the client doesn't generate this token itself. Instead, it delegates the job to another endpoint, called the [token endpoint](https://indieweb.org/token-endpoint). When the Micropub endpoint receives the access token, it contacts the token endpoint to verify that the token is valid. The HTML code provided above shows that I opt to use `https://tokens.indieauth.com/token` to be my token endpoint, although other options are also available. Note that in this setup, the Micropub endpoint doesn't need to know about the authorization endpoint. This is why in `gozette`'s source code, there is no reference to the authorization endpoint, but I had to provide the token endpoint.

To communicate to the locally running `gozette` with `curl`, an access token is needed. It's not trivial to find it, because it's not normally exposed to the end-user. Luckily, Sebastiaan Andeweg's [Gimme an Access Token webapp](https://gimme-a-token.5eb.nl) provides this function. After obtaining a token, I could request `gozette` to commit a new post to GitHub with the following command:

```bash
curl \
-X POST \
-H "Content-Type: application/json" \
-H "Authorization: Bearer XXXX" \
-d '{"content":"hello world","name":"new post", "category": ["tag1", "tag2"], "mp-slug": "first_post"}' \
http://127.0.0.1:3000/
```
Remember to replace `XXXX` with the access token, and `http://127.0.0.1:3000/` with the address returned from SAM. If you see a new file called `content/micro/first_post.md` in your GitHub repository, it means that `gozette` is ready to be uploaded to AWS Lambda. During debugging and testing, I made `gozette` commit to a test repository, rather than the repository of my Hugo site.

### Running the Micropub server on AWS Lambda
Now it's time to deploy `gozette` to AWS Lambda. Under `~/go/src/gozette-hhyu/`, run
```bash
sam deploy --guided
```
If everything works, SAM will return an "API Gateway endpoint URL". This is the AWS address that `gozette` will receive requests from. For example, I received `https://xxxxx.execute-api.ap-southeast-2.amazonaws.com/Prod/` (remember that this was the URL that I assigned to the `<link rel="micropub">` tag in the `<head>` section.). I tested it with the same `curl` command above, except that I replaced the local address with the AWS endpoint address, and I verified that a new file was committed to GitHub. So `gozette` is now running correctly on AWS Lambda. 

I had planned to redirect `hhyu.org/micropub` to the AWS API gateway URL. This can be done with S3's redirection rule, but this approach didn't seem to work with CloudFront.
