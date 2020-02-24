---
title: "S3 Cloudfront Static Site"
date: 2020-02-24T11:05:45Z
draft: false
---

Confession: I struggle with AWS services, in that I don't find them that
intuitive to use, and mostly default to bouncing around trying to figure out
which switches and dials to tweak in order to derive a desirable result.

That said, the complexity results in the ability to build some powerful services
pretty darn quickly when compared with attempting to orchestrate similar solutions
on your own.

My most recent foray into AWS (outside of the office) was getting this site running.
I'd previously hosted iterations of it on a Digital Ocean Kubernetes cluster,
but ultimately figured that spending in excess of $40 per month on a site that
nobody reads a bit silly. Next, I went for a GitHub Pages derivative. That
just works, but when considering things to play with the future, I figured that
I'd end up running some services out of AWS anyway...so why not just run the
site there?

The concept of running a static site on S3 is not new, nor is the idea of
fronting it with a CloudFront distribution. What I hadn't figured on was just
how much time I'd spend scratching my head saying "wut?" while getting it going.

Here's what I ended up with.

### Route53

This is relatively simple. Once you have CloudFront configured, you're going to
be using `alias` records which point your apex and a `www` record to CF.

### S3

I created two buckets. One is called `www.nosignal.io` and simply redirects traffic
to the primary bucket that contains the code, called `nosignal.io`.

Under properties for `www.nosignal.io`, **Static website hosting** is enabled,
with **Redirect requests** set to:

* `Target bucket or domain: nosignal.io`
* `Protocol: https`

The properties for `nosignal.io` have **Use this bucket to host a website** set,
with the `index document` and `error document` properties set.

Under permissions (for both buckets), I have **Block all public access** enabled.
The only service that I want to be able to access my content is CloudFront. This
should reduce the risk of any silliness whereby content could be pulled from S3
in sufficient enough volumes to cause me any financial misery, and means that
anything added to S3 is only accessible via the CDN that CloudFront provides.

Under **Permissions**, I have the Canonical ID of the S3 Origin from CloudFront
set to **List Objects**. This effectively allows CloudFront read access to the
bucket.

Under **Bucket Policy**, I have the following:

```json
{
    "Version": "2012-10-17",
    "Id": "PolicyForCloudFrontPrivateContent",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::cloudfront:user/CloudFront Origin Access Identity XXXXXXXXXXXXXX"
            },
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::nosignal.io/*"
        }
    ]
}
```

### CloudFront

I opted for a single CloudFront distribution that uses all edge locations and has
`CNAMEs` configured for `nosignal.io` and `www.nosignal.io`. After multiple bites
of the cherry in getting AWS Certificate Manager to issue a cert and *then* use it
in my CloudFront distribution, I ultimately ended up using the Distribution creation
wizard to request a certificate (also mapped to the apex and `www` names of my site).
Another tweak was to set the Security Policy to TLSv1.2_2018 and above, as well
as set the `Default Root Object` to `index.html`.

Under **Origins and Origin Groups**, I have `nosignal.io.s3.amazonaws.com` and
`www.nosignal.io.s3.amazonaws.com` which map to their respective Origin Domain
Names and use an identity created (AFAIK) during the creation of the Distribution.

Withion **Behaviors** I have a single `Default (*)` behaviour that maps to the
S3 bucket of my apex domain (which contains all of the content).

It is under **Origin Access Identity** under CF that you can find the
`Amazon S3 Canonical User ID` I references under `S3 > Permissions` to allow
CF originated requests read access to the S3 buckets.

### Fixing Directory Indexing

When you direct requests to an S3 bucket from CloudFront, those requests don't
read the objects from your bucket, they're fetched from the S3 REST API. For this
reason, a request to `/about/` will read from `/about/index.html`, but fails to
render as CloudFront isn't reading the file, it's reading an `index.html` object
hanging off of the `about` object from an API request. To get around this, you
can either switch off the protections that make CloudFront the default read
resource for your bucket, or you can enable `uglyurls` in Hugo, or you can set up
a Lambda@Edge function to handle the request translation for you.

I opted for a Lambda@Edge function. The basic code (direct from AWS) looks as
follows:

```javascript
'use strict';
exports.handler = (event, context, callback) => {

    // Extract the request from the CloudFront event that is sent to Lambda@Edge
    var request = event.Records[0].cf.request;

    // Extract the URI from the request
    var olduri = request.uri;

    // Match any '/' that occurs at the end of a URI. Replace it with a default index
    var newuri = olduri.replace(/\/$/, '\/index.html');

    // Log the URI as received by CloudFront and the new URI to be used to fetch from origin
    console.log("Old URI: " + olduri);
    console.log("New URI: " + newuri);

    // Replace the received URI with the URI that includes the index page
    request.uri = newuri;

    // Return to CloudFront
    return callback(null, request);

};
```

It simply plugs `index.html` onto the end of a request so that the required
object can be rendered in the browser.

To set one up, make sure that your region is **US East (N. Virginia)**. Any other
region won't show the required CloudFront trigger. Head over to Lambda and
create a new function. Choose **Author from scratch**, name your function, choose
**Node.js 10.x** as your runtime and then select **Create a new role from AWS policy
templates**. Give your new role a name, and then select **Basic Lambda@Edge permissions
(for CloudFront tigger)** and select **Create Function**.

Once the designer screen opens, click **Add trigger** and select **CloudFront**. Then
select **Deploy to Lambda@Edge**. This will create a layer in the Designer view
named after your function. Paste the code above into the `index.js` editor
window and save your code. You can then click on the **Actions** drop down and
select **Deploy to Lambda@Edge**. You may as well wander off at this point,
as your CloudFront Distribution is kicking off an update. As soon as your
Distribution has been updated, your site should be working as expected.

### In summary

It took me a couple of hours to figure out S3 + AWS Certificate Manager +
CloudFront + Lambda@Edge. Getting this done as a GitHub Pages site with an SSL
cert mapped to my custom domain takes a few minutes. Even getting this done on a
three node k8s cluster on DigitalOcean with some scripts to turn a `hugo` build
command into content on a custom Nginx image took a day of fiddling, and that
included writing all of the YAML, getting Nginx ingress working and hooking up
cert-manager.

So where's the benefit to this? My job here is now done. I have no further
maintenance to undertake. My site is deployed on the CDN of one of the largest
networks on the planet. It's delivered over HTTPS. I haven't had to open my
S3 bucket to the world. If I ever write something that ends up on the front page
of [Hacker News](https://news.ycombinator.com), I don't need to think about scale.
If I decided to muck around with some JavaScript to talk to backend services,
I have one of the largest toolboxes of toys to leverage. Going forward,
updates are as simple as `write` &rarr; `hugo server -D` &rarr; `hugo` &rarr; `hugo deploy`.
Probably.
