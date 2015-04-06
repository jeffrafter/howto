# Rails Security

* Run brakeman
* Keeping gems up to date (gemnasium)
* Rate limiting (rack attack)
* X-Content-Type headers (rack attack)
* CSP
* Reset the sessions/login
* Tracking the request
* Length masking
* Can a plugin limit SSL?
* Looking up tokens
* Responsible disclosure and security page
* Robots.txt
* Crossdomain.xml
* Preventing header injection
* Certificate pinning
* EU ePrivacy Directive / Do Not Track

## Run brakeman

## Keeping gems up to date using Gemnasium

## Rate limiting

https://www.kickstarter.com/backing-and-hacking/rack-attack-protection-from-abusive-clients

https://github.com/kickstarter/rack-attack

Things you should rate limit:

* Login attempts
* Forgot password tries (email enumeration)
* Forgot password token tries
* Forgot password submissions
* Token authentication misses
* OAuth misses
* Items which generate emails (header injections)
* Long running items (DOS)
* Clients getting errors

Things you should not rate limit via rack attack

* API requests

Note: remember that some web-servers are set to proxy their requests. Make sure your HTTP X Forwarded For is right.

### Be fail2ban friendly

How you raise errors should be friendly to fail2ban

Make a scanner-scanner?

## Headers

X-Content-Type-Options

## CSP

Use a content security policy which does not allow inline scripts and enumerates the possible asset sources. White-list third parties like Google Analytics.

https://github.com/blog/1477-content-security-policy

https://github.com/twitter/secureheaders

### Google Analytics and CSP
https://stackmachine.com/blog/google-analytics-and-content-security-policy

## Reset the sessions

You should be storing a list of all sessions that are logged in. In some cases all sessions should be reset:

* When the user changes the password
* When the user issues a forgot password request

Note this should also reset all existing password reset tokens.


## Tracking the request

https://github.com/efficiency20/ops_middleware

Dangers of adding tracking headers to the request (identify the server and protocol)

Legal requirements about tracking in Europe

## Length masking

BREACH and CRIME (breachattack.com)

breach and forgot-passwords (url encoded post reflected into body)

random length comment in each request
XOR masking on the CSRF token

https://github.com/meldium/breach-mitigation-rails

## Can a gem limit the kinds of SSL used

* Choose specific ciphers?
* Also tracking SSL cert expiry?

## Looking up tokens using params

* MySQL typecasting of packets [http://www.phenoelit.org/blog/archives/2013/02/05/mysql_madness_and_rails/index.html](http://www.phenoelit.org/blog/archives/2013/02/05/mysql_madness_and_rails/index.html)

## Setup a responsible disclosure policy

PGP email
Bug bounty
T-shirts
Respond immediately (example is the timing attack in Java)

## Robots.txt

## Crossdomain.xml


## Preventing header injection

http://homakov.blogspot.com/2014/01/header-injection-in-sinatra.html
https://gist.github.com/rkh/befd7d127f4052d292d5


## Certificate pinning

Are you consuming an API from your Rails app? How are you preventing man in the middle attacks?

* Cloak
* VPN
* Socks proxy
* Certificate pinning / Public Key pinning

For example: are you developing against the Shopify API? Are you doing that at this conference? How do you know someone isn't impersonating Shopify with a MitM.

Certificate pinning ensures against some privileged network position attacks.


# Classes of vulnerabilities

* Timing attacks
* Brute force protection
* Token leaking
* Open redirects (non-whitelisted)
* Concurrency issues (multiple avatar uploads http://homakov.blogspot.com/2014/11/hacking-file-uploaders-with-race.html)


# Legal

## EU ePrivacy Directive

https://github.com/peter-murach/rack-policy
http://en.wikipedia.org/wiki/Do_Not_Track (Roy Fielding)
https://ico.org.uk/for-organisations/guide-to-pecr/cookies/

## COPPA

http://www.coppa.org/comply.htm

## CAN-SPAM

http://www.ftc.gov/tips-advice/business-center/guidance/can-spam-act-compliance-guide-business

## PCI

https://www.pcisecuritystandards.org/

## HIPPA

http://www.hhs.gov/ocr/privacy/

## HITECH

http://www.hhs.gov/ocr/privacy/hipaa/administrative/enforcementrule/hitechenforcementifr.html

## FTC digital advertising disclosure (online advertising)

http://www.ftc.gov/sites/default/files/attachments/press-releases/ftc-staff-revises-online-advertising-disclosure-guidelines/130312dotcomdisclosures.pdf

## United Nations Convention on the International Sales of Goods

http://www.uncitral.org/pdf/english/texts/sales/cisg/V1056997-CISG-e-book.pdf
