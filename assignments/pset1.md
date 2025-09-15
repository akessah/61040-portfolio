# Akosua Essah PSet 1: Reading and Writing Concepts
## Exercise 1
1. Invariants:
    - The total number of purchases of an item must be at most the total number of that item requested
    - Any item in a purchase must match the item of the request it's in

    The first of these is more important because if it isn't held, that means more of the item was bought than was desired. Having unecessary and undesired gifts defeats the entire purpose of having a registry.

    The Purchase action is the one most affected by this. It preserves this invariant by requiring the request count is at least the purchase count.

2. Remove item may break this invariant. If the owner removes an item that has purchases, then there would be purchased items floating with no request. One way to fix this is to have removeItem set request count to 0 instead of removing the item entirely.

3. The registry can be opened and closed repeatedly under the spec. One reason to allow this is the owner may want to stop receiving items for some amount of time (maybe because they got an influx of items from some other source and need to reevaluate what they now need), but not permenantly.

4. Having no delete action may get annoying for owners if they somehow make a mistake when making the registry and have to keep it. Functionally, however, the close action can serve all purposes a delete action would.

5. See purchases and purchasers for the owner and see registry and items for the gift giver

6. You could include a hidden Flag in Registry so that the owner can only see purchasers if the Flag is false. Also have a makeHidden(r: Registry) and makeVisible(r: Registry) so that the owner can toggle this Flag

7. Names, descriptions, and prices aren't actually relevent to the actions in the concept, so we don't want to limit ourselves to having those be the identifiers of items if we don't have to

## Exercise 2
1. a set of Users with
    a username String
    a password String

2.
    register(username: String, password: String): (user: User)

        requires: User with same username not already in User set

        effects: adds a User with username and password to the User set

    authenticate (username: String, password: String): (user: User)

        requires: User with username exists in set, password matches the password of User in set

3. Users must have unique usernames. This is preserved by requiring username to not be in the set when completing the register action

4. First modify the state so that each user in the set has a confirmed Flag and a token Token. Modify register so that it returns that Token and modify authenticate so that it requires the confirmed Flag to be true.

    confirm(username: String, token: Token)

        requires: token matches the token associated with the User with username in the set, confirmed flag is false

        effects: sets confirmed flag to true

## Exercise 3
concept: PersonalAccessTokens [User, Token, Scope]

purpose: limit access to users in a secure way that users can easily control.

principle:
    a user registers with a username. The user can then generate PATs that they can specify the resources it has access to and set an expiration date on. The user uses a nonexpired PAT to sign into the service and has access to certain resources depending on the access level of the PAT.

state:
    a set of Users with
        a username String
        a set of Tokens

    a set of Tokens with
      a name String
      a scope Scope
      an expiration Date

actions:
    register (username: String): (user: User)
      requires: User with same username not already in User set
      effects: adds user to the User set

    addToken (expiration: Date, scope: Scope) (token: Token)
      effects: creates a token with these parameters and adds it to the user's set of Tokens

    deleteToken (token: Token)
      requires: token exists in the user's set of Tokens
      effects: remove the token from the set

    modifyToken (token: Token, expiration: Date, scope: Scope)
      requires: token exists in the user's set of Tokens
      effects: changes token's expiration and scope to the ones specified

    authenticate (token: Token)
      requires: token exists in user's set of Tokens

## Exercise 4
concept: URLShortener [ShortURL]

purpose: create short, easy-to-communicate urls from longer ones that will take users to the same page.

principle:

    a user enters a url and a path name the generate themselves. The service then creates a url with their domain and the path given that redirects to the page the original url does. The user can then share this url with whoever they want.

state:

    a set of shortURLs with
        a path String
        an url String

actions:

    createURL (url: String, path: String): (short: shortURL)
        requires: shortURL with same path not already in set
        effects: adds shortURL with the url and path to the set


concept: roomBooking [Slot, Location]

purpose: allow people to more accurately plan where they will hold meetings and activities.

principle:

    a user reserves a specific room at a specific day and time. The room then gets used by the user at that day and time.

state:

    a set of Slots with
        a time Time
        a room Location

    a set of Reservations with
      a user User
      a slot Slot

actions:

    reserve (user: User, time: Time, room: Location): (reservation: Reservation)
        requires: a Slot with the given Time and Location exists in the set and isn't reserved
        effects: creates and returns a Reservation with a Slot with the given Time and Location

    cancel (reservation: Reservation)
        requires: reservation exists in the set of Reservations
        effects: reservation is removed from the set of Reservations

    createSlot (time: Time, room: Location): (slot: Slot)
      requires: a Slot with the same Time and Location doesn't exist in the set of Slots
      effects: a Slot with the given Time and Location is created and returned

    deleteSlot (slot: Slot)
      requires: slot exists in the set of Slots
      effects: removes slot from set of Slots

concept: EBoardingPass [Pass, Flight, Passenger]

purpose: allow users to check-in to and board planes with mobile device and avoid needing to carry around seperate pass.

principle:

    a user buys a plane ticket and in sent an e-boarding pass. They then use the pass to gain entry into airport and onto the plane they bought a ticket for.

state:

    a set of Flights with
        a gate Number
        a departure Time
        a set of Passengers

    a set of Passes with
        a passenger Passenger
        a flight Flight
        an active Flag


actions:

    issuePass (flight: Flight): pass Pass
        requires: the current time to be one day before departure of flight
        effects: returns a pass to all the passengers in the flight's passenger set

    changeGate (flight: Flight, gate: Number)
        requires: flight to exist in Set
        effects:changes gate of all Passes with flight to the gate Number specified

    deactivate (pass: Pass)
        requires: pass exists in pass set and active flag is true
        effects: marks active flag as false


Note: the deactivate action is meant to be a general action that can used for multiple purposes such as if a passenger wants to cancel their flight or they get kicked off a flight for some reason.
