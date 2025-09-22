# Problem Set 2: Composing Concepts

## Concept Questions

1. The NonceGeneration concept ensures that the short strings it generates will be unique and not result in conflicts. What are the contexts for, and what will a context end up being in the URL shortening app?

It is possible for different contexts to have identical short strings, in this scenario, it is possible for different domains (base URLs) to have identical suffixes and have different target URLs. In the URL shortening app, a context would be a user-specified domain. 

2. Why must the NonceGeneration store sets of used strings? One simple way to implement the NonceGeneration is to maintain a counter for each context and increment it every time the generate action is called. In this case, how is the set of used strings in the specification related to the counter in the implementation? (In abstract data type lingo, this is asking you to describe an abstraction function.)

NonceGeneration must store sets of used strings so that when it generates a new string, it can check if that string had already been generated/used by checking the used strings set. In the given case, the counter is the concrete representation and the set of used strings is the abstract representation. Each abstract representation is mapped to an integer from 0 to n, where n is the current value of the counter. This can be done easily by just using the counter number as a suffix, so each newly generated nonce will have the counter number in the short suffix.

3. One option for nonce generation is to use common dictionary words (in the style of yellkey.com, for example) resulting in more easily remembered shortenings. What is one advantage and one disadvantage of this scheme, both from the perspective of the user? How would you modify the NonceGeneration concept to realize this idea?

One advantage is the URL becomes more accessible for the user. They can easily remember common dictionary words, so even if the user doesn't save or access the link immediately, all they have to do is memorize the common dictionary words and they can access the link themselves at a later time. One disadvantage is that the shortening using common words may be longer than an alternative shortening that just uses random alphanumerics, because there is no constraint on shortening length as opposed to words that have a defined length. To realize this idea, I would add a set of words to each Context in the state, and alter generate() to randomly pick words from that set.

## Synchronization Questions

1. In the first sync (called generate), the Request.shortenUrl action in the when clause includes the shortUrlBase argument but not the targetUrl argument. In the second sync (called register) both appear. Why is this?

The act of generating a nonce and the act of registering are independent and one does not necessarily link into the other, but in order to register, there needs to exist a shortURL using a generated nonce. The shortUrlBase is required by generate because the nonce generation step needs to check if the generated nonce already exists for the given context. Generate does not actually create the short URL, so there is no targetURL to link to. On the other hand, register actually creates the short URL and thus requires everything generate does, as well as the targetURL so that it can make a valid shortURL mapped to the targetURL.

2. The convention that allows names to be omitted when argument or result names are the same as their variable names is convenient and allows for a more succinct specification. Why isn’t this convention used in every case?

This convention grants brevity but may make the spec unclear in some cases. If an action takes the exact same variable twice, but as two different arguments, then this convention would be confusing, because two arguments can't have the exact same name, so it would be necessary to specify the argument name, then the variable name. 

3. Why is the request action included in the first two syncs but not the third one?

Setting the expiry is not contingent on the request happening exactly, but rather the URL being registered successfully. One issue with request is that a request can fail, but the sync description only cares when the action is completed, and a rejected request is a completed action. If it was tied to the request then the expiry would be set even if the registration failed, meaning there could be a scenario where there is not valid shortURL to set an expiry time for. So the setExpiry sync only happens when it is certain that the URL has been registered, and that comes from URLShortening, which should not fail if the request was valid. 

4. Suppose the application did not support alternative domain names, and always used a fixed one such as “bit.ly.” How would you change the synchronizations to implement this?

I would hard code "bit.ly" (or remove it entirely) for the shortUrlBase in Request.shortenUrl for generate, because there is now only one context every time, so there is no need to specify the context to look at when checking uniqueness of nonces. Same for NonceGeneration.generate. In register, I would do the same for shortUrlBase in the Request, and then also the same for UrlShortening.register.

5. These synchronizations are not complete; in particular, they don’t do anything when a resource expires. Write a sync for this case, using appropriate actions from the ExpiringResource and URLShortening concepts.

**sync** expire
**when** ExpiringResource.expireResource(): (resource: shortUrl)
**then** URLShortening.delete(shortUrl)

## Extending the Design

### Concepts

**concept** UrlOwnership 
**purpose** associate created Urls with their administrators/creators
**principle** Users create accounts, then authenticate to add links to their ownership
**state** a set of Users with a unique username, a password, a set of links
**actions** 

addUser(username: String, password: String): (User)
**requires** no User exists with the given username
**effects** returns a new User with the given username and password, and associates with an empty set of links

authenticate(username: String, password: String): (User)
**requires** there exists a User with the given username and password
**effects** returns the User handle, granting access

deleteUser(User, password: String)
**requires** given password matches password of the given User
**effects** deletes the User and its associated set of links

addLink(User, link: String)
**requires** given link is not already in given User's set of associated links
**effects** adds the given link to the set of links associated with the given User

deleteLink(User, link: String)
**requires** given link is in User's set of associated links
**effects** removes the link from the User's set

verifyOwnership(User, link: String): (Bool)
**effect** returns True if given User has the given link in their set of links, false otherwise




**concept** UrlAnalytics
**purpose** count the number of accesses to a link
**principle** allow users to see how many times a link has been used
**state** a set of Links with a count starting at 0
**actions**

init(link: String): (Link)
**requires** given link is not already initialized
**effect** returns a Link object with a count at 0

increment(Link)
**effect** increases the count for the given link by one

reset(Link)
**effect** set the count for the given link to 0

getCount(Link): (count: Number)
**effect** returns the count associated with the link


### Syncs

**sync** init
**when** Request.shortenUrl(User, targetUrl, shortUrlBase); UrlShortening.register(): (shortUrl)
**then** UrlOwnership.addLink(User, shortUrl); UrlAnalytics.init(shortUrl)

**sync** increment
**when** UrlShortening.lookup(shortUrl): (targetUrl)
**then** UrlAnalytics.increment(shortUrl)

**sync** getCount
**when** Request.getCount(User, shortUrl)
**then** UrlAnalytics.getCount(shortUrl): (count: Number)

### Feature Requests

1. I would alter Request.shortenUrl to take an additional argument, shortUrlSuffix, which is the User's desired shortURL. This would have effect in register and init. In register, I would remove NonceGeneration.generate() and replace the shortUrlSuffix argument in UrlShortening.register(shortUrlSuffix: nonce, shortUrlBase, targetUrl) with the given shortUrlSuffix in Request.shortenUrl

2. I would alter the NonceGeneration state so that each Context will have a set of words to be used when calling NonceGeneration.generate(). This set of words can also be global within NonceGeneration, not just Context-specific. 

3. I would alter UrlAnalytics' state to include a target link along with the count, for each Link in the set of Links. The init action would take a new argument, targetUrl, which not only sets the count to 0 for the given short link but also associates the short link with the targetUrl. I would add a new action getTargetCount(targetUrl: String): (count: Number) that returns the sum of counts across all Links that are associated with the given targetUrl. The init sync would have a second argument to UrlAnalytics.init, as mentioned above. I would add a new sync, getTargetCount: when Request.getTargetCount(User, targetUrl), then UrlAnalytics.getTargetCount(targetUrl)

4. This should not be included. The concept of a short URL is that it is a temporary link, meant to be used in the moment, like during a presentation or for a survey. Making the URL less guessable usually means making it more complicated, which hurts usability, as opposed to using common words, which are easier to remember than a URL that is harder to guess. I also believe it is inappropriate to use these shortURLs for sensitive information, and one way of combatting URLs being guessed is to shorten the expiry time.

5. Not desirable, URL creators should have to register as user to be able to see analytics. Without a registration, there is no way to know if someone who wants to see the analytics is the original creator. It may be possible using website cookies, but I believe the system specification should not allow a non-registered link creator to see analytics.



