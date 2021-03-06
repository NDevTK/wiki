+++
title = "Cache Protections"
description = "Cache Protection"
date = "2020-10-16"
category = [
    "Defense",
]
menu = "main"
+++

There are a number of different approaches for defending against cache probing based XS-Leaks. 

# Cache Protection via `Cache-Control` Headers

If it is acceptable to disable caching, this provides a strong defense against cache probing attacks. This means that every time someone loads a resource, the resource will have to be fetched again. This can be done by setting a `Cache-Control: no-store` header on every single response that you wish to protect. 

Advantages:
1. Supported by all major browsers

Disadvantages:
1. Negatively impacts site performance

# Cache Protection via Random Tokens

Rather than disabling caching, one can include additional data in URLs in order to defend against cache probing attacks. This can be done by including a random token in the URL of every subresource that you reference. If an attacker cannot guess this random token, then the attacker cannot determine whether items are in the cache via any straightforward techniques. 

{{< hint info >}}
Suppose that every page on your application loads the user's profile photo: `/user/<USERNAME>.png`. An attacker could check which user is signed in by probing the cache for `/user/john.png`, `/user/jane.png`, and so on. 

This is where a random token can come into play. Instead, on every load the application can load the user's profile photo from `/user/<USERNAME>.png?cache_buster=<RANDOM_TOKEN>`. The server does not need to do anything with this random token. It is there purely to ensure that there is no way for an attacker to probe the cache without knowing the random token. 
{{< /hint >}}

If implemented carefully, an application could even have a user specific random token that is reused across page loads. This would allow subresources to still be cached since the URL would remain constant for a given user. 

Advantages:
1. Supported by every major browser
2. Does not break caching

Disadvantages:
1. Difficult to implement

# Cache Protection via Fetch Metadata

[Fetch-Metadata]({{< ref "../opt-in/fetch-metadata.md" >}}) is meant to allow servers to determine how and why a request was initiated on the client side. One piece of information that is exposed is the [`Sec-Fetch-Site`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Sec-Fetch-Site) header which specifies whether a request is coming from the same origin or a different origin. This can be combined with the [`Vary`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Vary) header in order to force the browser to segment the cache based off of whether a request is made from the same origin or a different origin. 

This is done by setting `Vary: Sec-Fetch-Site` on all resources you wish to protect. 

{{< hint info >}}
Assume we have the resource `cdn.example.com/image.png` that we wish to protect from cache probing attacks. If we set `Vary: Sec-Fetch-Site` on it,

1. If `example.com` tries to load the resource, the request is initiated by the same site so it is cached under `(SFS: same-site, resource_url)`
2. If `cdn.example.com` tries to load the resource, the request is initiated by the same origin so it is cached under `(SFS: same-origin, resource_url)`
3. If `evil.com` tries to load the resource, the request is initiated by a different site so it is cached under `(SFS: cross-site, resource_url)`

Note that this means cross-site requests are separated from same-site and same-origin requests. 
{{< /hint >}}

Advantages:
1. Does not break caching

Disadvantages:
1. Fetch metadata is a new standard that is currently only supported in Chromium-based browsers (e.g. Chrome and Edge)
2. Cross-site subresources loaded on the page will not be protected (e.g. subresources from CDNs)
3. If third parties load the resource, they will not be protected 