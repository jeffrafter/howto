# Shopify Application Proxy Sessions

**Problem:**

- Cookies are not passed for application proxy pages (eliminating cookie based sessions)
- There is no way in normal shopify pages to read params except via javascript 
- Params are lost after built-in redirects (eliminating url / param based sessions)



## Using the customer

* When a customer is created you respond to a webhook and generate a secret token
* All future interaction with the proxy passes the embedded customer_id and customer_token
* The customer_id and customer_token can be embedded in forms throughout the store and can also be injected as metafields making AJAX easier
* Serving get requests for pages means you have to add the parameters to the top url (which is ugly). This could be masked with a container page rendered as liquid and a sub page rendered with an iframe (same domain)

### Downsides

* requires customer accounts :(
* get requests require url params

### Upside

* Requires way less adjustments to the rest of the store

## Using a JavaScript cookie

1. Check for app session cookie
2. If cookie does not exist you post to an `/a/session` route to get a secret token in the response 
3. Secret gets stored in session cookie via javascript
4. Subsequent requests can fetch the cookie via javascript and append `<iframe>` with the session token as a parameter in the iframed url
5. As an alternative, the extracted session cookie could be used to append onto urls throughout the system (i.e., on product pages)


### Downsides

* requires javascript and cookie interaction
* requires a setup request
* requires multiple requests per page (if using iframes)