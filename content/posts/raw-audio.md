---
title: Raw Audio
date: 2010-05-28
tags: [android]
---

{{< well >}}
I am not an android developer. I am not even a Java developer. What appears
below is my attempt to share what I've learned using the android documentation,
google, and copious amounts of trial and/or error. I apologize for mistakes;
please offer corrections via twitter or email.
{{< /well >}}

I was upset when I first got my Droid Eris that the media player
couldn't load a url by default. I stream my mpd out to the world so
that I can pick it up away from home. I had
[read](http://developer.android.com/guide/topics/media/index.html)
that the built-in MediaPlayer object in android supports any uri
that offered progressive download of media in a
[format](http://developer.android.com/guide/appendix/media-formats.html)
supported by the device (both ogg and mp3 are supported). So why
not expose this feature in the UI?

I soon found out that the only way to play a custom audio stream
from the internet was to purchase StreamFurious Pro. Not a bad app,
but overkill for my situation. I had to load up this internet radio
player with all these built in stations just to load my own custom
url. I then had to wait literally minutes while it buffered a
hard-coded \~5MB before beginning to play. Garbage.

So why not write [my own](http://github.com/pbrisbin/raw-audio)?

## Android

I decided to take this as an opportunity to get my feet wet with
Android (and Java in general). I figured I could write a simple App
to accept a URL entered into a text box and load it using the
built-in MediaPlayer() class. Because of the underlying simplicity,
you could even pass `file:///foo/bar.mp3` and it would work.

Well, I did that. And it even works surprisingly well; I figured I
could post what I've done, and hopefully it'll be useful to
others.

## Activities and Services

My App would use two of the major building blocks in Android:
Activities and Services.

An Activity is the object, the class, that runs when you first open
your UI. It basically *is* the UI. It handles accepting user input
and doing something with it. It's designed to only be Active while
the user is viewing your UI. Once your App closes, it can and will
be safely killed.

So what if you need to do some persistent background work? Like,
say, playing music? That requires a Service.

A Service is another class that runs in the background. It can be
created, bound to, and controlled by your Activity. It gets a
slightly higher priority in the grand scheme of things, so it's
less likely to get killed when the user's not interacting with your
Activity.

## The IDE

First, let me describe a little bit about my development
environment because it handles a lot of the back-end stuff that I'm
not going to touch on here.

First thing I did was install a few packages. Android-sdk of
course, eclipse-android (available in the AUR to Arch users), and a
nice little package called eclim.

Eclim is awesome. It's a vim plugin that let's me hook into eclipse
from within vim. I can do all my coding in my favorite editor ever,
and let eclipse handle the heavy lifting. Things like:
auto-completion of method names, automatic imports, syntax
checking, and documentation lookups.

I also hop back over to eclipse to do my compiling, and
installation.

I'll leave setting up your sdk and eclipse project to you since
there's a ton of OS/distro-specific ways to do it;
[google](http://www.google.com/search?q=developing%20java%20in%20(linux%7Cmac%7Cwindows))
is your friend.

## The UI

It's possible to define your UI programmatically from within your
main Activity. This is frowned upon and not only is it a best
practice but it's also wicked convenient to define your UI via an
xml file instead.

Under your project directory are two very important
sub-directories: src and res. In your src folder is the actual Java
*source*, and in your res directory are your *resources*. Things
like images, static string values, and xml layout files. With a
vanilla package template, your UI is defined in
`res/layout/main.xml`.

Here's my `res/layout/main.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
                android:layout_width="fill_parent" 
                android:layout_height="wrap_content"
                android:padding="15dip" >

    <TextView android:id="@+id/label" 
              android:layout_width="fill_parent" 
              android:layout_height="wrap_content" 
              android:text="@string/label" />

    <EditText android:id="@+id/entry" 
              android:layout_width="fill_parent" 
              android:layout_height="wrap_content" 
              android:background="@android:drawable/editbox_background"
              android:layout_below="@id/label"
              android:text="@string/my_server" />

    <Button android:id="@+id/playpause"
            android:layout_width="100dip"
            android:layout_height="wrap_content"
            android:layout_below="@id/entry"
            android:layout_alignParentLeft="true"
            android:text="@string/play_pause" />
</RelativeLayout>
```

Here I've defined a layout where objects are placed relative to
each other, with a TextView label with id "label", an EditText
entry box with id "entry", and a single Play/Pause button with id
"playpause".

You'll also notice I use the `@string/foo` convention. This means *pull the
value from the res/values/strings.xml file*.

Here's mine:

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="app_name">Raw Audio</string>
    <string name="my_server">https://pbrisbin.com:8000/mpd.mp3</string>
    <string name="label">Enter stream url:</string>
    <string name="play_pause">Play/Pause</string>
</resources>
```

So `@string/play_pause` is a variable for "Play/Pause".

{{< well >}}
I pre-populate `@id/entry` with my server's address
(`@string/my_server`). I only need this App to pickup my mpd
stream, so why should I always have to type it? This way, it's
there, but I can overwrite it if I want to.
{{< /well >}}

## The Activity

Now that that's out of the way, onto the Java.

I define a class `RawAudio` which *extends* the Android base class
`Activity`.

One thing I do differently is *implement* `OnClickListener`. I picked 
this up in one of my google results, and I find it to be a much cleaner 
approach to button handling.

This allows us to assign the keyword `this` (meaning the very class that 
we are) as an `OnClickListener` for any of our buttons. That way, when 
we click that button, our very own `onClick()` will be called.

I know what your thinking: *But, what if you have more than one button? 
Don't you need an OnClickListener for each of them?* Of course not; 
`onClick()` is passed a view that corresponds to the button that was 
clicked, so with a simple `switch` statement we could determine what 
button was clicked and act accordingly.

Sweet, I know, but I've only got one button so I don't do that here. 
Anyway, here's the top portion of my RawAudio class:

```java
package rawAudio.apk;

import android.app.Activity;

import android.content.ComponentName;
import android.content.Intent;
import android.content.ServiceConnection;

import android.os.Bundle;
import android.os.IBinder;

import android.util.Log;

import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;
import android.widget.EditText;

public class RawAudio extends Activity implements OnClickListener {
    private static final String APP = "RawAudio";

    // our background player service
    private StreamMusic streamMusic = null;

    @Override public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        // setup ui
        setContentView(R.layout.main);

        // setup play/pause button
        Button ppButton = (Button)findViewById(R.id.playpause);
        ppButton.setOnClickListener(this);

        // connect/create our service
        connectToService();
    }
```

First we have our overridden `onCreate()` call. This method is run
when your App is launched. You can think of it like `main()` or
`run()` from other languages. There are other important "lifecycle"
methods you should look into, but this and `onDestroy()` are the
only two I use here.

I use this starting point to setup my UI based on `main.xml` via
`setContentView()`. Notice the argument is almost literal:
*R.layout.main -\> res/layout/main.xml*.

I also create a Button object based on `R.id.playpause` from our
`main.xml` file and set `this` as its `onClickListener`.

Lastly, I `connectToService()`.

So at this point time-wise, I can assume that my UI is loaded and
I'm connected to my service, ready for user input.

The other overridden method is `onDestroy()`. This is called when
your App is exited or killed. The only reason I need to override it
is so that I can disconnect from the service. `unbindService()` is
a built-in method that I don't need to define, I just have to call
it. `onService` will be defined later.

```java 
    @Override public void onDestroy() {
        // disconnect from our service
        unbindService(onService);

        super.onDestroy();
    }
```

Now that I've mapped things out at a high level, all that's left is
to fill in the blanks: what to do `onClick()`, how to
`connectToService()`, etc.

So take a break, make sure you've understood up to this point, then
continue onto the "plumbing" below.

## Play/Pause

I decided to leave the logic of how to handle a Play/Pause "event"
up to the Service. I would send over the command, and it would
determine if it should load or pause a stream by its current
internal state.

For this reason, `onClick()` is fairly simple. We get the stream
url from the text box, pass it and `this` as arguments to a Service
method.

I'll explain more when we get to it. But basically, `streamMusic`
is an instance of our service. It will be `null` when we're not
connected to the service. It also has the public method
`playPause()` which will be written to accept a context (`this`)
and a stream url. It will take the appropriate action of loading,
pausing, or resuming the stream.

```java
    public void onClick(View v) {
        if (streamMusic != null) {
            EditText t = (EditText)findViewById(R.id.entry);
            String   s = t.getText().toString();

            // send play/pause command
            streamMusic.playPause(this, s);
        }
        else {
            Log.d(APP, "service not bound yet.");
        }
    }
```

We've left the most important piece for last. I'm not going to say
much because I don't fully understand it, but it's pretty
copy/paste-able code for situations where you need to bind to a
service. All I know is that it instantiates `streamMusic` on
connect and resets it to `null` on disconnect; that's good enough
for me.

For this to work, you have to define `StreamMusic.LocalBinder` when
you write your Service, but I'll get to that...

```java
    // establish the connection
    private void connectToService() {
        try {
            bindService(new Intent(this, StreamMusic.class), onService, BIND_AUTO_CREATE);
        }
        catch (Exception e) {
            Log.e(APP, "error binding to service.", e);
        }
    }

    // once connected, set up the interface object
    private ServiceConnection onService = new ServiceConnection() {
        public void onServiceConnected(ComponentName className, IBinder iBinder) {
            streamMusic = ((StreamMusic.LocalBinder)iBinder).getService();
        }

        public void onServiceDisconnected(ComponentName className) {
            streamMusic = null;
        }
    };
```

That's it for our Activity. I hope all that made sense. It's all
quite useless however without our Service.

## The Service

Besides actually creating and using the Service, you also have to
add a `service` tag to your manifest. Here's my
`AndroidManifest.xml`.

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
      package="rawAudio.apk"
      android:versionCode="1"
      android:versionName="1.0">
      <application android:label="@string/app_name" 
                   android:icon="@drawable/icon48x48" >
        <activity android:name=".RawAudio"
                  android:label="@string/app_name">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        <service android:name=".StreamMusic" />
    </application>
</manifest>
```

Eclipse took care of all of that save for the `android:icon` and
`service` tags which I added by hand.

{{< well >}}
If you're using an eclipse-eclim-vim setup like I am, don't edit
the Manifest in vim. Since it's used to do the precompiling in
eclipse, it somehow gets ahead of itself and throws errors. Just
edit that one in eclipse directly.
{{< /well >}}

My Service at this point does a few things:

-   Intelligently handle the Play/Pause command
-   Load a stream using `MediaPlayer()`
-   Show a Progress Notification when connecting

That last piece is why we need to pass `this` in the `playPause()`
method.

So the first thing we do is setup that binder voodoo that I still
don't fully understand ;). I'll also override `onDestroy()` to stop
the music.

```java 
package rawAudio.apk;

import java.io.IOException;

import android.app.ProgressDialog;
import android.app.Service;
import android.content.Context;
import android.content.Intent;

import android.media.MediaPlayer;
import android.os.Binder;
import android.os.Handler;
import android.os.IBinder;
import android.os.Message;

import android.util.Log;

public class StreamMusic extends Service {
    private static final String APP = "StreamMusic";

    private static MediaPlayer mediaPlayer = null;
    private static String      STREAM      = "";

    private final Binder binder = new LocalBinder();

    private ProgressDialog pd;

    @Override public IBinder onBind(Intent intent) {
        return(binder);
    }

    @Override public void onDestroy() {
        // stop the music
        mediaPlayer.stop();
        mediaPlayer = null;

        super.onDestroy();
    }

    public class LocalBinder extends Binder {
        StreamMusic getService() {
            return (StreamMusic.this);
        }
    }
```

Next up, we define the main interface with our activity: the
`playPause()` handler.

```java
    // called from the main activity
    synchronized public void playPause(Context _c, String _s) {
        if (mediaPlayer == null) {
            // show progress dialog
            pd = ProgressDialog.show(_c, "Raw Audio", "connecting to stream...", true, true);

            // not playing -> load stream
            STREAM = _s;
            loadStream();
        }
        else if (mediaPlayer.isPlaying()) {
            // playing -> pause
            mediaPlayer.pause();
        }
        else {
            // paused -> resume
            mediaPlayer.start();
        }
    }
```

Here you can see why we passed the UI Context in with the stream.
Our progress notification needs a context in which to display
itself. And if it's not the main UI context, your App crashes. By
simply passing `this` along to the service we can easily deploy our
progress notification to it. Pretty sweet.

I'm not sure if `synchronized` is strictly needed but it was in the
code I was working from and it makes sense logically. I haven't
tested omitting it so who knows.

## Loading...

Again, we save the meat for last. The following two methods do some
special things.

First, we setup and run the actual `mediaPlayer` in its own thread.
We define a `Runnable` object with a `run()` that does the actual
work. Then we use the `Thread` class to start it. Don't ask me how
it works, just copy the code :).

```java
    private void loadStream() {
        // run the stream in its own thread
        Runnable r = new Runnable() {
            public void run() {
                try {
                    mediaPlayer = new MediaPlayer();

                    mediaPlayer.setDataSource(STREAM);
                    mediaPlayer.prepare();
                    mediaPlayer.start();

                    // dismiss the dialog
                    handler.sendEmptyMessage(0);
                }
                catch (IOException e) {
                    Log.e(APP, "error loading stream " + STREAM + ".", e);
                    return;
                }
            }
        };

        new Thread(r).start();
    }
```

You can see when the stream loads, we call
`handler.sendEmptyMessage(0)`. This is because we can't directly
dismiss our progress notification from inside that thread. We need
to somehow let the external context know that we're done. We send
out an empty message to a handler which actually dismisses the
notification:

```java
    // a handler to dismiss the dialog once mp starts
    private Handler handler = new Handler() {
        @Override public void handleMessage(Message msg) {
            pd.dismiss();
        }
    };
```

And there you go; Service done.

## Screenshots

As always, the obligatory screenshots.

![Raw Audio](/images/raw-audio/raw-audio.png)

![Raw Audio Progress](/images/raw-audio/raw-audio-progress.png)

## References

Sadly, I can't track down all the various google results that I
used while writing this thing but here are a few I could dig up.

-   Android docs regarding the
    [Service type](http://developer.android.com/reference/android/app/Service.html)
    -- full of great examples
-   This guy's
    [git tree](http://github.com/commonsguy/cw-android/tree/master/Service/WeatherPlus/)
    -- great example of Activity/Service interaction
-   This
    [tutorial](http://www.helloandroid.com/tutorials/using-threads-and-progressdialog)
    on using a Progress notification inside a thread
