---
title: Ajax
date: 2011-01-29
tags: [self]
---

I always thought Ajax (and JavaScript for that matter) was some crazy 
web technology, some new way of programming the web, some big scary 
thing that would be really difficult to learn or use.

It's not. It's actually just a defined, convenient way of using existing 
tools to accomplish some goal. I won't say it's perfect, or the 
be-all-end-all of web-tech, but it did come in handy for me in one case.

I'd like to share the experience, shed some light on this methodology 
and perhaps make it easy for someone else to add it to their bag of 
tricks.

## What is Ajax?

Ajax stands for *Asynchronous JavaScript and XML*. It's a means of 
making static web pages dynamic.

The way it works is this:

You've got some page that loads up in a user's browser presenting some 
information. Once the content is served to the user that's it. It's 
there, static and stale.

You want to periodically update some information on that page, but how?

JavaScript could update tag contents, but the source code for your 
awesome JavaScript is served out along with the content and it's just as 
stale. It has no idea what new information it needs to present on your 
behalf.

Well, what if we could *phone home*, tell the JavaScript to call back 
to the server and ask for updated information, then update the page 
accordingly.

That is Ajax.

You basically develop two webpages: the first, is the normal static html 
page that the user requests. The second is a dynamic page which serves 
an XML document (or JSON) which provides updated info through 
server-side logic.

The first page comes complete with JavaScript capable of requesting the 
second page on an interval and updating the first page with the new 
information it finds in the XML.

Pretty slick.

## An Example

I recently wrote a "subsite" for the Haskell web framework Yesod. That's 
a lot of jargon, but just think of it as a plug-in.

It lets you control an instance of [mpd][] running on the same server 
via a web page.

More information (and the full source) can be found [here][github-mpc] 
but I'll try and distill out the generic Ajax involved.

Let's say I have a "Now Playing" page which shows the currently playing 
Artist, Album, and Title. I want to update that information as the 
track changes without the user having to constantly refresh the page.

Here's how you do that:

The main page is served as normal html with some handy id's so we can 
find specific tags when we want to update them.

This page will also include all the JavaScript to make the request and 
do the updates, but I'm going to save that information until after the 
XML part.

```html 
<!-- http://server/pages/nowplaying.html -->
<!DOCTYPE html>
<html>
    <head>
    <script>
        // There be lots of JavaScript here, but I'll get to that 
        // later...
    </script>
    </head>
    <body>
        <p id="artist">Medeski Martin &amp; Wood</p>
        <p id="album">Tonic</p>
        <p id="title">Thaw</p>

    <!-- our main ajax entry point will be this function, call it on 
         document load to kick things off -->
    <script>window.onload = timedRefresh;</script>
    </body>
</html>
```

Behind the scenes, you'll need that second page which is just XML. It'll 
have to be driven by server-side code to serve updated information each 
request.

```xml 
<!-- http://server/nowplaying.xml -->
<?xml version="1.0" encoding="utf-8"?>
<xml>
    <status>OK</status>
    <artist>Dave Weckl Band</artist>
    <album>Multiplicity</album>
    <title>Mixed bag</title>
</xml>
```

The status tag is an extra fail-safe in-case your server runs into 
trouble coming up with updated info. You'll see we write our JavaScript 
to be conditional on this tag's value.

With the XML page available at any moment to provide updated 
information, here's the JavaScript which you would need to put in the 
user-facing page to accomplish the constant screen updates.

```javascript 
// this object will make the request for the updated xml and provide an 
// easy means for parsing it
var xmlhttp = new XMLHttpRequest();

// when it receives a response from your server this code will run
xmlhttp.onreadystatechange = function()
{
    // if the state is 4 and the status is 200, that means we're ready 
    // to parse the response
    if (xmlhttp.readyState == 4 && xmlhttp.status == 200)
    {
        // parse the response into a workable document
        xmlDoc = xmlhttp.responseXML;

        // use a helper function to get the contents of specific tags 
        // and deal with them appropriately
        xStatus = xmlHelper(xmlDoc, "status");

        // make sure our server thinks all's ok
        if (xStatus == "OK")
        {
            // get the new info from the xml response
            xArtist = xmlHelper(xmlDoc, "artist");
            xAlbum  = xmlHelper(xmlDoc, "album" );
            xTitle  = xmlHelper(xmlDoc, "title" );

            // another helper function updates a specific tag on the 
            // current page by its id value
            docHelper(true, "artist", xArtist);
            docHelper(true, "album" , xAlbum );
            docHelper(true, "title" , xTitle );
        }
    }
}

// the helper to get a specific tag from the xml:
function xmlHelper(_xmlDoc, _tag) {
    return _xmlDoc.getElementsByTagName(_tag)[0].childNodes[0].nodeValue;
}

// the helper to set (or get) the value of a tag in this document by
// its id:
function docHelper(_set, _id, _value) {
    if (_set) {
        document.getElementById(_id).innerHTML = _value;
    }
    return document.getElementById(_id).innerHTML;
}

// this function actually makes the request, phoning home for updated 
// now playing information
function getNowPlaying() {
    xmlhttp.open("GET", "http://server/nowplaying.xml", true);
    xmlhttp.send();

    // loop again
    timedRefresh();
}

// on document load, we call this function which will start the 
// never-ending loop of updates
function timeRefresh() {
    var delay = 1000; // seconds * 1000
    setTimeout("getNowPlaying();", delay);
}
```

And that's it my friends. Constantly updating screen content without 
compulsive refreshing cluttering up your server logs.

## Update: JSON and jQuery

So, the above all works fine and dandy. Nowadays though, seems all the 
cool kids are doing this with JSON and jQuery.

JSON is a structured data response that can better play the role 
initially delegated to XML. It's nicer because it can be worked with in 
JavaScript without parsing.

Using jQuery just removes boilerplate code and makes a lot happen in 
just a few lines.

Combine this with the awesome JSON support built into Yesod, and I was 
able to get my [MPD controller][github-mpc] updating via JSON with much 
cleaner code than the XML version.

For those that are interested, here is how you set up a page in Yesod to 
reply with *either* the normal HTML or a JSON response depending on what 
the client asks for.

This means I don't need a separate status.xml to deliver the updated 
info, just call the current page url but ask for JSON this time.

```haskell 
-- note: nowPlaying returns a 'Maybe NowPlaying' data type which holds 
-- (Just) all the information about the currently playing song or a 
-- 'Nothing' if there's, well, nothing playing.
getStatusR :: YesodMPC m => GHandler MPC m RepHtmlJson
getStatusR = do
    mnp <- nowPlaying
    defaultLayoutJson (htmlReply mnp) (jsonReply mnp)

    where
        -- here's what's returned if the client requests HTML:
        htmlReply mnp = do
            addJulius [$julius|
                // add your jQuery here (see below).
                |]

            case mnp of
                Just np -> [$hamlet| 
                    <h1>MPD
                    <div #artist>#{npArtist np}
                    <div #album>#{npAlbum np}
                    <div #title>#{npTitle np}
                    |]

                Nothing -> [$hamlet| Nothing playing? |]

        -- and what's returned if the client requests JSON data:
        jsonReply mnp = case mnp of
            Just np -> jsonMap
                [ ("status"  , jsonScalar $ "OK"       )
                , ("artist"  , jsonScalar $ npArtist np)
                , ("album"   , jsonScalar $ npAlbum  np)
                , ("title"   , jsonScalar $ npTitle  np)
                ]

            Nothing -> jsonMap
                [ ("status", jsonScalar $ "ERR"               )
                , ("error" , jsonScalar $ "MPD threw an error")
                ]
```

So now my JavaScript can be simplified (don't forget to source some 
jquery.js in the head of your page):

```javascript 
function getNowPlaying() {
    var delay = 1000; // seconds * 1000

    if (delay != 0) {
        $.getJSON(window.location.href, {}, function(o) { // <- Ajax jQuery style...
            if (o.status == "OK") {
                // update each div with the info in "o"
                $("#artist").text(o.artist);
                $("#album").text(o.album);
                $("#title").text(o.title);
            }
        });

        // and, loop again
        setTimeout("getNowPlaying();", delay);
    }
}

$(function() { getNowPlaying(); })
```

Pretty cool, huh?

[mpd]:        http://mpd.wikia.com/wiki/Music_Player_Daemon_Wiki "music player daemon"
[github-mpc]: https://github.com/pbrisbin/yesod-mpc              "yesod-mpc on github"
