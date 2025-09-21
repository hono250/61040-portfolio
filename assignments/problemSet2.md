## Concepts for URL Shortening

1. **Contexts**

The contexts are ensuring that nonces are unique in specific scopes instead of globally.

In the URL shortening app, the context will be the shortUrlBase (ex: "bit.ly" or "tinyurl.com"). This means that bit.ly and tinyurl.com can generate the same nonce but there will be no conflict because they are different contexts. 

2. **Storing used strings**

*NonceGeneration* stores sets of used strings to enable verification before generation of new ones to insure uniqueness of nonces in a context. 

In the case a counter is used and is being incremented each time a new string is generated, the set of used strings contains string representations of all numbers from 1 to the current counter value. 
As an example, each generate() could be incrementing the counter and returning counter.toString()

3. **Words as nonces**

*Advantage*: the URLs would be more memorable and easier to share verbally.

*Disadvantage*: There might be a limited word pool which would mean that for popular contexts, you would run out of words faster 

*Modification of NonceGeneration*

    concept NonceGeneration [Context]
    
    purpose generate unique strings within a context
    
    principle each generate returns a string not returned before for that context
    
    state
     a set of Contexts with
      a used set of Strings
      a wordPool set of Strings
      
    actions
    
    addWords (context: Context, words: set String)
     effect adds words to the wordPool for this context
    
    generate (context: Context) : (nonce: String)
     requires wordPool for this context has some unused words
     effect returns a nonce that is not already used by this context

## Synchronizations for URL Shortening

1. **Partiela matching**

In the first sync *generate* shortUrlBase is passed in but not targetUrl because this sync is triggered immediately when the 
user makes a shortening request which means that at this point it only needs the domain/context to generate the nonce and does not need to wait for the full request details. 

The second sync *register* needs to wait for both the original request completion and the nonce generation completion before proceeding.
It needs both targetUrl (from the request) and the generated nonce to create an actual shortening that is linked to the targetUrl. 

2. **Omitting names**

The convention only works when ther argument matches the variable name and can't be used if names differ or if you need to bind values differently.

3. **Inclusion of request**

the *request* action is included in the *generate* and *register* syncs but not in the *setExpiry* one because the first two syncs are user-triggered and they 
happen in response to user requests but the thirs sync is system-triggered which meand that it happens automatically when a shortening has been registered regardeless of user action. 

4. **Fixed domain**
   
For a fixed somain like "bit.ly" the sync would become:

    sync generate
    when Request.shortenUrl ()
    then NonceGeneration.generate (context: "bit.ly")

    sync register  
    when
     Request.shortenUrl (targetUrl)
     NonceGeneration.generate (): (nonce)
    then UrlShortening.register (shortUrlSuffix: nonce, shortUrlBase: "bit.ly", targetUrl)

    sync setExpiry
    when UrlShortening.register (): (shortUrl)
    then ExpiringResource.setExpiry (resource: shortUrl, seconds: 3600)

5. **Adding an expiration sync**

       sync expire
       when ExpiringResource.expireResource (): (resource)
       then UrlShortening.delete (shortUrl: resource)

## Extending the design

1. **Additional concepts**
   
       concept Analytics [User, Resource]
   
       purpose track usage statistics for resources owned by users
   
       principle after a user registers a resource for tracking,
        each access increments the count and can be queried by the owner
   
       state
        a set of TrackedResources with
         an owner User
         a resource Resource
         a accessCount Number

       actions
        startTracking (owner: User, resource: Resource)
         requires no tracking exists for this resource
         effects create new TrackedResource with accessCount = 0

        recordAccess (resource: Resource)
         requires tracking exists for this resource
         effects increment accessCount for this resource

        getStats (user: User, resource: Resource): (count: Number)
         requires user is owner of tracking for this resource
         effects return accessCount for this resource


        concept UserAccount
   
        purpose manage user identity and ownership
   
        principle users register with credentials and can then be authenticated
   
        state
         a set of Users with
          a username String
          a password String

         actions
   
          register (username: String, password: String): (user: User)
           requires username not already taken
           effects create new user with hashed password

         authenticate (username: String, password: String): (user: User)
          requires user exists with matching credentials
          effects return authenticated user

2. **Synchronizations**

       sync trackNewShortening
       when
        UrlShortening.register (): (shortUrl)
        UserAccount.authenticate (): (user)
       then Analytics.startTracking (owner: user, resource: shortUrl)

       sync recordUrlAccess
       when UrlShortening.lookup (shortUrl)
       then Analytics.recordAccess (resource: shortUrl)

       sync viewAnalytics
       when
        Request.getAnalytics (shortUrl)
        UserAccount.authenticate (): (user)
       then Analytics.getStats (user, resource: shortUrl): (count)


3. **Feature assessment**

- *Allowing users to choose their own short URLs*: Modify NonceGeneration to have an optional *customNonce* parameter.
  
- *Using the “word as nonce” strategy to generate more memorable short URLs*: as demonstrated, use word pools and modify *NonceGeneration* to us those generate words from word pools (added through the *addWords* action.
  
- *Including the target URL in analytics, so that lookups of different short URLs can be grouped together when they refer to the same target URL*:  track stats by *targetURL* instead
  of *shortURL* (modify the syncs and pass in *targetURL* as the Resource in the syncs to create trackings for them instead of tracking *shortURL*)
  
- *Generate short URLs that are not easily guessed*: modify NonceGeneration to not use simple sequential patterns but maybe use cryptographically secure random generations.
  
- *Supporting reporting of analytics to creators of short URLs who have not registered as user*: undesirable because it may create privacy issues (maybe may result to using IP based identification which may also cause privacy concerns to users)

