# The Ajaxson5

One of the classic problems in computer science is the question of how to optimally implement a web page that displays GIFs of the Jackson 5. Today you'll take a stab at it.

## The Giphy API

You are going to use the [Giphy API](https://github.com/Giphy/GiphyAPI), whose purpose, as you might guess, is to serve up GIFs to developers.

The basic goal of your page will be to fetch a GIF from Giphy, and then insert it into the DOM to display it on your page. We'll get into the details shortly, but first, let's play with Giphy for a minute.

#### The host

The host url we want is `https://api.giphy.com/v1/gifs/`. Let's try sending a GET request to that url. Try entering the following command into your terminal:

```nohighlight
$ curl -G https://api.giphy.com/v1/gifs/
{"meta":{"status":404,"msg":"Not Found!"}}
```

We received a 404 Not Found response from Giphy. Maybe they ran out of GIFs?

#### The Endpoint

Actually, the problem is that we need to choose an "endpoint" to indicate which specific service we want. The endpoints are all listed in the documentation linked above, so check them out if you are interested. For this assignment, we will use the Random endpoint, which, upon receiving a request, responds with a randomly chosen GIF related to a particular topic (unless you don't provide a specific topic, in which case you'll get back a *very* random GIF).

The url for the Random endpoint is simply `/random`, so let's try this:

```nohighight
$ curl -G https://api.giphy.com/v1/gifs/random
{"message":"No API key found in request"}
```

This time we got a different error, 403 Forbidden. Like most APIs, Giphy requires us to authorize ourselves with a key. Luckily, they make a public "Beta" key available for people like us who are just testing, playing around, or doing LaunchCode Assignments. The key is "dc6zaTOxFJmzC".

#### Adding Params: api_key

Let's authorize ourselves by including in our request a parameter whose value is that magic string and whose key name is "api_key".

We can associate data with our curl request by using the -d flag:

```nohighlight
$ curl -G https://api.giphy.com/v1/gifs/random -d api_key=dc6zaTOxFJmzC
{"data":{"type":"gif","id":"RBLigAVE0xJte","url":"http:\/\/giphy.com\/gifs\/food-dessert-etc-RBLigAVE0xJte","image_original_url":"http:\/\/media1.giphy.com\/media\/RBLigAVE0xJte\/giphy.gif","image_url":"http:\/\/media1.giphy.com\/media\/RBLigAVE0xJte\/giphy.gif","image_mp4_url":"http:\/\/media1.giphy.com\/media\/RBLigAVE0xJte\/giphy.mp4","image_frames":"35","image_width":"245","image_height":"180","fixed_height_downsampled_url":"http:\/\/media1.giphy.com\/media\/RBLigAVE0xJte\/200_d.gif","fixed_height_downsampled_width":"272","fixed_height_downsampled_height":"200","fixed_width_downsampled_url":"http:\/\/media1.giphy.com\/media\/RBLigAVE0xJte\/200w_d.gif","fixed_width_downsampled_width":"200","fixed_width_downsampled_height":"147","fixed_height_small_url":"http:\/\/media1.giphy.com\/media\/RBLigAVE0xJte\/100.gif","fixed_height_small_still_url":"http:\/\/media1.giphy.com\/media\/RBLigAVE0xJte\/100_s.gif","fixed_height_small_width":"136","fixed_height_small_height":"100","fixed_width_small_url":"http:\/\/media1.giphy.com\/media\/RBLigAVE0xJte\/100w.gif","fixed_width_small_still_url":"http:\/\/media1.giphy.com\/media\/RBLigAVE0xJte\/100w_s.gif","fixed_width_small_width":"100","fixed_width_small_height":"73","username":"","caption":""},"meta":{"status":200,"msg":"OK"}}
```

We got some stuff! It's not very readable, but if we paste it into a [JSON Prettifyier](http://jsonprettyprint.com), we can see the structure very clearly:

```nohighlight
{
  "data": {
    "type": "gif",
    "id": "RBLigAVE0xJte",
    "url": "http:\/\/giphy.com\/gifs\/food-dessert-etc-RBLigAVE0xJte",
    "image_original_url": "http:\/\/media1.giphy.com\/media\/RBLigAVE0xJte\/giphy.gif",
    "image_url": "http:\/\/media1.giphy.com\/media\/RBLigAVE0xJte\/giphy.gif",
    "image_mp4_url": "http:\/\/media1.giphy.com\/media\/RBLigAVE0xJte\/giphy.mp4",
    "image_frames": "35",
    "image_width": "245",
    "image_height": "180",
    ...
  },
  "meta": {
    "status": 200,
    "msg": "OK"
  }
}
```

That's a lot of data for just one GIF. For our purposes, the only thing we care about is the value of the `"image_url"` key, which is inside of the `"data"` key:

```nohighlight
http:\/\/media3.giphy.com\/media\/RBLigAVE0xJte\/giphy.gif
```

If we fix up the "escaped" forward slashes, we get a valid url to a GIF!

```nohighlight
http://media3.giphy.com/media/RBLigAVE0xJte/giphy.gif
```

And then, if we set this url as the `"src"` attribute of an `<img>` tag in HTML, like so:

```html
<img src="http://media1.giphy.com/media/RBLigAVE0xJte/giphy.gif"/>
```

the result will be a GIF on our page!

<img src="http://media3.giphy.com/media/RBLigAVE0xJte/giphy.gif"/>

#### Adding Params: tag

But artisan lollipops is a topic for another time, because today is all about the Jacksons. To specify the topic of interest, we simply need to add another data parameter with the key "tag" and the value "jackson+5":

```nohighlight
$ curl -G https://api.giphy.com/v1/gifs/random -d api_key=dc6zaTOxFJmzC -d tag=jackson+5
```

which should respond with a GIF like this one:

<img src="http://media2.giphy.com/media/BrzAAl0VMNkk/giphy.gif" />

Much better!

## The Goal

Your site should provide the user with a form where she can type a search query:

<img src="screenshots/blank.png"/>

Upon submitting the form, the user should briefly see an indication that something awesome is about to happen:

<img src="screenshots/loading.png"/>

After a little while, the user should see a GIF appear!

<img src="screenshots/dance.png"/>

If the user clicks the button again, even with the same search term, a new request should be sent, yielding a (probably) new GIF:

<img src="screenshots/dance2.png"/>

The gif should ideally be relevant to both the user's search term ("dance" in this case) and the Jackson 5 (or at least Michael), but depending on the search term, and just luck-of-the-draw, you might find that only one or the other could be satisfied. For example, a third click might yield this:

<img src="screenshots/dance3.png"/>

Dance: check. Jacksons: not so much. That's OK if the results don't quite work out, as long as you put in the effort and heart and soul (where "effort and heart and soul" means your form submitted to Giphy a request in which the "tag" key had a value of `"Jackson 5 dance"`).

Finally, it is possible that something might go wrong in the process of making the request. If so, you should report an error to the user, like so:

<img src="screenshots/error.png" />

## Obtaining the Starter Code

1. Visit our [starter-code repository][starter-repo] on Github.

2. **Fork** our repo by pressing the "Fork" button. You will now have your own version of the project, hosted on your Github profile.

3. Back on your terminal, use `cd` to navigate to the place in your file system where to the place you want this project to live.

4. Clone your remote repo:

	```nohighlight
	$ git clone https://github.com/bobthebuilder/the-ajaxson-5
	```

	where `bobthebuilder` is your own username.

[starter-repo]: https://github.com/bgschiller/the-ajaxson-5

## Take a Look

Look inside the folder:

```nohighlight
$ cd the-ajaxson-5/
$ ls
index.html request-gif.js
```
You should see only two files: `index.html` and `request-gif.js`.


#### index.html

In `index.html` we have four main things:
* a `<form>` where the user can type a search query and request a new GIF
* an `<img>` where the GIF will be displayed
* a `<p>` that we can use to report feedback about the image loading or an error having occurred.
* a couple `<script>` tags to load Vue.js and our own `request-gif.js` script.

Changes you'll need to make in `index.html`:

1. You'll add a loading indicator for when we're waiting for Giphy to hand us back a GIF.
2. Change the tag input so that it's kept in sync with your Vue's data.

#### request-gif.js

This file defines a new Vue with a few data properties (`tagValue`, `errorMessage`, `loading`, and `imgSrc`), and a single large method: `fetchGif`. You'll also notice the line that says `el: '#mount-point'`. That tells Vue to look to the html element with an id of `mount-point` in order to know how to render.

Let's take a look in the `fetchGif` function. This is where you'll do all of your work in this file.  You'll see a handful of TODOs sprinkled throughout the body of this function. The code that's in there currently provides a skeleton for the following game plan:

1. Check the Vue's data to figure out what the user typed
2. make an AJAX request
3. when the request comes back, save the `image_url`. This causes Vue to re-render the DOM, showing the GIF.

To make the AJAX request, we use the built-in `fetch()` function. Take a look at the way we're calling fetch. We're passing in a [template string](https://wesbos.com/javascript-template-strings/), using backticks. It has the params already filled out, but the URL is wrong. It's currently `http://example.com`.

There are many more settings you can configure using fetch but, for now, the URL is all we need.

 The fetch function makes an HTTP request, and produces something called a `Promise`. Promises in javascript are all over the place. They're a way of saying "This action isn't done yet but when it is, here's what I'd like to do next." That's the `.then(callback)` portion. It's also possible to say "Here's what we should do if something goes wrong". That's the `.catch(callback)` portion. We'll talk more about promises in class.

 As it happens, we have some things we'd like to do once the action is done. Specifically, we'd like to update our `imgSrc` property with the new data, and set `loading` to be false. You'll see a couple TODO comments for those items.

The last section of this function is a TODO where we instruct you to give the user a "Loading..." message while they wait for the response to come back. You might be wondering: Why are we displaying a loading message AFTER we've already done the whole request and handled the response in the `then(...)` (or `catch(...)`) function? Remember, those callback functions will not actually be *executed* until later, after the response comes back. Just because the `then(...)` function is defined *above* line 41 does not mean that it will actually be invoked before line 41.

## Your Task

The assignment has 3 parts:

#### Part 1: Core Functionality

This part is easy to explain: go ahead and fill in those `TODO` comments!

You know you are done when your site performs all the functionality described in the [The Goal](./#the-goal) section above.

You may find the following Vue Documentation pages helpful:

* [Declarative Rendering](https://vuejs.org/v2/guide/#Declarative-Rendering)
* [Conditionals and Loops](https://vuejs.org/v2/guide/#Conditionals-and-Loops)
* [Handling User Input](https://vuejs.org/v2/guide/#Handling-User-Input)

#### Part 2: Validation

Next, it is time to add some validation to the page. We are going to create a contrived scenario here, just to make you practice your skillz.

You know those "Prove you are not a robot" CAPTCHA widgets? Your  job is to create one of those, albeit a nontraditional one. Specifically,  the task is as follows:

Add a second field to your form. It should look like this:

![captcha](./screenshots/captcha.png)

If the user gets the answer right, then go ahead and load the GIF like normal.

If not, bring the hammer down:

![rejected](./screenshots/rejected.png)

Of course, this would actually be a terribly ineffective way to prevent robots from searching for GIFs on your site, so don't go getting any ideas that this is something you should actually do In Real Life.


#### Part 3: Beauty

Finally, add some CSS to make your page beautiful and responsive.

Here is an example of a nicer looking page:

![pretty-page-1](./screenshots/pretty-page-1.png)
![pretty-page-2](./screenshots/pretty-page-2.png)
![pretty-page-3](./screenshots/pretty-page-3.png)

But feel free to get creative and style the page however you want.

The one requirement is that your page **must be reasonably responsive!**. Specifically, this means that it must be equally functional and non-ugly on mobile phones as it is on larger desktop screens. You should make your browser window really narrow while you design, so as to force yourself to think "mobile first".

You are free to use [Bootstrap](http://getbootstrap.com/css/) or any other CSS framework to help in this. I also recommend flexbox for making your page responsive.

## How to Submit

1. Follow the [submission instructions](..)
2. Hit the dance floor!
