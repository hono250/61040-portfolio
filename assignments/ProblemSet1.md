# Exercise 1 
1. **Invariants**
   
- *Invariant 1*: For a specific item in a registry, its count remains a non-negative integer at all times. This important because it prevents purchasing more of an item than requested.
- *Invariant 2*: Every purchase must correspond to a specific request.
  
Invariant 1 is more important because it prevents purchasing more of an item than requested uphold the purpose of this concept. The action most affected by this invariant is *purchase* and it prevents breaking the invariant checking first upon getting a request that for that specific registry, there is a request for this item with at least the purchase count and the decrementing the count for that request after upon the purchase.   

2. **Fixing an action**
   
The *purchase* action can potentially break Invariant 1 in concurrent purchases. If more than one people try to purchase an item at a time, they might pass the "at least count" precondition before either of them decrements the count which might lead to over purchasing of an item. This could be fixed by implementing waits so that once check and decrement actions are atomic. 

4. **Inferring behavior**

A registry can be opened and closed repeatedly. For instance, a couple may close their registry after a wedding but might want to reopen it on their anniversary instead of creating a new one. 

6. **Registry deletion**
   
In practice, the fact that a registry cannot be closed can be problematic in terms of storage (for creators) and also privacy concerns (for users).

8. **Queries**
   
- What gifts have already been purchased and by whom?
- What gifts are still available to purchase?

6. **Hiding purchases**
   
For the state, add a HidePurchases flag and then add a setHidePurchases actions that if set true, purchases will be hidden to the recipient. 

8. **Generic types**
   
The Item types as it is preferable because the concept should focus more on registry logic and not item details. 

# Exercise 2

**concept** PasswordAuthentication

**purpose** limit access to known users

**principle** 

    after a user registers with a username and a password, they can authenticate with that same username and password and be treated each time as the same user
    
1. **concept state**
   
**state**

    a set of Users with
        a username String
        a password String 
        
3. **Specifications for actions**
   
**actions**

    register (username: String, password: String): (user: User)
        requires no other existing user has this username
        effects creates a new user with this username and this password, add to the set Users and return user 
        
    authenticate (username: String, password: String): (user: User)
        requires a user with this username and this password exists
        effects return the matching user (no state change here)

5. **Invariant**
   
Each user has a unique username. This invariant tis preserved when creating a new user, it is required that no other user has that username

7. **Email confirmation extension**
   
9.
**state**
  
    a set of Users with
        a username String
        a password String
        a confirmed Boolean
  
    a set of PendingRegistrations with
        a username String
        a password String
        a token String
   
**actions**

    register (username: String, password: String): (user: User, token: String)
        requiresusername is not already taken by an existing user or pending registration
        effects create a new pending registration with this username, password and a random token and return a placeholder user (confirmed Boolean is set to False for now) and the token
        
    confirm (username: String, token: String): (user: User)
        requires a pending registration exists with this username and token
        effects set the confirmed user Boolean to True for user with the username and password from pending registration, remove the pending registration, return the user
        
    authenticate (username: String, password: String): (user: User)
        requires a confirmed user exists with this username and password
        effects return the matching user

# Exercise 3

##### PersonalAccessToken Concept Specification

**concept** PersonalAccessToken [User, Scope]

**purpose** provide time-limited, scope-restricted authentication for programmatic access

**principle** 

      a user creates a token with specific scopes and expiration; 
      the token can then be used for authentication instead of a password until it expires or is revoked; 
      the token only grants access to resources within the specified scopes
      
**state**

    a set of Tokens with
        an owner User
        a tokenString String
        a set of scopes Scope
        an expiration Date
        an active Boolean
        
**actions**

    createToken (user: User, scopes: set Scope, expiration: Date, name: String): (token: Token)
        effects create a new active token with generated tokenString for this user with specified scopes and expiration
        
    authenticate (tokenString: String): (user: User, scopes: set Scope)
        requires a token exists with this tokenString, is active, and has not expired
        effects return the token owner and granted scopes
        
    revokeToken (token: Token)
        requires token exists
        effects set token to inactive
        
    autoExpire ()
        effects set all tokens past their expiration date to inactive

**Differences between PersonalAccessToken and PasswordAuthentication**: PersonalAccessToken are limited to so that each token is limited to specific scopes; they are also time limited and revocable. 

**GitHub page improvements**: The GitHub page could be improved by leading with clarity. The page should start by clearly explaining that Personal Access Tokens are fundamentally different from passwords in purpose and scope, that passwords are for human interactive login with full account access while tokens are for automated access with limited and specific permissions. 

# Exercise 4

##### 1. URL Shortener Concept

**concept** URLShortener

**purpose** create short, memorable aliases for long URLs

**principle** 

    a user provides a long URL and optionally a custom suffix;
    the system creates a short URL that redirects to the original;
    anyone can use the short URL to access the original destination
  
**state**

    a set of Mappings with
        a shortCode String
        a originalURL String
        a creator User (optional)
        a createdAt Date
        a clickCount Number
        
**actions**

    shortenURL (originalURL: String, customSuffix: String (optional), creator: User (optional)): (shortURL: String)
        requires originalURL is valid, if customSuffix provided, it is not already taken
        effects if customSuffix provided, create mapping with that suffix otherwise generate unique random suffix; create mapping with shortCode, originalURL, creator, current date, clickCount = 0; return shortURL with shortCode
        
    redirect (shortCode: String): (originalURL: String)
        requires mapping exists for this shortCode
        effects increment clickCount for this mapping; return originalURL
        
    getStats (shortCode: String): (clickCount: Number, createdAt: Date)
        requires mapping exists for this shortCode
        effects return click statistics for this mapping
        
    deleteMapping (shortCode: String, user: User)
        requires mapping exists and user is creator (if creator was specified)
        effects remove mapping from system
        
**Notes**:
- This concept checks for uniqueness of short URls before creating
- Only verified creators of mappings are able to delete them
- It has a builtin tracking tool for the number of clicks
- supports defined suffixws and auto generated ones

##### 2. Conference Room Booking Concept

**concept** ConferenceRoomBooking [User, Room]

**purpose** reserve meeting rooms for specific time periods

**principle**

    a user books a room for a specific time slot;
    the room becomes unavailable for other users during that period;
    the user can modify or cancel their booking;
    rooms automatically become available again after bookings end
    
**state**

    a set of Bookings with
        a room Room
        a booker User
        a startTime DateTime
        an endTime DateTime
        a purpose String
        a status BookingStatus (confirmed, cancelled)
        
**actions**

    bookRoom (user: User, room: Room, startTime: DateTime, endTime: DateTime, purpose: String): (booking: Booking)
        requires startTime < endTime, no confirmed booking exists for this room that overlaps with [startTime, endTime)
        effects create new confirmed booking with these parameters
        
    cancelBooking (booking: Booking, user: User)
        requires booking exists and user is the booker, booking startTime is in the future
        effects set booking status to cancelled
        
    modifyBooking (booking: Booking, user: User, newStartTime: DateTime, newEndTime: DateTime)
        requires booking exists and user is the booker, booking startTime is in the future, newStartTime < newEndTime, no other confirmed booking overlaps with new time range for this room
        effects update booking with new times
        
    checkAvailability (room: Room, startTime: DateTime, endTime: DateTime): (available: Boolean)
        effects return true if no confirmed bookings overlap with the time range
        
    getCurrentBookings (room: Room): (bookings: set Booking)
        effects return all confirmed bookings
        
**Notes**:
- This concept has an invariant that prevents overlaps in bookings.
- cancelled bookings don't block availability of a room 
- The concept only allows future manipulations 

##### 3.  Time-Based One-Time Password (TOTP) Concept

**concept** TimeBasedOTP [User]

**purpose** provide temporary codes for two-factor authentication

**principle** 

    user scans a setup QR code with their authenticator app;
    the app displays 6-digit codes that change every 30 seconds;
    user enters the current code along with password to authenticate
    
**state**

    a set of UserSecrets with
        a user User
        a secret String
        a isEnabled Boolean
        
**actions**

    setupTOTP (user: User): (qrCode: String)
        effects generate new shared secret, create inactive TOTPSecret
        
    enableTOTP (user: User, code: String): (success: Boolean)
        effects if code matches what the secret should generate now, enable TOTP
        
    verifyCode (user: User, code: String): (success: Boolean)
        requires TOTP is enabled for user
        effects return true if code matches current expected value
      
**Notes**:
- compromised passwords from other sites won't grant access
- sucessfully protects even with password breaches but also malware on your device can read TOTP codes


