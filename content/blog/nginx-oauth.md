---
title: "Implementing Discord OAuth in NGINX"
date: 2022-11-18T15:38:51-05:00
draft: false
---

You have your own NGINX instance serving content to the internet. This is great.

Now you need to secure your private content. You use `auth_basic` to implement basic HTTP authentication.

This is an annoying process and, depending on the browser, requires multiple steps with unavoidable (?) wait times in between.

Enter OAuth – a standardized way to offload your authorization concerns to a third party. In this case, I implemented Discord OAuth logins for our upcoming browser-based party game, [Lie to Me](https://lietome.genieindex.ca).

I recently began offloading the authentication and authorization portion of my self-hosted applications to third party services by way of OAuth. This post is about how I implemented the OAuth2 token issuance logic in an application-agnostic manner, fully in NGINX, with the help of the NJS module.

If you are not familiar with OAuth, I will demonstrate it below with an example scenario.

Client wishes to authorize their discord credentials to log into example.com
Usually, the client will click on some login link which will initiate roughly the following flow:

1. Client navigates to a third party provider login page with required parameters
    * In the case of discord, this contained:
        * client_id
        * redirect_uri (must be registered for your discord application in the developer portal)
        * response_type
        * scope
    * discord provided a tool to generate this URL
2. Client successfully logs in to the third party login page and clicks “Authorize”
3. Client is redirected to example.com (according to the preset redirect URL) with a code provided as a query parameter
4. Server retrieves token from the token endpoint using a number of parameters along with the code from the previous step, returns it to the client

This is trivial enough to implement in the application layer, but I wanted to avoid this repetition of code between my projects.

I set out to implement this flow in NGINX. To do this, I need to accomplish a number of things:

1. An endpoint that will retrieve the code returned to the client by the discord login page
2. An endpoint that will subsequently be used to retrieve the token using the code from the previous step
3. and the tricky part is here – the response we are looking for (the access token) is returned as JSON in the response body of (2)
    * we need to parse the response body and save the values we are looking for
4. in a future subrequest, set the appropriate response headers so the client saves the tokens as session cookies – making them available to my web applications served from behind nginx

After much trial and error, I came upon a method that works for me.
Issues I ran into along the way included “tried to make http response from https endpoint” or some shit and another annoying one that was incoherent – the body was just gibberish. Straight up gibberish. Sometimes I would include the `gunzip on` directive and it would change the gibberish, but it was still gibberish. I still have not figured out exactly what did it in the end, but I realized I could not examine the body in certain situations, and also I needed to set the “Accept-Encoding” to “” in order to prevent the discord API from returning anything compressed. Some combination of those things did the trick, and that’s good enough for me. I’ll include all my config somewhere.

[This guy](https://stackoverflow.com/questions/63273312/nginx-subrequest-response-encoding-issue) seems to have run into the same issues as well as [this](https://stackoverflow.com/questions/44855538/gibberish-instead-of-html-response-as-body-in-node-request) dude

also ran into connect() failed (111: Connection refused) while connecting to upstream ipv6
The parts of interest:

The logic in Javascript implemented in the application layer
    console.log(`Found token ${TOKEN}`);

    // If there is a code provided, try to authenticate with it
    const code = url.searchParams.get('code');

    if (code) {
        console.log(`authorizing with code ${code}`);

        let body = new URLSearchParams();
        body.set('client_id', '###################')
        body.set('client_secret', '################################')
        body.set('grant_type', 'authorization_code')
        body.set('code', code)
        body.set('redirect_uri', 'https://example.com/')
        const res = await fetch("https://discord.com/api/v10/oauth2/token", {
            method: "POST",
            headers: {
                "Content-Type": "application/x-www-form-urlencoded",
            },
            body: body,
        });
        let data = await res.json();

        console.log(data);

        if (data.access_token && data.refresh_token) {
            cookies.set('discord_access_token', data.access_token);
            cookies.set('discord_refresh_token', data.refresh_token);
        }

        throw redirect(302, '/');
    }

    console.log("Retrieving saved token values");
    console.log(cookies.get('discord_access_token'));
    console.log(cookies.get('discord_refresh_token'));

---
WIP:

Useful links:
* https://www.nginx.com/blog/validating-oauth-2-0-access-tokens-nginx/
* https://nginx.org/en/docs/http/ngx_http_js_module.html
* Use auth_request to on page endpoints to validate the provided token
    * https://nginx.org/en/docs/http/ngx_http_auth_request_module.html
TODO:
* Implement token refreshing
    * See below link for oauth2 refreshing
    * https://discord.com/developers/docs/topics/oauth2#authorization-code-grant-refresh-token-exchange-example

More links from safari

* THE STARTING POINT: https://gist.github.com/Paturages/9da505dd5c6fde93b53d605eeb4ec12e
    * WHICH did not work

all this was an alternative to using https://www.passportjs.org middleware like this

* oauth2 resources from discord https://discord.com/developers/docs/topics/oauth2#authorization-code-grant
* oauth2 url generator from discord https://discord.com/developers/applications/1042267625692082236/oauth2/url-generator
* NJS example code https://github.com/nginx/njs-examples#hello-world-example-http-hello
                   https://github.com/nginx/njs-examples#https-fetch-example-http-certs-fetch-https
                   https://github.com/nginx/njs-examples
* using sub request authentication in NGINX https://gock.net/blog/2020/nginx-subrequest-authentication-server/
* looking up errors that were got https://www.google.com/search?hl=en&q=connect()%20failed%20(111%3A%20Connection%20refused)%20while%20connecting%20to%20upstream%20ipv6
* simple load balancing and sub request auth with nginx https://medium.com/swlh/simple-http-load-balancing-and-subrequest-authentication-with-nginx-b3a6d9cfa6c0
* https://stackoverflow.com/questions/72380200/nginx-not-sending-headers-or-variables-to-js-content-inside-auth-request
* https://stackoverflow.com/questions/53380843/oauth-unsupported-grant-type-discord-api

* https://github.com/nginx/njs/issues/136

* https://nginx.org/en/docs/njs/reference.html#r_request_buffer
* https://nginx.org/en/docs/njs/reference.html#ngx_fetch
* https://nginx.org/en/docs/njs/reference.html#r_request_buffer
* https://docs.nginx.com/nginx/admin-guide/web-server/compression/
* https://nginx.org/en/docs/http/ngx_http_js_module.html#js_include
* https://nginx.org/en/docs/njs/reference.html
* https://nginx.org/en/docs/http/ngx_http_js_module.html
* https://nginx.org/en/docs/http/ngx_http_auth_request_module.html
* https://nginx.org/en/docs/http/ngx_http_js_module.html#js_header_filter
* https://nginx.org/en/docs/http/ngx_http_proxy_module.html
* https://nginx.org/en/docs/http/ngx_http_js_module.html#js_body_filter
* https://nginx.org/en/docs/http/ngx_http_core_module.html#var_sent_http_
* https://nginx.org/en/docs/http/ngx_http_core_module.html#var_http_
* https://nginx.org/en/docs/http/ngx_http_upstream_module.html#var_upstream_http_
* https://nginx.org/en/docs/varindex.html
* https://nginx.org/en/docs/http/ngx_http_sub_module.html
* https://nginx.org/en/docs/http/ngx_http_js_module.html#js_body_filter

* Converting response body characters to lower case https://clouddocs.f5.com/training/community/nginx/html/class3/module1/module16.html

Future reading
* https://liamhieuvu.com/setup-nginx-authentication-and-discord-alert-for-full-nodes

* Get query parameter in nginx https://stackoverflow.com/questions/26133592/how-to-get-query-parameter-in-lua-or-nginx

* Validating oauth 2 tokens with nginx: https://www.nginx.com/blog/validating-oauth-2-0-access-tokens-nginx/

* lua nginx module that I did not use in the end https://github.com/openresty/lua-nginx-module

* set cookie in nginx https://www.digitalocean.com/community/questions/can-you-set-up-cookies-in-nginx

* dead leads https://stackoverflow.com/questions/58311270/how-to-add-the-content-of-request-body-as-response-header-in-nginx
    * https://github.com/nginx/njs/issues/488