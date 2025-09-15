# Problem Set 1: Reading and Writing Concepts

## Exercise 1: Reading a Concept

### Question 1: Invariants

For any request, the current remaining count plus the amount of purchases for that request must sum to the count of the request when it was initially added. 
A purchase can only be made for an existing request in that registry, there cannot be a purchase without a request.

I believe the purchase invariant is more important, because it would be unreasonable to purchase an item that the person had not requested, defeating the purpose of the list in the first place.
The "purchase" action is the most affected. The action preserves the invariant because it requires that the registry exists and that the item to be purchased has been requested. It also decrements the count in the request, so there would need to be a linked request to decrement.

### Question 2

Removing the item from the registry after purchases have been made for it could break this invariant. Then, there would be purchases made for a nonexistent item request. The problem could be fixed if the removed item's purchases were cancelled or refunded after the item was removed.

### Question 3

Yes, the "active" status is only a flag and does not actually get rid of the registry or alter its state (besides the flag). Opening and closing the registry only requires the registry to be active/inactive, only changes the flag and nothing else. This could be done in a scenario where a person makes a wishlist designed to be fulfilled over many years, and decides to only activate it at certain times in their life. 

### Question 4

This would matter in the database, it would continually be filled with closed registries that are no longer being used, and that would take up space and computing power. To the users, there is no significant difference, closing the registry makes it invisible to others.

### Question 5

Two likely queries are: the registry owner will want to see who purchased what gift, and a gift giver would like to know what gifts are still available to purchase. 

### Question 6

I would add another flag to the registry called "hidden", and have the create action automatically set hidden to false upon registry creation. I would add a new action called hidePurchaser, which turns the hidden flag active if it is not, and another action called showPurchaser that does the opposite. While the hidden flag is active, the registry will not show the user in any purchases.

### Question 7

Using SKU codes offer specificity. A recipient could want an extremely rare item, or some item in particular, not just a generic one. Using SKU codes will ensure the item is exactly how the recipient wants it, and there is no room for ambiguity.

## Exercise 2: Extending a familiar concept

### Questions

1. A set of Users with a unique username and a password.

2. The register action requires: User with the given username does not already exist. Effects: creates a new User with the given username and password, adds it to the set of Users, return the User. The authenticate action requires: a User with the specified username exists, and the given password matches the User's password. Effects: returns that User.

3. The invariant is that no two different users may have the same username. It is preserved because register requires the given username to not match a username in the set of Users, and authenticate does not affect the state. 

4. To the state, I will add a "confirmed" flag to the set of Users, and a new set of PendingConfirmations, each with a username and a token. I will modify register to set the confirmed flag to false upon first registration, as well as generate a secret token to be emailed to the user. The action will require a new parameter, email address. Register will also create a new PendingConfirmation object with the given username and secret token, and add it to the set of PendingConfirmations. It will then return the new User and the secret token (to be sent in an email to the user via sync). The new "confirm" action takes a username and a token, and requires that there is a PendingConfirmation object with the given username and token. The action will set the confirmed flag to true, delete the PendingConfirmation object, and return the User.

## Exercise 3

**concept** PersonalAccessToken
**purpose** grants access to users, tools, or services without sharing the main user's password
**principle** After a User generates a token for their account, it can then be used instead of the password to authenticate. Scripts, tools, or services can access the User's repositories with the token. After a certain amount of time, tokens can expire, or can be revoked. 
**state** a set of Users, each with a unique username, a password, and a set of Tokens. A set of Tokens, each with a unique value, associated User, "active" flag, and optional expiration date.
**actions** 
generateToken(user: User, expiry: Date): (token: Token)     
requires: an authenticated User, optionally an expiry date
effects: creates a Token associated with the given user with the optionally given expiry date and a unique value, with the "active flag"

revokeToken(token: Token): None
requires: an existing "active" Token associated with a current user
effects: turns off the active tag, makes the token inactive.

authenticate(username: String, tokenValue: String): None
requires: a User with the given username that is associated with an existing, active Token with the given tokenValue.
effects: grants access to the user

It differs from PasswordAuthentication because a User can have many tokens, but only one password. Access is granular as you can generate, give, and revoke tokens individually without affecting access of others. Tokens are designed for not only other users but also programs like APIs, passwords are designed for primarily user access. Tokens can have a set expiration date, but passwords are permanent. 

I could change the Github page by emphasizing that tokens are advantageous/different from passwords because of their increased granularity of access, as well as the fact that they are better suited for use with APIs, scripts, and tools than the user's password. 

## Exercise 4

### URL Shortener

**concept** URLShortener
**purpose** create a shorter link that redirects to the user's intended link
**principle** a user can generate a shorter link to replace a longer one
**state** a set of Links, each associated with a long link and a unique shorter url, either automatically generated or optionally decided by the user
**actions**
shortenLink(link: String, shortLink: String): (shortenedLink: Link)
requires: a shortLink not already associated with an existing Link if it is user-specified. 
effects: generates a new Link, with the associated given link, plus a given or randomly generated shortLink for the user to use. 

Thoughts: the usage should be kept simple, this is a tool often used to give links to large groups of people, so creating or interacting with the link should not be complicated. Ensuring the given link actually works is not a responsibility of the program, but the user, so the program performs no checks for that. Two identical short links should not redirect to different sites, this defeats the short link's advantage, being easy to remember/use. Therefore, user given short links need to be unique. 
 
### Billable Hours Tracking

**concept** HourTracker
**purpose** record when employees begin and end work on a project, so that they can be paid according to their time worked
**principle** an employee signs in to work at their start time describing their plans for the day, then signs out when they leave. 
**state** a set of unique Projects, each associated with a mapping of Employees to their sign in and sign out times, as well as their description for the day. A set of Employees, each with an "clocked in" tag and a set of Projects they have clocked into.
**actions**
createProject(name: String): (project: Project)
requires: there does not currently exist a Project with the given name
effects: creates a new Project with no employees

closeProject(project: Project): None
requires: there exists a Project with the given name
effects: closes an existing Project

clockIn(employee: Employee, project: Project, time: DateTime, desc: String): (project: Project)
requires: there exists a Project and Employee with the given names, the given time is within working hours, the Employee is not currently clocked in
effects: returns a modified Project with a new entry in the mapping for that Employee on the given date and time, with the given description, employee is clocked in

clockOut(employee: Employee, project: Project, time: DateTime): (project: Project)
requires: there exists a Project and Employee with the given names, optional time is within working hours, the employee is currently clocked in. If no time is given, defaults to end of day.
effects: returns a modified Project with a modified entry in the mapping for that Employee on the given time, employee is no longer clocked in

Thoughts: An Employee should not be able to clock in without picking a project / a project being available to work on, so they must pick a valid Project object. An Employee that forgets to clock out will have the clock in flag on until they clock out again. Their clock out time will be set as the end of working hours. They must clock out and set the actual time that they left for that day before they can clock in again. The program itself should not enforce any disciplinary action for falsifying times, it should give users the ability to alter the clock in date when they forget to clock out the day before. The action is for the employers to take separately.

### Conference Room Booking

**concept** ConferenceRoomBooking
**purpose** allow users to reserve conference rooms for meetings
**principle** a user requests to reserve a specific room at a specific time, then only they have access to the room during that time
**state** a set of unique Rooms, each with a set of time slots that may or may not be reserved. A set of unique users that map to rooms they have reserved, and what time.
**actions**
addRoom(room: Room): None
requires: the Room has not already been added to the set of Rooms
effects: adds the room to the set of Rooms, with no time slots and no requests

addSlot(room: Room, starttime: DateTime, endtime: DateTime): (room: Room)
requires: the given Room is in the set of rooms, and there does not already exist a slot at the given time frame.
effect: adds a slot to the Room from the given start time to the given end time. Returns the Room with those states modified.

reserve(room: Room, user: User, starttime: DateTime, endtime: DateTime): (room: Room)
requires: the room is in the set of Rooms, the user is in the set of Users, there exists one or more time slots, in total equal to or greater than the amount of time determined by the start and end time, not reserved for the Room on that date. 
effects: reserves the room for that user from the given start to end date. Returns a modified Room object with those time slots reserved, user added to the room's mapping

cancel(room: Room, user: User): (room: Room)



