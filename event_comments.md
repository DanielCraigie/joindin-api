---
layout: page
title: Event Comments
---

Users can comment on events.  Currently the user must be signed in before they can comment (anonymous options may appear in the future).

## Event Comments Format

Event comments are available from the location in the ``comments_uri`` of an event.  The format looks like this:

~~~~
comments:
    0:
        rating: 4
        comment: Proin feugiat mattis dui, ut cursus purus feugiat vel. Etiam ligula elit, condimentum lacinia fermentum nec, elementum id urna. Etiam ligula elit, condimentum lacinia fermentum nec, elementum id urna. Proin feugiat mattis dui, ut cursus purus feugiat vel. Vivamus gravida, dolor ut porta bibendum, mauris ligula condimentum est, id facilisis ante massa a justo. Etiam ligula elit, condimentum lacinia fermentum nec, elementum id urna. Vivamus gravida, dolor ut porta bibendum, mauris ligula condimentum est, id facilisis ante massa a justo. Vivamus gravida, dolor ut porta bibendum, mauris ligula condimentum est, id facilisis ante massa a justo.
        created_date: 2013-10-03T16:30:02+02:00
        user_display_name: Maria Hansen
        user_uri: http://api.joindin.local/v2.1/users/10
        comment_uri: http://api.joindin.local/v2.1/event_comments/22
        verbose_comment_uri: http://api.joindin.local/v2.1/event_comments/22?verbose=yes
        event_uri: http://api.joindin.local/v2.1/events/31
        event_comments_uri: http://api.joindin.local/v2.1/events/31/comments
~~~~

## Event Comments Verbose Format

Verbose adds the ``source`` and ``gravatar_hash`` fields to the representation.

## Event Comments Data Fields

The fields in an event comment are as follows:

*  ``rating``: The rating from the user (not all our platforms do or historically did support this, so this field is often empty)
*  ``comment``: The comment made by the user
*  ``source``: Which tool the user made to create this comment (can be empty if we don't have the info)
*  ``created_date``:  The date that this comment was made, in ISO format
*  ``user_display_name``: This is a convenience field (and it's optional, the original website supports anonymous event comments) so you can show a user name with the comment, it relies on the user_uri
*  ``gravatar_hash``: Unique identifier for showing their gravatar image; append this to ``http://www.gravatar.com/avatar/`` to make the image URL

## Event Comments Hypermedia

*  ``user_uri``:  The identifier of the user that made the comment (optional, the old site didn't require sign-in)
*  ``comment_uri``:  The identifier of this record
*  ``verbose_comment_uri``: The verbose representation of this record
*  ``event_uri``: The event this comment relates to
*  ``event_comments_uri``:  Where to find all the comments for this event


## Adding Comments To Events

You can add comments to events via the API; you must be authenticated to do so.

To create a new comment, POST the ``rating`` and ``comment`` comments fields in an array to the event comments collection that you want to add it to.  The API will pick up your identity and add the timestamp.  E.g (using curl against my test system):

<pre class="embedcurl">curl -v -H "Content-Type: application/json" -H "Authorization: Bearer f9b4f1a9b30bdc0d" -X POST http://api.dev.joind.in/v2.1/events/31/comments --data '{"rating": 5, "comment": "Wonderful event, thanks!"}'
</pre>

<!-- You only need to reference this script once per page. -->
<script src="https://www.embedcurl.com/embedcurl.min.js" async></script>

The ``-v`` switch is there so that you see the whole response, which looks something like this:

~~~~
> POST /v2.1/events/31/comments HTTP/1.1
> User-Agent: curl/7.32.0
> Host: api.joindin.local
> Accept: */*
> Content-Type: application/json
> Authorization: Bearer f9b4f1a9b30bdc0d
> Content-Length: 34
> 
* upload completely sent off: 34 out of 34 bytes
< HTTP/1.1 201 Created
< Date: Sun, 19 Jan 2014 21:51:37 GMT
* Server Apache/2.4.6 (Ubuntu) is not blacklisted
< Server: Apache/2.4.6 (Ubuntu)
< X-Powered-By: PHP/5.5.3-1ubuntu2.1
< Location: http://api.joindin.local/v2.1/event_comments/204
< Content-Length: 0
< Content-Type: text/html
~~~~

The ``Location`` header will point to the newly-created comment, and the status code of 201 indicates that all went well.  If anything does go wrong, you will get a 4xx status code response with a message indicating what the problem is.

Duplicate comments produce a 400 with "Duplicate comment" in the body.  Spam comments return a 400 with the message "comment failed spam check".

## Reporting a comment

If a comment contains inappropriate content, a user can report it.  This can be done by sending a POST request to the ``reported_uri`` URL available in the comment resource.

Example request/response:

~~~~
> POST /v2.1/event_comments/204/reported HTTP/1.1
> User-Agent: curl/7.38.0
> Host: api.dev.joind.in
> Accept: */*
> Authorization: Bearer 11260f116ecc0fc7
> 
< HTTP/1.1 202 Accepted
< Date: Thu, 19 Nov 2015 17:27:42 GMT
< Server: Apache/2.2.22 (Debian)
< X-Powered-By: PHP/5.6.10-1~dotdeb+7.3
< Location: http://api.dev.joind.in/v2.1/events/65/comments
< Vary: Accept-Encoding
< Content-Length: 0
< Content-Type: text/html; charset=UTF-8
< 
~~~~

This adds the comment to the list of reported comments and stops returning it in the collection (but the deletion can be undone by an admin when they moderate the comments).

If successful, a 202 Accepted status will be returned along with a ``Location`` header pointing back to the comments collection that this reported comment was in.

