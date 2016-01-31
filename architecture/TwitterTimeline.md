# What's this
In this post, I will describe how a twitter-like system implements the timeline service. I will concentrate on the core concept and omit all other things. Actually, this is just a example of realtime feeding systems. I will only talk about the scalable architecture and let out other QoS issues.

# Behavior of twitter timeline
To begin with, I'll describe the behavior. Think about the following elements, be sure that your know what there are:     

+ User social graph: A follows B, C. B follows C, D, E, F, C follows A.
+ Timeline in the home page
+ Timeline in the mobile app.
+ Timeline events: twitters by people who you are following ordered by time.

So what happens if C posts a new twitter? 
   
+ C posts a twitter
+ The server received the twitter, and notify other components.
+ A in browser periodically polls the server and requests for new timeline events such as new twitters posted by people who he followed.
+ B in mobile app received a message from the push server telling him that there was a new twitter.

Basically, there are two ways to get new timeline events, polling and pushing. Browsers use polling for historical reasons and apps use pushing since sockets can be used in apps.

# What happens in the background?
Now let's follow a new twitter and see what happens. 

A twitter firstly goes to the web app. The app persists the twitter(actually we don't care who does this) and notify other components that we have a new twitter `<id: 123, user: C, ...>`. Now we only care about two components: Fanout and PushServer.

## Fanout
Fanout prepare timeline events for polling.

When Fanout received a NewTwitterEvent(`<id: 123, user: C, ...>`), it queries SocialGraph to retrieve all users who are following C. Each active user(logined in last few months) has their own timeline cache(in memory). The NewTwiiterEvent will be dispathed to timeline caches of user A and user B. And we're done. There are some other topics related to Fanout such as redundancy, they deserve other posts since this post is only about how the timeline service is implemented.

## PushServer
***Note***: I'm not sure how this part is implemented, I just write down a typicall implementation.

The PushServer has many rooms, each room is just a server maintaining connections to clients. There are two questions followed: 
 
+ Why we have many rooms? Because there are too many clients, a single server can only serve limited number of clients. Clients will be assigned to a specified room based on clients' location, network status and other factors.
+ How PushServer forwards the event? There are two choices:    
    + Forward to every room and let the room itself to deal with the rest
    + Forward only to rooms of followers of C(the creator of the new twitter)

We use the first approach. Maybe the second one is also acceptable, who knows. 

The PushServer queries the SocialGraph, getting all followers. For each follower, it finds out the room he belongs to and sends the room a message. The room will push the new twitter to the client(via a socket connection).

# What happens in the frontend?
So we almost know what happens in the background, now let's tell the rest of the story.

When a user enters his home page. he will requests the server for its timeline page. When its timeline cache exists, simple returns, otherwise we have to reconstruct his timeline which invovles complicated database queries.

After that?

## Browser users
A browser user will periodicall poll the server and see if there are new events in their timeline cache.

## Mobile users
A mobile user will request the server for a room endpoint, and connect to it. Once the connection is established, it can received new events pushed by the PushServer.

# Addition
+ Actually, you can use pushing consistently in both browsers and apps, but it requires additional work.
+ About the room server there are some issues you may not know. Refer to my other post: Subtleties in a chat room server. 
+ The content above is just part of the story. Other topics such as error handling, redundancy, stability are also important, but I won't address them in this post.