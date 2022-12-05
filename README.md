# BenchBnB: Part 1

[Live Demo]

In this two-day project, you will be building BenchBnB, a full stack application
inspired by AirBnB. Instead of vacation rentals, though, your app will show off
all the hottest bench locations in cities around the world!

You will build multiple CRUD features and integrate maps and photos, using the
[Google Maps Javascript API][maps-api] and [Active Storage][active-storage],
respectively. You will be building off of your basic Authenticate Me
application, re-organizing and refactoring as necessary.

To prepare you for your full stack project, this project is intentionally
challenging. The instructions will generally describe end goals and requirements
rather than detailed implementation steps, especially in the later phases. This
means there won't always be one right way to solve things.

This makes it especially important to regularly and thoroughly test your code,
not only to make sure you're meeting any described requirements, but also to
make sure your own code is doing exactly what you think it's doing.

That said, don't hesitate to ask an instructor for guidance if you're ever
unsure of your approach or of where to start. You're not alone in this!

Good luck!

[Live Demo]: https://aa-bench-bnb.herokuapp.com/

## Phase 0: Setup

If you haven't already, complete Authenticate Me, Parts 1 and 2 (Backend and
Frontend). Test that you are able to sign up, log in, and log out without
causing any errors in your browser console. Also test that a user's session
persists after refreshing and that appropriate errors display on invalid login
and sign up attempts.

### Reviewing Authenticate Me

Next, take as much time as you need reviewing and tinkering with your code until
you feel comfortable with how each part of your application is working and how
the different parts are talking to each other. Use `console.log`s and
`debugger`s to track execution order and data flow in your app.

Here's an example of how you can test your understanding of the login process.
What follows is a list of places to put `debugger`s, along with questions to
ask yourself whenever you hit these debuggers. Before proceeding, make sure to
log out if you've already logged in. After placing the `debugger`s, predict the
order that you'll hit them after refreshing the page and then logging in.

1. (Frontend) Entry file (__frontend/src/index.js__), right before rendering
   your application
   - Does the Redux `store` exist? If so, what's in it and why?
2. (Frontend) `login` thunk action, at the beginning and after parsing the
   response body as JSON
   - What are the `credential` and `password` arguments? Where did they come
     from? What does this change about the request you will make to your
     backend?
   - After parsing the server response as JSON in your `login` thunk action,
     what does the response data look like? Why?
3. (Frontend) session reducer, in the `case` for `SET_CURRENT_USER`
   - What is the value of the `action` argument? Where did this action come
     from? Why is it structured the way it is?
4. (Frontend) `Navigation` component, right before returning
   - What is the value of `sessionUser`? Why? What does this mean for what it
     will render?
5. (Frontend) `ProfileButton` component, right before returning
6. (Frontend) `LoginFormModal` component, right before returning
   - What is the value of `showModal`? Why? What does this mean for what it will
     render?
7. (Frontend) `LoginForm` component, in `handleSubmit`
8. (Backend) `Api::SessionsController#create` action, at the beginning
   - What are the `params` when you hit your backend's `sessions#create` action?
     Where did the data inside it come from?
9. (Backend) `users/show` Jbuilder view

Feel free to replicate this process for any other parts of your application.
Once you feel comfortable with how Authenticate Me works, create a copy of it.
You will begin by renaming your application.

### Renaming the Backend

Head to __config/application.rb__ and change the name of the module from
`AuthenticateMe` to `BenchBnb`. In this same file, change the name of your
session key to `_bench_bnb_session`.

```rb
# config/application.rb

# ...

module BenchBnb
  class Application < Rails::Application
    # ...
    config.middleware.use ActionDispatch::Session::CookieStore, 
      key: '_bench_bnb_session',
      same_site: :lax, 
      secure: Rails.env.production?
      # ...
  end
end
```

Then, head over to __config/database.yml__. Change each occurrence of the name
`authenticate_me` to `bench_bnb`, matching case as necessary. (One such
occurrence is within a comment; change this as well to prevent future
confusion).

```yml
# config/database.yml

# ... 

development:
  <<: *default
  database: bench_bnb_development

  # The specified database role ...
  #username: bench_bnb

  # ...

test:
  <<: *default
  database: bench_bnb_test

# ...

production:
  <<: *default
  url: <%= ENV['DATABASE_URL'] %>
```

> Note: If you didn't get to the Authenticate Me deployment instructions (Part
> 3), `production` likely has keys of `database`, `username`, and `password`
> instead of `url`, with each key having a value involving `authenticate_me`.
> Simply replace those three keys with the single `url` key-value pair shown
> above.

If you have a __render.yaml__ file--added in the deployment
instructions--replace the four instances of `authenticate_me` with `bench_bnb`.
(If you do not have this file, don't worry about it.)

That's it! Go ahead and run `rails db:setup` to set up a new database for
BenchBnB.

### Renaming the Frontend

First, open the __package.json__ in your root directory. Change the value at the
key of `"name"` to `bench_bnb` and the `"description"` to something appropriate.

```json
// package.json

{
  "name": "bench_bnb",
  // ...
  "description": "Everyone's favorite app for finding and rating benches!",
  // ...
}
```

Run `npm install`.

Then head to __frontend/package.json__ and change the value at the key of
`"name"` to `bench_bnb_frontend`.

```json
// frontend/package.json

{
  "name": "bench_bnb_frontend",
  // ...
}
```

Run `npm install` in the __frontend__ directory.

Next, head to __frontend/public/index.html__ and change the title of your
document to `BenchBnB`:

```html
<!-- frontend/public/index.html -->

<!-- ... -->
<head>
  <!-- ... -->
  <title>BenchBnB</title>
  <!-- ... -->
</head>
<!-- ... -->
```

Finally, head to __frontend/public/manifest.json__ and change the `"short_name"`
and `"name"` keys:

```json
// frontend/public/manifest.json

{
  "short_name": "BenchBnB",
  "name": "BenchBnB App Academy Project",
  // ...
}
```

That's it--your application is renamed!

**Now is a good time to commit your code.**

### Set up your API key in an environment variable

For this project, you will need a Google Maps API key in order for the maps
feature to work. You have hopefully already set up your Google account and
generated an API key. If not, go back and complete the "Set Up Maps API Key"
homework before proceeding.

Now that you have set up your API key, let's add it to your BenchBnB project!

__Quick Note__: It's important to take this upcoming section very seriously!
There have been multiple anecdotes from students who accidentally pushed API
keys to a public Github repo. Those keys were then scraped by bad actors who
racked up tens of thousands of dollars in charges using those keys.

Any time you add an API key to your project, you want to take precautions to
prevent bad actors from stealing and misusing your keys. In the case of your
Maps API key, because billing is calculated per map load, bad actors might want
to use your key to load maps on their own projects.

Because the Maps API key is loaded in your frontend HTML, there's actually not
much you can do to prevent the key from being publicly accessible if your web
app is deployed to production. If you decide you want to deploy your BenchBnB
project (for example, using Heroku), then **it's highly recommended that you
protect your publicly-accessible API key by restricting the websites that can
use your key.**

![How to restrict API key to specific websites][maps-api-5]

For now though, since you're planning on only working on this in a local
development capacity, you don't have to worry too much about that.

You should, however, take the necessary precautions to keep your API key from
being publicly exposed in a GitHub repository. To prevent your API key from
being pushed to GitHub while still being able to conveniently use the key
locally, store it in an environment variable.

First, go to your [Google console] and select your `benchbnb` project. In the
left-hand dropdown menu, select `APIs & Services`, then `Credentials`. Click the
`SHOW KEY` option on your BenchBnB Maps API row and copy the key.

Next, create an __env.development.local__ file in your __frontend__ folder.
Insert the following line into the file:

```plaintext
REACT_APP_MAPS_API_KEY=<your-copied-api-key-here>
```

You can name the environment variable anything you want as long as it starts
with `REACT_APP`. (If the variable name starts with anything else, React will
simply ignore it. React adopts this behavior to protect against inadvertently
exposing hidden values in your frontend's build.) Assuming you use the name
above, your frontend code can now access your API key using
`process.env.REACT_APP_MAPS_API_KEY`.

Confirm that __env.development.local__ appears in the __.gitignore__. Run `git
add .` and make sure __env.development.local__ does not get staged. If you
accidentally commit it, remember you can delete it from the repo while
maintaining your local copy with `git rm --cached .env.development.local`.

Once everything looks good, go ahead and **commit your code.** Then on to Phase
1!

[maps-api]: https://developers.google.com/maps/documentation/javascript/overview
[active-storage]: https://guides.rubyonrails.org/active_storage_overview.html
[maps-api-5]: https://assets.aaonline.io/fullstack/react/projects/bench_bnb/maps_api_5.png
[Google console]: https://console.cloud.google.com/