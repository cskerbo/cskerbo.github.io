---
layout: post
title:      "User Authentication - SPA, JavaScript and Rails"
date:       2020-04-26 23:18:23 +0000
permalink:  user_authentication_-_spa_javascript_and_rails
---


I recently finished my portfolio project for the Learn JavaScript section. The challenge was to build a SPA (single-page application) using a JavaScript frontend that communicated with a Rails backend.

On paper, this seemed straight-forward. I knew I wanted to create a Slack clone so I dove right in. The challenge appeared when I started to think through how to authenticate a user and still keep the application single-page.

I searched, and searched, and then searched some more. Surprisingly, there was very little documentation out in the wild on how to move forward with this. Sure, there were many tutorials and posts that seemed promising - but in the end they either broke the SPA requirement, were too dated to work in any modern Rails builds, or were far too much effort for something that *should* be simple. On top of that, no one covered how you would actually create or login a user once you had your backend setup appropriately. I knew the best approach was to utilize a JWT (JSON web token), but how did I implement that in practice?

Eventually, I realized I was going to have to dive in and figure this out on my own - so I did! Using a combination of a couple of gems I found a very quick way to add authentication to your SPA. I hope the following tutorial is able to help anyone in who finds themself in a similar position I was in.

If you want to check out my JavaScript SPA, Clack, take a look at the repo here: `https://github.com/cskerbo/Clack`

<a href="https://imgur.com/2BbJfqN"><img style="width: 500px; height: 300px;" src="https://i.imgur.com/2BbJfqN.png" title="source: imgur.com" /></a>

Now, on to the fun part!

## Authentication Setup

I'm going to assume you've already covered some of the basics, if you haven't, pause and take care of these items:

* bcrypt gem: Your user is likely going to require a password, and you'll want to be able to use Active Record's has_secure_password method.
* rack-cors gem: You need to enable cross-site requests.
* Your database, models, and associations are setup. Make sure your user fields are validated, and your relationships are set up correctly for your project needs.

Alright, let's get started.

1. Install the following 2 gems:

```
gem 'jwt'
gem 'knock'
```

2. Include Knock in your application controller:

```
class ApplicationController < ActionController::API
  include Knock::Authenticable
end

```

3. Create a user_token_controller with the action to skip verifying authenticity:

```
class UserTokenController < Knock::AuthTokenController
  skip_before_action :verify_authenticity_token
end
```

**Wait - why would you want to do this?**

In Rails 5.2, you will receive a  422 response from your server when trying to log in. Consider not including this at your own risk.

4. Add a before_action to authenticate a user. *Note: You should customize this to apply to certain controller actions based on your project. For example, the "create" action should not require authentication.*

```
class UsersController < ApplicationController
  before_action :authenticate_user
end
```

5. Set up a route to obtain a JWT, and mount the Knock Engine

```
Rails.application.routes.draw do
  post 'user_token' => 'user_token#create'
  mount Knock::Engine => '/knock'

end
```

These two lines are required for this process to work.

Your routes file should obviously have much more than this, I didn't include any of my other routes because they are likely not going to work for your project. Set yours up accordingly!

That's it - it's that easy to set up your Rails API to require a JWT when a user makes a call to it. 

**To break this down simply: We included Knock in the application controller, created a User Token Controller that's tied to Knock, created a route to obtain a JWT, and we've required authentication in the User controller.**

## Examples of User Creation, Login, and Authenticated API Calls

This is the part that took me the longest to work through, as I could not find hardly any documentation on making fetch calls in order to create or login users. 

You will need to adapt these based on your needs, but I feel these should give you a great idea on where to start.

Here is a breakdown of my application's needs, and you can always dig deeper into my source code I linked above:

* User Creation: through a form, a user is required to enter an email, username, and password
* User Login: through a form, a user is required to enter their email and password
* Upon successful login, a user receives a JWT that is then utilized to make calls to the Rails API
* For my use-case, I store the JWT in localStorage until the user logs out and localStorage is cleared.


### User Creation

```
function createUser(email, username, password) {
    fetch(`${BASE_URL}/users`,{
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
        },
        body: JSON.stringify({
            user: {email, username, password}
        })
    })
        .then(response => response.json())
        .then(responseHandler)
}
```

This is a fairly straightforward POST request to our API. The createUser function accepts 3 arguments from my user creation form on the web page, and then sends that information to the 'CREATE' action in my users controller.

*Remember - you shouldn't require authentication for this, as your will receive an unauthorized response.*

After the POST I am taking the response, converting it to JSON, and then reviewing the response. At this point, you should customize your code for your needs. In my case, I have an additional function called 'responseHandler" which checks to see if the POST was successful or if there are errors. I notify the user accordingly based on the results.

### User Login

Alright, you can create users now and you're able to POST that information to your API. But where does this JWT-business come into play?

Let's take a look at a sample login request:

```
function userLogin(email, password) {
    fetch(`${BASE_URL}/user_token`,{
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
            'Accept': 'application/json'
        },
        body: JSON.stringify({
            auth: {email, password}
        })
    })
        .then(response => response.json())
        .then(userObject => {
            localStorage.setItem('token', `${userObject.jwt}`);
        })
}
```

This function accepts 2 arguments that are retrieved from the user login form on the web page. It passes these arguments in a POST request to the user_token route we set up earlier.

Upon successful validation of the user's credentials, the response will contain a JWT. You should edit this function to handle the response as you wish - perhaps check for errors, or perhaps you want to utilize the JWT in an alternate way?

In my case, I set the JWT in localStorage. Since calls to the API from the user need to include a valid JWT, I can now retrieve that information from the user's localStorage.

### Authenticated API Calls

Here is an example of a function used to find a user by their user id:

```
function findUserById(id) {
    return fetch(`${BASE_URL}/set_user`, {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${localStorage.getItem('token')}`
        },
        body: JSON.stringify({
            user: {id}
        })
    })
}
```

Notice in particular the header line 'Authorization'. You can see here I am retrieving the user's JWT from local storage and passing that in the post request.

Any POST request you make that requires authentication should look similar to this. The idea is simple - send a POST request with a valida JWT and receive a response with the data you requested. From that point, you can do whatever you need to with the data included in the response.

That's it! If you've followed these steps correctly you should now be able to validate users with a JWT. I hope you found this guide helpful.
