PersistJS 0.3.1 README
======================

Table of Contents
-----------------
  1. Introduction
  2. Rationale
  3. Using PersistJS
  4. Size Limits
  5. Other Limits
  6. Extending PersistJS
  7. Where is it being used?
  8. About the Author

1. Introduction
---------------
PersistJS is a JavaScript client-side persistent storage library.

PersistJS features include:

  * Small (<10k minified, 3k gzipped)
  * Standalone: Does not need any additional browser plugins or
    JavaScript libraries to work on the vast majority of current
    browsers.
  * Consistent: Provides a consistent, opaque API, regardless of
    the browser.
  * Extensible: Custom backends can be added easily.
  * Backwards Compatible: Can fall back to flash or cookies if no
    client-side storage solution for the given browser is available.
  * Forwards Compatible: Supports the upcoming versions of Internet
    Explorer, Firefox, Chrome, and Safari (Opera too, if you have Flash).
  * Unobtrusive: Capability testing rather than browser detection, so
    newer standards-conformant browsers will automatically be supported.

The latest version of PersistJS is always available online at the
following URL:

    http://github.com/jeremydurham/persist-js

2. Rationale
------------
Why use PersistJS?  What's the problem with using cookies directly or
simply requiring Flash?

Currently the only reliable cross-platform and cross-browser mechanism
for storing data on the client side are cookies.  Unfortunately, using
cookies to store persistent data has several problems:

  * Size: Cookies are limited to about 4 kilobytes in size.
  * Bandwidth: Cookies are sent along with every HTTP transaction.
  * Complexity: Cookies are difficult to manipulate correctly.

Modern web browsers have addressed these issues by adding non-Cookie
mechanisms for saving client-side persistent data.  Each of these
solutions are simpler to use than cookies, can store far more data, and
are not transmitted along with HTTP requests.  Unfortunately, each
browser has addressed the problem in a different and incompatible way.
There are currently 5 different client side persistent data solutions:

  * globalStorage: Firefox 2.0+, Internet Explorer 8
  * localStorage: development WebKit (Safari, iPhone, etc)
  * openDatabase: Safari 3.1+
  * userdata behavior: Internet Explorer 5.5+
  * Google Gears: Chrome

Some developers have attempted to address the client side storage
issue with the following browser plugins:

  * Adobe Flash
  * Google Gears

The problem with relying on plugins, of course, is that users without
the plugin installed miss out on the feature in question, and your
application is dependent on software from a particular vendor.  Google
Gears, for example, is not widely deployed.  Flash is, but it has
problems of its own:

  * Many users block Flash or require a click in order to enable
    flash content; this makes Flash unsuitable as a transparent,
    client-side data store.
  * Flash is notoriously unreliable on newer 64-bit machines.
  * Some businesses block Flash content as a security measure.

Anyway, if we include Gears and Flash, that means there are no less than
6 incompatible solutions for storing client-side persistent data.  

The most notable attempt at addressing this problem is probably Dojo
Storage.  Unfortunately, Dojo Storage does not support Internet Explorer
without Flash, and it does not support Safari or other WebKit-based
browsers at all (at least, not without Flash).  Also, Dojo Storage is
not standalone; it requires a several other Dojo components in order to
operate.

PersistJS addresses all of the issues above.  It currently supports
persistent client-side storage through the following backends:

   * flash:         Flash 8 persistent storage.
   * gears:         Google Gears-based persistent storage.
   * localstorage:  HTML5 draft storage.
   * globalstorage: HTML5 draft storage (old spec).
   * ie:            Internet Explorer userdata behaviors.
   * cookie:        Cookie-based persistent storage.

Each backend is wrapped by PersistJS and exploses the exact same
interface, which means you don't have to know or care which backend is
being used.  The next section explains how to use the PersistJS API.

3. Using PersistJS
------------------
Using PersistJS is fairly straightforward.  First, you include
`persist-min.js` in your web site:

    <head>
      <title>My Wonderful Page</title>
      <script type='text/javascript' src='persist-min.js'></script>
    </head>

After the DOM has loaded, you create a persistent store object:

    // create new store named "My Application"
    var store = new Persist.Store('My Application');

The store constructor has one required parameter: a store name.  You can
create as many stores as you'd like, as long as they each have a unique
name.  Store names should begin with a letter, and can consist of upper
and lower case letters, numbers, spaces, and dashes.  

As I mentioned before, you shouldn't create a persistent store until
after the DOM has loaded.  The easiest browser-agnostic way to do this
is to set an `onload` handler on the `body` element, like this:

    <body onload='load_data();'>

And the JavaScript:

    // global object
    var store;

    function load_data() {
      // load persistent store after the DOM has loaded
      store = new Persist.Store('My Application');
    }

Most popular JavaScript libraries such have their own way of adding DOM
ready handlers.  Here's how you do it in jQuery:

    $(function() {
      // load persistent store after the DOM has loaded
      store = new Persist.Store('My Application');
    });

And in YUI:

    function init() {
      store = new Persist.Store('My Application');
    }

    // call when the DOM has loaded
    YAHOO.util.Event.onDOMReady(init);

Anyway, after you have created a persistent store, you save values to
the it:

    // save data in store
    store.set('some_key', 'this is a bunch of persistent data');

Note that the value must be a string.  If you want to save structured
data like arrays or hashes, you should serialize it using Array.join or
JSON.

Once you have saved a value to the store, you can read it back, 
like this:

    // get value from store and prompt user
    val = store.get('some_key')

Here's an example of removing a key:

    // remove key from store and prompt user
    store.remove('some_key') 
    // prompt user
    alert('some_key was removed');

If you're in a hurry, then you can stop reading right now, because
that's all you need to know!

Still here?  Okay, here are some additional details.  When you create a
new store, you can also pass a hash of optional parameters, like so:

    // create a new deferred data store with a description
    var store = new Persist.Store('My Data Store', {
      about: 'This is my data store.',
      defer: true
    });

These parameters allow you to pass additional information or fine-tune
the behavior of the data store.  Here's a complete list of the available
parameters:

  * defer:    Defer saving until `save()` is called (used by `ie`).  See
              below for additional details.
  * domain:   Limit store to given domain or sub-domain (used by `cookie`
              and `globalstore`).
  * expires:  Number of days before store expires (used by `cookie`).  
              Defaults to 2 years (730 days).
  * path:     Limit store to given path (used by `cookie`).
  * size:     Estimated size of data set (used by `whatwg_db`).
  * swf_path: Path to file `persist.swf` (used by `flash`).  Defaults to
              `./persist.swf` if unspecified.

Notes: The `defer` option exists because there is no way to load and
save individual keys in the `ie` backend.  By default, the `ie` backend
will load all data when getting a value, and save all data when setting
a value.  When the `defer` flag is set, the store data will only be
loaded when the store is created or when the `load()` method is called.  

More importantly, data will _not_ be saved unless the `save()` method is
called.  If you choose to use the `defer` flag, the easiest way to make
sure `save()` is called is to use an unload handler, like so:

    <body unload='save_data();'>

And the JavaScript:

    function save_data() {
      // save store data
      store.save();
    }

It's probably best not to use this feature unless you really need it.

One final note about the Flash and Gears backends: They will be disabled
unless you include swfobject.js and gears_init.js, respectively.  These
files are available in the extras/ directory.

If you'd rather include all of these in one combined file (to enable and
check for all possible backends), you can use the file
`extras/persist-all-min.js` in place of `persist-min.js`.
`persist-all-min.js` is the the following files concatenated together
and minified:
  
  * extras/gears_init.js
  * extras/swfobject.js
  * persist-min.js

Note that `persist-all-min.js` is roughly 60% larger than
`persist-min.js`.

4. Size Limits
--------------
Each backend has a different data size limit.  While you generally
aren't concerned about _which_ backend is being used, you may care about
the amount of data you are able to store.  

To deal with this, the attribute `Persist.size` is set to the
_approximate_ size limit, in bytes, of the active backend.  For backends
where the size limit is unlimited or unknown, `Persist.size` is set to
`-1`.  Here's a rough breakdown of the size limits for each backend:

  * cookie:         4 kilobytes
  * gears:          unknown
  * flash:          unknown (at least 100k)
  * globalstorage:  5 megabytes
  * ie:             64 kilobytes
  * localstorage:   unknown 

(Note that the key length is also included in the data size limit).

Rather than testing for a specific backend, it is probably better to
calculate the approximate size of the data that you need to save, and
then prompt the user if there is insufficient space.  For example:

    var lots_of_data = '...'; // value with lots of data

    try {
      // check size of data
      if (Persist.size != -1 && Persist.size < lots_of_data.length)
        throw new Error('too much data');

      // try and save data
      store.set('some_key', lots_of_data);
    } catch (err) {
      // display save error
      alert("Couldn't save data: " + err);
    }

If you absolutely _must_ know which backend is in use, you can do so by
checking the `Persist.type` value.  Also, if you'd like to disable a
specific backend, use `Persist.remove()`, like this:

    // disable "cookie" backend (will never be selected)
    Persist.remove('cookie');

5. Other Limits
---------------
The `cookie` backend is limited by the number of maximum number of
cookies that can be saved by the browser.  Older browsers typically
limited the number of cookies to 20 per domain, although newer browsers
have increased this limit to 50 cookies per domain.  

You can work around this limit by serializing your data as JSON or
some other format.

The `cookie` backend is the only backend with any practical limit on the
number of keys.

6. Extending PersistJS
----------------------
PersistJS exposes one method -- `Persist.add()` -- for extending
PersistJS and adding custom backends.  This method is currently
undocumented and may change in future versions.

PersistJS also includes a full copy of EasyCookie 0.2.1, which is
exposed as Persist.Cookie.  For documentation on the EasyCookie API, see 
the EasyCookie page at:

  http://pablotron.org/software/easy_cookie/

7. Where is it being used?
-------------------------
* Beacon Interactive Systems (http://www.beaconinteractive.com)
* MochaUI
* tDash (http://tdash.org) - the only Twitter client that works in your
    browser, directly talking to Twitter.

8. About the Author
-------------------
Paul Duncan (pabs@pablotron.org) - http://pablotron.org/

9. Contributors
---------------
Jeremy Durham (jeremydurham@gmail.com) - http://www.jeremydurham.com
Marcus Spiegel (marcus.spiegel@gmail.com) - http://marcusspiegel.de
Matt Pizzimenti (mjpizz+github@gmail.com) - http://mjpizz.com
Mayank Sharma (mayanks@gmail.com) - http://mayanks.blogspot.com
Maximilian Batz (m.batz@ideaday.de) - http://www.ideaday.de