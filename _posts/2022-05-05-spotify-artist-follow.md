---
layout: post
category: blog
title: Spotify Artist Follow
permalink: /blog/spotify-artist-follow
description: Spotify App To Follow Multiple Artists
image: spotify-follow.jpg
---

![Spotify Artist Follow](../../../img/spotify-follow.jpg)
<span class="credit">Photo by <a href="https://unsplash.com/@jaysung">Jehyun Sung</a> on <a href="https://unsplash.com/s/photos/invert?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a></span>

I recently helped with integrating Spotify APIs into a Wordpress website for a collective of musicians. The ask was for a public user to be able to follow a group of artists on Spotify through a single button click. It seemed to be simple enough, but as all engineers know, [reality has a surprising amount of detail](http://johnsalvatier.org/blog/2017/reality-has-a-surprising-amount-of-detail). I wasn't very familiar with PHP and relied heavily on Google. I did this on a Mac running Monterey, but most of the steps (except for website setup) are universally applicable.

### Create Spotify App

To begin with, we need a free or premium [Spotify](https://www.spotify.com/us/signup) user account. Then, we need to login to the [Spotify Developer](https://developer.spotify.com/dashboard/login) website, and create an app. Give it a name and a description.  

![](../../../img/spotify-create-app.jpg)

Then, hit `Edit Settings` on the app:

![](../../../img/spotify-app-created.jpg)

Enter a callback URL (enter `http://localhost/follow/callback.php` for now - this is the URL for my local test website) and hit `Add`, followed by `Save` at the bottom (this is the link to our website that will be redirected to after Spotify authorization call):
![](../../../img/spotify-app-settings.jpg)

### Authorization Flow

I needed a way for a user to login to Spotify, and authorize my app to follow artists on their behalf. I found some [sample code](https://github.com/spotify/web-api-auth-examples) from this [tutorial](https://developer.spotify.com/documentation/web-api/quick-start/) that did just that.

Based on the [Authorization Guide](https://developer.spotify.com/documentation/general/guides/authorization/), the [Authorization Code Flow](https://developer.spotify.com/documentation/general/guides/authorization/code-flow/) seemed like what I needed.
> "In scenarios where storing the client secret is not safe (e.g. desktop, mobile apps or JavaScript web apps running in the browser), you can use the [authorization code with PKCE](https://developer.spotify.com/documentation/general/guides/authorization/code-flow/), as it provides protection against attacks where the authorization code may be intercepted."

### Follow API

Next, I needed a bulk API to follow artists. I found this [one](https://developer.spotify.com/console/put-following/), to which I needed to pass the access token from the authorization call.

```javascript
curl PUT https://api.spotify.com/v1/me/following?ids=artist1,artist2,...,artistn
-H "Authorization: Bearer BQD.....jwq6c1PBd"
```

### Local test website setup

Since it was a Wordpress site, the backend had to be PHP. I created three PHP files - `login.php` for the authorization, `callback.php` to handle the callback from Spotify's authorize endpoint, and `follow.php` for the artist follow call.

Next, I setup a local PHP web server. PHP comes bundled with Mac OS pre-Monterey. Post-Monterey, it has to be installed using Homebrew. From a terminal window, type (this takes a while to finish):

```javascript
brew install PHP
```

There was a lengthy [detour](https://www.simplified.guide/macos/apache-php-homebrew-codesign) to self sign PHP, since Monterey doesn't accept unsigned packages. This involved setting up a local certificate authority and a code signing certificate.

Then, in the apache root folder (usually `/Library/WebServer/Documents`), I created a `follow` folder and dropped the `index.html`, `login.php` and `follow.php` files into it. 

### Home page

`index.html` is my one page test website. It uses [handlebars](https://handlebarsjs.com/) to populate markup with results from the API calls, and [Jquery](https://jquery.com/) to call the Spotify APIs.

### Login button

The `Login with Spotify` button redirects to `login.php`:

```javascript
document.getElementById('login-button').addEventListener('click', function() {
            window.location = 'http://localhost/follow/login.php';   
          }, false);
```

`login.php` redirects to Spotify's authorize endpoint (the `response_type` must be set to `code` for the authorization code and the redirect URI passed has to match the redirect URI in the Spotify app settings) :

```javascript
$client_id = "<client id from the Spotify app in developer dashboard>";  
$state = generateRandomString(16);

$scope = "user-follow-modify";

$url = "https://accounts.spotify.com/authorize";
$url .= "?response_type=code";
$url .= "&client_id=" . urlencode($client_id);
$url .= "&scope=" . urlencode($scope);
$url .= "&redirect_uri=" . urlencode($redirect_uri);
$url .= "&state=" . urlencode($state);

header("Location: " . $url);
```

Spotify authorize endpoint redirects back to `http://localhost/follow/callback.php` passing it the authorization code. `callback.php` exchanges this code for an access token by `POST`ing to the `https://accounts.spotify.com/api/token` endpoint. It cookies the access token and redirects back to the `index.html` page:

```javascript
$client_id = config()["client_id"];
$client_secret = config()["client_secret"];
$redirect_uri = config()["redirect_url"];

$url = "https://accounts.spotify.com/api/token";

$fields = [
    "code"          => urlencode($code),
    "redirect_uri"  => $redirect_uri,
    "grant_type"    => "authorization_code",
    "state" => $state
];
    
//url-ify the data for the POST
$fields_string = http_build_query($fields);
console_log("fields_string = " . $fields_string);

$result = http_request("POST", $url, 
    array(
        'authorization: Basic ' . base64_encode($client_id . ":" . $client_secret)
    ),
    $fields_string);
if ($result !== FALSE) {    
    $result_arr = json_decode($result, true);
    $access_token = $result_arr["access_token"];
    setcookie("t", $access_token, time()+1200); 
    setcookie("state", $state, time()+1200); 
    setcookie("id", $client_id);
    header("Location: /follow");    
``` 

### Profile information

Back on the `index.html` page, we call the `https://api.spotify.com/v1/me` endpoint to get user profile information (user name, etc.), and display the `Follow All` button. 

### Follow button

The `Follow All` button redirects to `follow.php` passing in `access_token` and `user_id` (which comes from the `/v1/me/` API call):

```javascript
document.getElementById('follow-button').addEventListener('click', function() {
  const urlParams = new URLSearchParams(window.location.href.split('#')[1]);
  const access_token = urlParams.get('access_token');
  var url = `http://localhost/follow/follow.php?token=${access_token}&id=${user_id}`;
  $.ajax({type: 'GET', url: url})
  .done(function(response) {
    artistsFollowedPlaceholder.innerHTML = artistsFollowedTemplate(response);
  })
  .fail(function(msg) {
    alert(`Failed with error: ${JSON.stringify(msg)}`);
  });
}, false);
```

`follow.php` calls the Spotify `/follow` API. The `/follow` API is limited to 50 artists per call. I got around that by calling `/follow` in batches of 50 at a time. Then, I call `/artists?ids=` to get the artist names and links, which are returned as the response to this call. An admin email is sent as well to track users who followed:

```javascript
$access_token = $_GET["access_token"];
    $spotify_id = $_GET["id"];
    $artist_ids = config()["artist_ids"];

    $api_root_url = "https://api.spotify.com/v1";
    $url = $api_root_url . "/me/following?type=artist&ids=" . $artist_ids;
    
    $startIdx = 0;
    $more = True;
    $artist_ids_arr = str_getcsv($artist_ids);
    $batch_size = 50;
    $result_json = [];
    $followed = FALSE;
    while ($more) {
        $artist_ids_batch = join(",", array_slice($artist_ids_arr, $startIdx, $batch_size));
        $result = http_request("PUT", $url, array('authorization: Bearer ' . $access_token));
        if ($result !== FALSE) {
            $url = $api_root_url . "/artists?ids=" . $artist_ids_batch;

            $result = http_request("GET", $url, array('authorization: Bearer ' . $access_token));
            $result_json = array_merge($result_json, json_decode($result,true)["artists"]);

            $followed = True;
        }
        else {
            $followed = False;
            $result_json = json_decode("{\"name\": \"Error following artists\",\"images\":[{\"url\":\"\"},{\"url\":\"\"},{\"url\":\"\"}]}");
            break;
        }
        $startIdx = $startIdx + $batch_size;
        if (count($artist_ids_arr)-1 < $startIdx) {
            $more = False;
        } 
    }

    if ($followed) {
        mail(config()["admin_email"], $spotify_id . "followed all artists on Auricle collective", "");
    }
    else {
        mail(config()["admin_email"], $spotify_id . "Failed to follow all artists on Auricle collective", "");
    }

    header('Content-Type: application/json; charset=utf-8');
    echo(json_encode($result_json));
```
### Logout button

When I submitted a quota extension request for the app, Spotify asked me to provide a way for users to disconnect from Spotify. I added a logout button which clears cookies and redirects to `logout.php`:
```javascript
document.getElementById('logout-button').addEventListener('click', function() {
	Cookies.remove("t", { path: '' });
	Cookies.remove("state", { path: '' });
	Cookies.remove("error", { path: '' });
	window.location = "/follow/logout.php";
}, false);
```

`logout.php` in turn redirects to Spotify's logout URL:
```PHP
<?php
    header("Location: https://www.spotify.com/logout/");
?>
```

This seems to work fine on Chrome. On Safari and Firefox, however, hitting the back button shows the user as logged in. It seems to be loading the page from cache. I tried putting in the usual HTTP no-cache headers, but Safari wouldn't budge. After some thrashing, I found this snippet on [Stackoverflow](https://stackoverflow.com/a/34236065) which did the trick. It reloads the page :

```javascript
window.addEventListener("pageshow", function(evt){
        if(evt.persisted){
        setTimeout(function(){
            window.location.reload();
        },10);
    }
}, false);
```

### App flow

That's it. The flow looks like this:
![](../../../img/spotify-follow-1 1.jpg)
![](../../../img/spotify-follow-2.jpg)
![](../../../img/spotify-follow-3.jpg)
![](../../../img/spotify-follow-4.jpg)
![](../../../img/spotify-follow-5.jpg)

### Source code for this article
[Github](https://github.com/cs31415/artist-follow)


### Final notes: 

- After publishing this code to your server, update the redirect URI in the Developer dashboard and in `login.php`. 

- Some improvements are possible to this code. I am currently passing access token in a cookie. I could probably make the `/v1/me` call in `callback.php` and pass back `client_id` in the cookie, and store the access token in session.

- The Spotify app starts out in Development mode. This means that any user other than yourself has to be explicitly granted access to the app through the Developer dashboard in order to use it. Once you have tested it, you can submit a quota extension request to make it publicly accessible for all users. The first time I did this, Spotify rejected it and asked me to provide a logout feature, add their logo, and provide links to artist content on Spotify. I had to resubmit and am still waiting to hear back.