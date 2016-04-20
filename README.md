# NodeJS command-line Reddit clone

In this project, we will be using the not-so-secret Reddit JSON API to create a tiny command-line version of Reddit.

## What is an API?
First, let's remind ourselves what an API is: a set of functions that make it easier to build applications. APIs can present themselves in different forms. We have already seen quite a few of them:

### Built-in APIs
JavaScript has a mathematics API. All the functions of this API are located under the `Math` global object. The functions of this API can be accessed by calling them as properties of the global `Math` object. Each function has unique inputs and a unique output. Together, they characterize that function. For example, the `Math.sqrt` function takes a number as input, and returns its square root as output.

### Web APIs
[The web is full of APIs](http://www.programmableweb.com/apis/directory). As an example, the [Big Huge Thesaurus](https://words.bighugelabs.com/api.php) has an API that consists of one function. The function is located under the `http://words.bighugelabs.com/api/{version}/{api key}/{word}/{format}` URL. It can be accessed by making an HTTP request to the `words.bighugelabs.com` server. The function returns its output as an HTTP response (text) with a choice of different representation formats (JSON, XML, ...). The Reddit JSON API which we will be using for this project is another example of web API.

### NodeJS APIs
We can create our own NodeJS APIs by simply writing one or more functions. To package them, we can put them in a JavaScript file. We can use the `module.exports` global to determine which functions will be part of our API. For example, we created a fortune telling API that has only one function called `getFortune`. The module of this API contained a few other things: an array of fortunes, maybe even a random number generator. But we chose to expose only the `getFortune` function, which is what our API consists of. The rest is **encapsulated** in our module and cannot/should not be used by the outside world.

## Getting acquainted with the Reddit JSON API
If you haven't before, get to https://www.reddit.com/ and browse for a bit. When you are done looking at the cat photos, I'd like you to notice the following points:

1. The Reddit site, like most other sites can be decomposed in two strictly non-overlapping parts: the *data* and the *presentation*.
2. The **data** on the main Reddit page consists of a list of links. Each link is associated to the user who posted it, the number of votes, the created date and so on.
3. The **presentation** consists of mostly everything else: the fonts, colors, spacing, layout and so on.

Now, I'd like you to load the following URL: https://www.reddit.com/.json. Notice that there are only five more characters than the previous URL. We simply added `.json` at the end.

If you are seeing an unintelligible pile of letters and symbols, you may need to install the [JSONView Plugin](https://chrome.google.com/webstore/detail/jsonview/chklaanhfefbnpoihckbnefhakgolnmc?hl=en).

This my friends is the Reddit JSON API. Any page you are looking at on the Reddit site, you simply add `.json` to it and suddenly you **only get the data part**. Unfortunately not all web APIs are *that* nice, but in essence they work the same way. A URL is linked to a function, and its content is the "return value" of that function.

## The project
For this project, we are going to build a command-line Reddit browser. Our application will make HTTP requests to the Reddit API and output the data in as nice a way as possible in the confines of the command-line. You'd be surprised how far we can get. There's even an [NPM module to display images using colored text on the command-line](https://www.npmjs.com/package/image-to-ascii)!

### Baby steps

#### A survey of Reddit API URLs
All the Reddit API functions we are interested in return "listings". A "listing" is an object with a `data` property which is itself an object. The `data` object contains an array of items called `children`, and properties called `before` and `after`. The `before` and `after` properties can be used for navigating the prev/next pages on the Reddit site.

Here are a few examples of Reddit API URLs and the data that they would return. If in doubt, remember you can always remove the `.json` part to look at the actual web page. For each URL, go to it in your browser and identify all the elements we are interested in:

* https://www.reddit.com/.json

  This is the Reddit homepage data. It lists the posts that appear on reddit.com when a user is not logged in. By default, the posts are listed in order of "hot", so that the page can stay relevant.

* https://www.reddit.com/.json?after=XXXX

  This is the "next page" of the reddit homepage. The XXXX part would be replaced with whatever is the value of `after` in the result you get from the previous page.

* https://www.reddit.com/controversial.json

  This is also the Reddit homepage, but with posts listed in order of their controversy score. It's another way at looking at all the posts on Reddit.

* https://www.reddit.com/subreddits.json

  This is a listing of popular subreddits.

* https://www.reddit.com/r/SUBREDDIT.json

  This is a listing of all the posts for the subreddit called SUBREDDIT, again in order of "hotness", the default. For example, if you want to see all the Montreal posts in JSON format ordered by hotness, you would go to https://www.reddit.com/r/montreal.json

* https://www.reddit.com/r/SUBREDDIT/comments/POST_ID/POST_TITLE.json

  This is a listing of the top comments for a certain Reddit posting. When looking at a listing of posts, the `permalink` property of a post object will give you the URL of the comments link.

  In contrast to the other listings, comments are a beast! Instead of getting back a regular list, comments are nested. This is because a comment can come as a reply to another comment, and we want to be able to display this fact perhaps by nudging the text to the right by a bit.

Using the NPM `request` module and the built-in `JSON.parse` function, create a module called `reddit.js` and expose the following functions:

```javascript
/*
This function should "return" the default homepage posts as an array of objects
*/
function getHomepage(callback) {
  // Load reddit.com/.json and call back with the array of posts
}

/*
This function should "return" the default homepage posts as an array of objects.
In contrast to the `getHomepage` function, this one accepts a `sortingMethod` parameter.
*/
function getSortedHomepage(sortingMethod, callback) {
  // Load reddit.com/{sortingMethod}.json and call back with the array of posts
  // Check if the sorting method is valid based on the various Reddit sorting methods
}

/*
This function should "return" the posts on the front page of a subreddit as an array of objects.
*/
function getSubreddit(subreddit, callback) {
  // Load reddit.com/r/{subreddit}.json and call back with the array of posts
}

/*
This function should "return" the posts on the front page of a subreddit as an array of objects.
In contrast to the `getSubreddit` function, this one accepts a `sortingMethod` parameter.
*/
function getSortedSubreddit(subreddit, sortingMethod, callback) {
  // Load reddit.com/r/{subreddit}/{sortingMethod}.json and call back with the array of posts
  // Check if the sorting method is valid based on the various Reddit sorting methods
}

/*
This function should "return" all the popular subreddits
*/
function getSubreddits(callback) {
  // Load reddit.com/subreddits.json and call back with an array of subreddits
}
```

#### The browsing interface
Since we are in command-line mode, we will be using text and the keyboard as our main sources of interactivity.

The [Inquirer.js](https://github.com/sboudrias/Inquirer.js) module is pretty good for that. It can let us display a list of things, and give the user a choice of what to do, like a menu.

Before starting the project, it would pay off to start getting familiar with the Inquirer module. In a new workspace, use NPM to install inquirer. Then run this example:

```javascript
var inquirer = require('inquirer');

var menuChoices = [
  {name: 'Show homepage', value: 'HOMEPAGE'},
  {name: 'Show subreddit', value: 'SUBREDDIT'},
  {name: 'List subreddits', value: 'SUBREDDITS'}
];

inquirer.prompt({
  type: 'list',
  name: 'menu',
  message: 'What do you want to do?',
  choices: menuChoices
}).then(
  function(answers) {
    console.log(answers);
  }
);
```

Note that since prompting the user is a long-running function, we are using a callback to receive the answers. Contrary to other callbacks you have used before, here we are doing `.then(callback)`. This is because inquirer uses [JavaScript Promises](http://www.html5rocks.com/en/tutorials/es6/promises/) instead of "regular callbacks". We will not be studying the Promise at this time, so for the moment you can see this as simply passing a callback to get your answers.

### Let's do it!
Building upon the "baby steps" section, let's code the actual project. This will be done by implementing feature after feature. The different features are quite independent of each other so you can tackle them in any order you wish. Some of them are more challenging than others. They will be marked with :star:.

#### Basic feature: the main menu
An example of main menu prompt was provided to you. This should be the starting point of the application. When someone runs `node reddit.js`, this menu should be displayed. As you keep adding features, you can expand this menu by adding options to it.

#### Basic feature: the homepage posts
When the user chooses the homepage option, you should display the list of posts from the `getHomepage` function you created previously. For each post, list at least some of the info that appears on reddit: title, url, votes, username. After the list of posts is displayed, you should display the main menu again.

#### Basic feature: subreddit posts
When the user chooses the subreddit posts option, you should ask him -- again using inquirer -- which subreddit he wants to see. Then, display the list of posts in the same way as the homepage.

#### Feature: list of subreddits :star:
When the user chooses the list of subreddits option, you should load the list of subreddits using the `getSubreddits` function you created previously. Then, using inquirer, show the list of subreddits to the user. The user will be able to choose a subreddit to display its posts, or go back to the main menu. You can use an [Inquirer Separator](https://github.com/SBoudrias/Inquirer.js/#separator) to create a visual separation between the list of subreddits and the "go back to main menu" option.

#### Feature: post selection and "image" display :star::star:
When the user is shown a list of posts, instead of going back to the main menu every post should be selectable -- again using inquirer. When selecting a post, the terminal screen should be cleared and only that post should be displayed (title + url + username). In addition to this, if the URL of the post turns out to be an image -- ends in `.jpg`, `.gif` or `.png` -- you should use the [`image-to-ascii`](https://github.com/IonicaBizau/image-to-ascii) module to load the image and display it on the command line. After the post details are displayed, you should show the main menu again.

#### Feature: comments listing :star::star::star:
When the user is shown a list of posts, instead of going back to the main menu every post should be selectable -- again using inquirer. When selecting a post, the user should be shown a listing of comments for that post. Since comments are threaded -- replies to replies to replies ... -- we would like to **indent** each level of comments , perhaps by two or three spaces. To do this properly, we can make use of the [`word-wrap` NPM module]. After displaying the list of threaded comments, display the main menu again.

One of the difficulties of implementing this feature is to *properly iterate through the comments and their replies*. To do this, you will first have to analyze the way the comment listing is presented in the JSON.
