# Akosua Essah PSet 1: Composing Concepts
## Concepts for URL Shortening
1. Context is used to seperate the different services that would use this concept so that the generated nonces are unique to the service, not necessarily unique to the entire concept. In a URL shortening app, a context will end up being a domian.
2. The used string sets are used to determine which nonces shouldn't be returned on any given call to generate. If using the counter implementation, the counter will represent the number of strings in the used string set. If using a function that maps a number to a string to generate strings, calling generate will use the counter number as input.
3. One advantage is that the shortened URLs will be even easier to communicate and remember for users, further fulfilling the purpose of the concept. One disadvantage is that there are a limited number of dictionary words, so depending on the total volume of uses, users may need to have sooner expirations forced on them to avoid running out of possible URLs. For NonceGeneration to reflect this idea, have it store a used word set instead of a generic used string set.

## Synchronizations for URL Shortening
1. The then clause for register uses both targetUrl and shortUrlBase arguments. The then clause for generate, however, only uses shortUrlBase, so including the targetUrl argument wouldn't give any useful or relevent information for that specific sync.
2. It makes it more clear what the variable is in the context of the action being called. This is especially important considering not every argument is included when writing synchronizations, so going by the argument's position won't always be accurate.
3. The UrlShortening.register() action in the when clause has all of the arguments needed to make a call to ExpiringResource.setExpiry(), so including a request action, which is already implied to have been called in order to call the register action, is redundant.
4. For the both the generate and register syncs, remove the shortUrlBase argument from the request. For the generate sync, pass the fixed domain into the generate action instead of shortUrlBase. Similarly, for the register sync, pass the fixed domain into the register action instead of shortUrlBase.
5. **sync** expire<br>**when** ExpiringResource.expireResource(): (resource)<br>**then** UrlShortening.delete(shortUrl: resource)

## Extending the Design
1. Add a new concepts:
    * ViewAnalytics[Resource] which keeps records of views for a resource:<br>
    	**state**:<br>
        	a set of Resources with <br>
                a views Number <br><br>
        **actions**:<br>
            addResource(resource: Resource)<br>
                **requires**: resource not in set<br>
                **effects**: adds resource to set, sets views to 0<br>
            incrementViews(resource: Resource)<br>
                **requires**: resource in set<br>
                **effects**: add 1 to that resource's views in the set<br>
            removeResource(resource: Resource)<br>
                **requires**: resource in set<br>
                **effects**: removes resource from set<br><br>
    * PasswordAuthentication:<br>
        **state**:<br>
            a set of Users with<br>
                a username String<br>
                a password String<br><br>
        **action**:<br>
            register(username: String, password: String): (user: User)<br>
                **requires**: User with same username not already in User set<br>
                **effects**: adds a User with username and password to the User set<br>
            authenticate (username: String, password: String): (user: User)<br>
                **requires**: User with username exists in set, password matches the password of User in set<br>
            changePassword(username: String, oldPassword: String, newPassword: String)<br>
                **requires**: User with username exists in set, oldPassword matches the password of User in set<br>
                **effects**: modifies the password associated with User to be newPassword<br>


2. New synchcronizations:
    **sync** createAnalytics<br>**when** Request.authenticatePassword(username, password), UrlShortening.register():(shortUrl)<br>**then** ViewAnalytics.addResource(resource: shortUrl), PasswordAuthentication.register(username, password)

    **sync** addView<br>**when** UrlShortening.lookup(shortUrl)<br>then ViewAnalytics.incrementViews(shortUrl)

    **sync** viewAnalytics<br>**when** Request.viewAnalytics(username, password, shortUrl)<br>**where** ViewAnalytics: views count for shortUrl is v<br>**then** PasswordAuthentication.authenticate(username, password), Request.respone(v)
3.
* Add another argument to Password.shortenUrl, shortUrlSuffix, which can take a string or None. Modify the generate and register syncs to have shortUrlSuffix: None in the Request.shortenUrl. Add an additional sync, registerUserGenerated:
    when Request.shortenUrl (targetUrl, shortUrlBase, shortUrlSuffix)<br>then UrlShortening.register (shortUrlSuffix, shortUrlBase, targetUrl)
* To use the word as nonce strategy, use the modified NonceGeneration concept from problem 1.3 instead of the original one.
* In the ViewAnalytics, have a target field associated with each resource along with a views variable. Users can now query for the sum of views fields of their shortUrl's the have the same target.
* The stated purpose of the UrlShortening concept is to get shorter/more memorable links, so to generate harder to guess short URLs goes against the main purpose and shouldn't be included.
* This may end up being a security risk if users for some reason want to keep the number of lookups of their shortUrl private, so this should not be included.
