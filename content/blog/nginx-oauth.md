---
title: "Implementing Discord OAuth in NGINX"
date: 2022-11-18T15:38:51-05:00
draft: false
---

Implementing Discord OAuth2 NGINX using the NJS module

I recently began offloading the authentication and authorization portion of my self-hosted applications to third party services by way of OAuth. This post is about how I implemented the OAuth2 token issuance logic in an application-agnostic manner, fully in NGINX.

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



Useful links:
* https://www.nginx.com/blog/validating-oauth-2-0-access-tokens-nginx/
* https://nginx.org/en/docs/http/ngx_http_js_module.html
* https://github.com/nginx/njs-examples#https-fetch-example-http-certs-fetch-https
* Use auth_request to on page endpoints to validate the provided token
    * https://nginx.org/en/docs/http/ngx_http_auth_request_module.html

TODO:
* Implement token refreshing
    * See below link for oauth2 refreshing
    * https://discord.com/developers/docs/topics/oauth2#authorization-code-grant-refresh-token-exchange-example
