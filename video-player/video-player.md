#Basic Cross Browser Video Player

This article describes a simple HTML5 video player that uses the Media and Fullscreen APIs and works across most major desktop and mobile browsers. The player itself won't be styled beyond that which is required to get it working, full styling of the player will be taken care of in a future article.

##HTML Markup

To start off with, let's take a look at the HTML that makes up the player.

First of all the ``video`` element is defined, contained within a ``figure`` element which acts as the video container. To anyone familiar with HTML5 markup and the video element, there should be nothing here that surprises you.

```html
<figure id="videoContainer">
   <video id="video" controls preload="metadata" poster="img/poster.jpg">
      <source src="video/tears-of-steel-battle-clip-medium.mp4" type="video/mp4"></source>
      <source src="video/tears-of-steel-battle-clip-medium.webm" type="video/webm"></source>
      <source src="video/tears-of-steel-battle-clip-medium.ogg" type="video/ogg"></source>
      <!-- Flash fallback -->
      <object type="application/x-shockwave-flash" data="flash-player.swf?videoUrl=video/tears-of-steel-battle-clip-medium.mp4" width="1024" height="576">
         <param name="movie" value="flash-player.swf?videoUrl=video/tears-of-steel-battle-clip-medium.mp4" />
         <param name="allowfullscreen" value="true" />
         <param name="wmode" value="transparent" />
         <param name="flashvars" value="controlbar=over&amp;image=img/poster.jpg&amp;file=flash-player.swf?videoUrl=video/tears-of-steel-battle-clip-medium.mp4" />
         <img alt="Tears of Steel poster image" src="img/poster.jpg" width="1024" height="428" title="No video playback possible, please download the video from the link below" />
      </object>
      <!-- Offer download -->
      <a href="video/tears-of-steel-battle-clip-medium.mp4">Download MP4</a>
   </video>
</figure>
```

Even though this player will define its own custom control set, the ``controls`` attribute is still added to the ``video`` element, with the player's default control set being switched off later with JavaScript. Doing things this way still allows users who have JavaScript turned off (for whatever reason) to still have access to the browser's native controls.

A poster image is defined for the video, and the ``preload`` is set to ``metadata`` which informs the browser that it should only attempt to load the metadata off the video file rather than the entire video file. This provides the player with data such as video duration.

Three different video sources are provided for the player: MP4, WebM, and Ogg. Using these different source formats gives the best chance of being supported across all browsers that support HTML5 video. For further information on video formats and browser compatibility, see [supported media formats](https://developer.mozilla.org/en-US/docs/HTML/Supported_media_formats#Browser_compatibility).

For browsers that do not support HTML5 video, a Flash player is provided that will allow playback of the MP4 video source, provided the end user has Flash installed. In addition a download link is displayed which allows users to download the MP4 video file, should they wish to (and providing those without Flash installed with a method of viewing the video, a fallback for a fallback if you like). 

The code above would allow playback of the video in most browsers, using the browser's default control set. The next step is to define a custom control set, also in HTML, which will be used to control the video.

##Control Set

Most browser's default video control set contain the following controls:
* play/pause
* mute
* volume control
* progress bar
* go fullscreen

The custom control set will also support this functionality, with the addition of a stop button.

Once again the HTML is quite straight forward, using an unordered list with ``list-style-type:none`` to enclose the controls each of which is a list item with ``float:left``. For the progress bar, the ``progress`` element is taken advantage of, with a fallback provided for browsers that don't support it (e.g. IE8 and IE9). This list is inserted after the ``video`` element, but inside the ``figure`` element (this is important for the fullscreen functionality, which is explained later on).

```html
<ul id="video-controls" class="controls">
   <li><button id="playpause" type="button">Play/Pause</button></li>
   <li><button id="stop" type="button">Stop</button></li>
   <li class="progress">
      <progress id="progress" value="0" min="0">
         <span id="progress-bar"></span>
      </progress>
   </li>
   <li><button id="mute" type="button">Mute/Unmute</button></li>
   <li><button id="volinc" type="button">Vol+</button></li>
   <li><button id="voldec" type="button">Vol-</button></li>
   <li><button id="fs" type="button">Fullscreen</button></li>
</ul>
```

Each button is given an id so it can be easily accessed with JavaScript. The ``span`` within the ``progress`` is for [browsers that do not support the ``progress`` element](http://caniuse.com/#search=progress) and will be updated at the same time as ``progress`` is (this ``span`` element won't be visible on browsers that support ``progress``).

The controls are initially hidden with a CSS ``display:none`` and will be enabled with JavaScript. Again if a user has JavaScript disabled, the custom control set will never appear and they can use the browser's default control set.

Of course this custom control set is currently useless and doesn't do a thing, and this can only be chaned with JavaScript.

##Using the Media API

As you may already know, HTML5 comes with a JavaScript [Media API](https://developer.mozilla.org/en/docs/Web/API/HTMLMediaElement) that allows developers access to and control of HTML5 media. Naturally this API is what will be used to make the custom control set defined above actually do something. In addition, the fullscreen button will use the [Fullscreen API](https://developer.mozilla.org/en-US/docs/Web/Guide/API/DOM/Using_full_screen_mode), a W3C API.

###Setup

Before dealing with the individual buttons, a number of initialisation calls are required.

To begin with, it's a good idea to first check if the browser actually supports the ``video`` element and to only setup the custom controls if it does. This is done by simply checking if a created ``video`` element supports [the ``canPlayType()`` method](http://www.w3.org/html/wg/drafts/html/master/embedded-content.html#dom-navigator-canplaytype), which any supported HTML5 ``video`` element should.

```javascript
var supportsVideo = !!document.createElement('video').canPlayType;
if (supportsVideo) {
   // set up custom controls
   // ...
}
```

Once it has been confirmed that the browser does indeed support HTML5 video, it's time to set up the custom controls. A number of handles to HTML elements are required:

```javascript
var videoContainer = document.getElementById('videoContainer');
var video = document.getElementById('video');
var videoControls = document.getElementById('video-controls');
```

As mentioned earlier, the browser's default controls now need to be disabled, and the custom controls need to be displayed:

```javascript
// Hide the default controls
video.controls = false;

// Display the user defined video controls
videoControls.style.display = 'block';
```

With that done, a handle to each of the buttons is now required:

```javascript
var playpause = document.getElementById('playpause');
var stop = document.getElementById('stop');
var mute = document.getElementById('mute');
var volinc = document.getElementById('volinc');
var voldec = document.getElementById('voldec');
var progress = document.getElementById('progress');
var progressBar = document.getElementById('progress-bar');
var fullscreen = document.getElementById('fs');
```

Using these handles, events can now be attached to each of the custom control buttons making them interactive. Most of these buttons require a simple ``click`` event listener to be added, and a Media API defined method and/or attributes to be called/checked on the video.

###Play/Pause

```javascript
playpause.addEventListener('click', function(e) {
   if (video.paused || video.ended) video.play();
   else video.pause();
});
```

When a ``click`` event is detected on the play/pause button, the handler first of all checks if the video is currently paused or has ended (via the Media API's ``paused`` and ``ended`` attributes), and if so, it uses the ``play()`` method to playback the video. Otherwise the video must be playing, so it is paused using the ``pause()`` method.

###Stop

```javascript
stop.addEventListener('click', function(e) {
   video.pause();
   video.currentTime = 0;
   progress.value = 0;
});
```

The Media API doesn't have a 'stop' method, so to mimic this, the video is paused, it's ``currentTime`` (i.e. the video's current playing position) is reset and so is the progress element's position (more on that later).

###Mute

```javascript
mute.addEventListener('click', function(e) {
   video.muted = !video.muted;
});
```

The mute button is a simple toggle button which uses the Media API's ``muted`` attribute which is a Boolean indicating whether the video is muted or not.

###Volume

```javascript
volinc.addEventListener('click', function(e) {
   alterVolume('+');
});
voldec.addEventListener('click', function(e) {
   alterVolume('-');
});
```

Two volume control buttons have been defined, one for increasing the volume and another for decreasing it. A user defined function, ``alterVolume(direction)`` has been created which deals with this:

```javascript
var alterVolume = function(dir) {
   var currentVolume = Math.floor(video.volume * 10) / 10;
   if (dir === '+') {
      if (currentVolume < 1) video.volume += 0.1;
   }
   else if (dir === '-') {
      if (currentVolume > 0) video.volume -= 0.1;
   }
}
```

This function makes use of the Media API's ``volume`` attribute which holds the current volume value of the video. Valid values for this attribute are 0 and 1 and anything in between. The function checks the ``dir`` parameter which indicates whether the volume is to be increased (+) or decreased (-) and acts accordingly. The function is defined to increase or decrease the video's ``volume`` attribute in steps of 0.1, ensuring that it doesn't go lower than 0 or higher than 1.