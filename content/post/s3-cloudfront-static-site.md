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

### Broken as of 24-02-2020

I can't get to my `/about/` and `/links` paths because the routing doesn't
know what `/about/index.html` is. This is a known behaviour of CloudFront. From
my brief reading into solutions, it seems that I can either implement a LambdaEdge
function to enrich request routing or enable `uglyurls` in Hugo. It just works
when you're accessing the site directly from S3, so whatever I do I'll need to
do with CloudFront. I'll update this when I fix the issue.
