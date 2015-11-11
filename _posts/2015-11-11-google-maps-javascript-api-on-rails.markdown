---
published: false
title: Google Maps Javascript API, on Rails
layout: post
tags: [javascript, google-maps, richmarker]
---
This is a guide to using Google Maps Javascript API on Rails 4.2.4, with Turbolinks enabled. Initially, when I simply included the Google Maps script inside the `<head>` tag, I kept getting this error:

```Uncaught ReferenceError: google is not defined```

This error was occurring because I was trying to create the map...

```javascript
var mapCanvas = document.getElementById('map-canvas');
var mapOptions = { center: new google.maps.LatLng(3.139003, 101.68685499999992) };
var map = new google.maps.Map(mapCanvas, mapOptions);
```

...before the Google Maps Javascript files had been loaded. The solution I used was to append a `callback` parameter when loading Google Maps script: `https://maps.googleapis.com/maps/api/js?v=3.exp&libraries=places&callback=init` where `init` was a function expression that created the map:

```javascript
var init = function() {
  var mapCanvas = document.getElementById('map-canvas');
  var mapOptions = { center: new google.maps.LatLng(3.139003, 101.68685499999992) };
  var map = new google.maps.Map(mapCanvas, mapOptions);
};
```

This worked, but I soon discovered that on certain pages, the map would only load when I refreshed the page. Turns out this is a common problem that has to do with how Turbolinks functions. From the Turbolinks documentation:

> Turbolinks makes following links in your web application faster. Instead of letting the browser reload the JavaScript and CSS between each page change, and spend extra HTTP requests checking if the assets are up-to-date, we keep the current instance alive and replace only the body and the title in the head.

So when I visited that page, Turbolinks was only changing the content of `<title>` and `<body>`, without reloading the scripts. So naturally, the callback that ran the `init` function never happened, because the Google Maps script had already been loaded (Turbolinks wasn't reloading it).

I tried the [jQuery-turbolinks](https://coderwall.com/p/fajmvq/fixing-the-map-doesn-t-show-up-until-i-refresh-when-working-with-turbolinks-in-ruby-on-rails) solution here, but it didn't quite work. Here's what worked for me:

1. **DO NOT** use jQuery-turbolinks.
2. Use two different events, `$(doument).ready` and `$(document).on('page:load')
    ```javascript
    // Function to load the Google Maps script with a callback
    var loadGoogleMaps = function() {
      var script = document.createElement('script');
      script.type = 'text/javascript';
      script.src = 'https://maps.googleapis.com/maps/api/js?v=3.exp&'+'libraries=places&'+'callback=init';
      document.body.appendChild(script);
    };
    
    // On page reload/first load (Google Maps script hasn't already loaded)
    $(document).ready(function() {
      loadGoogleMaps();
    });
    
    // When page is loaded via turbolinks (Google Maps script already loaded),
    // call the init function right away
    $(document).on('page:load', function() {
      init();
    });
    ```