---
title: "This simple package will protect your web app"
date: 2022-08-09T16:30:26+05:00
draft: false

description: "Helmet is a node package help secure Express apps with various HTTP headers "

featuredImage: "https://repository-images.githubusercontent.com/3329923/2fd1c70a-c521-4087-9c1d-bf3e1fff3e4d"
featuredImagePreview: "https://repository-images.githubusercontent.com/3329923/2fd1c70a-c521-4087-9c1d-bf3e1fff3e4d"

tags: ["web", "security"]
categories: ["WEB"]
theme: "full"
---

We will explain how to protect you MERN app by simply using the helmet package. But before we dive into this we should know what are http headers because helmet basically used for setting our http headers.

<!--more-->

## Http Headers

![headers](https://twilio-cms-prod.s3.amazonaws.com/original_images/hero-header.jpg)

An HTTP header is a field of an HTTP request or response that passes additional context and metadata about the request or response.

### Types

We can divide them into four types

#### HTTP Request Header

Whenever you type a URL into the address bar and try to access it, your browser sends an HTTP request to the server. This include info about clients browser, OS etc

- Type, capabilities and version of the browser that generates the request.
- Operating system used by the client.
- Page that was requested.
- Various types of outputs accepted by the browser.

#### HTTP Response Header

Upon receiving the request header, the Web server will send an HTTP response header back to the client. An HTTP response header includes information in a text-record form that a Web server transmits back to the client's browser.

The response header contains particulars such as the type, date and size of the file sent back by the server, as well as information regarding the server.

#### HTTP General Header

These headers contain directives that need to be followed, for both the requester and receiver. This can include information regarding:

- Caching directives.
- Specified connection options.
- The date (always listed in Greenwich Mean TIme)
- Pragma
- Upgrade (for if the protocols need to be switched)
- Via (to indicate intermediate protocols)
  Warning (for additional information not found elsewhere in the header. There may be more than one warning listed.)

#### HTTP Entity Header

These headers include information regarding:

- Allow (methods supported by the identified resource)
- Content Encoding.
- Content Language.
- Content Location.
- Content Length.
- MD-5 (for checking the integrity of the message upon receipt).
- Content Range.
- Content Type.
- When it Expires.
- When it was last modified.

### What are HTTP Security Headers ?

**The http security headers are basically the part of Http Response Headers.**
When we visit any website in the browser, the browser sends some request headers to the server and the server responds with HTTP response headers. These headers are used by the client and server to share information as a part of the HTTP protocol.

### Why HTTP Security Headers are necessary ?

As you know, nowadays too many data breaches are happening, many websites are hacked due to misconfiguration or lack of protection. These security headers will protect your website from some common attacks like **_XSS, code injection, clickjacking_**, etc. Additionally these headers increases your website SEO score.

## HELMET to the rescue

Helmet helps you secure your Express apps by setting various HTTP headers. You can take a look at their work [github/helmet](https://github.com/helmetjs/helmet).

Or you can simply install with npm

```
npm install helmet --save
```

### Using

Now simply require this package to your app

```js
const express = require("express");
const helmet = require("helmet");

const app = express();

app.use(helmet());

// rest of code
```

## Configuring with your app

Now you can set any headers via helmet easily.

### Default Settings

Helmet set the following headers in default

```json
Content-Security-Policy: default-src 'self';base-uri 'self';block-all-mixed-content;font-src 'self' https: data:;form-action 'self';frame-ancestors 'self';img-src 'self' data:;object-src 'none';script-src 'self';script-src-attr 'none';style-src 'self' https: 'unsafe-inline';upgrade-insecure-requests
Cross-Origin-Embedder-Policy: require-corp
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Resource-Policy: same-origin
Expect-CT: max-age=0
Origin-Agent-Cluster: ?1
Referrer-Policy: no-referrer
Strict-Transport-Security: max-age=15552000; includeSubDomains
X-Content-Type-Options: nosniff
X-DNS-Prefetch-Control: off
X-Download-Options: noopen
X-Frame-Options: SAMEORIGIN
X-Permitted-Cross-Domain-Policies: none
X-XSS-Protection: 0
```

## Make web-app secure

You can set the headers like this

```js
// This disables the `contentSecurityPolicy` middleware but keeps the rest.
app.use(
  helmet({
    contentSecurityPolicy: false,
  })
);
```

But the thing is that disabling this will cause many security issues thats why this is not recommended you have to set the headers according to you web-app domain.

For example for my app i use firebase for images. So setting helmet default setting i.e **simply using app.use(helmet())** will not allow my images to load correctly and like wise it will also block any other cross domain request

{{< figure src="https://i.stack.imgur.com/SJxVB.png" title="Blocked request due to helmet" >}}

So now we will try to configure it according to our site and to keep our site secure.

```js
app.use(
  helmet({
    contentSecurityPolicy: {
      directives: {
        defaultSrc: ["'self'"],
        scriptSrc: [
          "'self'",
          "'unsafe-inline'",
          "https://memobooks.herokuapp.com/",
          "https://firebasestorage.googleapis.com/",
        ],
        styleSrc: ["'self'", "https://fonts.googleapis.com", "'unsafe-inline'"],
        imgSrc: ["'self'", "https://firebasestorage.googleapis.com/"],
        fontSrc: ["'self'", "https://*.com", "data:"],
      },
    },
    crossOriginEmbedderPolicy: false,
    crossOriginResourcePolicy: { policy: "cross-origin" },
  })
);
```

Take a look at [helmet Documentation at github](https://github.com/helmetjs/helmet).

## Acknowledgement

These resources helped a lot in while writing this blog.

- [techopedia.com/definition/27178/http-header](https://www.techopedia.com/definition/27178/http-header)
- [loginradius.com/blog/engineering/http-security-headers](https://www.loginradius.com/blog/engineering/http-security-headers/)
