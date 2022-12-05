# BenchBnB, Phases 5-6

[Live Demo]

In Phases 5, you will create a "New Bench" form and then refactor it with custom
hooks and components to DRY up your code. Phase 6 then consists of implementing
Reviews.

[Live Demo]: https://aa-bench-bnb.herokuapp.com/

## Phase 5: Create a "New Bench" page

Create a new directory in __components__ called __BenchFormPage__ containing an
__index.js__ and a __BenchFormPage.css__. Inside the __index.js__, create a
functional `BenchForm` component and export it as the default. Then go to
__App.js__ and add a `Route` in `App` rendering `BenchFormPage` at
`/benches/new`.

The `BenchForm` component should create state values and render form inputs for
`title` (text), `price` (number), `description` (text area), and `seating`
(number). There should also be an `errors` state that defaults to an empty
array.

This page will be opened by clicking a spot on a map to create a new bench. The
latitude and longitude of this spot will be passed in as query string
parameters. These can be accessed via `location.search`, where `location` is the
object returned by React Router's `useLocation`. There should be disabled inputs
for `lat` and `lng` with these values.

Add a submit handler which passes an object containing the input values stored
in state to `createBench` and `dispatch`es the resulting thunk action. A
successful response should trigger a redirect to `/`. If there is an error
response, the error messages should be rendered in a `ul` with distinctive
styling (e.g., red text).

If there is no current user, redirect to the login page.

Once you have your form working, go to __App.js__ and add a `Route` in `App`
rendering `BenchFormPage` at `/benches/new`.

[`FormData`]: https://developer.mozilla.org/en-US/docs/Web/API/FormData/FormData

### Extracting Form Hook and Components

At this point, you have a good opportunity to make the form code across the
login, signup, and new bench components more DRY. Consider the following
elements that all of your forms share:

- Change handlers on all inputs have the form `e => setValue(e.target.value)`
- All forms have a `submit` handler that
  - calls `preventDefault`
  - dispatches a thunk action that makes an API request
  - populates an `errors` state with error messages when a request is invalid
- In terms of rendering,
  - all inputs always have associated labels
  - all forms render a `ul` for errors

These common elements can all be extracted to DRY up your code.

Begin by writing a custom `useInput` hook for the change handlers. Create a
__src/hooks__ folder containing an __index.js__ file. Inside the file, export a
`useInput` function that takes in an `initialValue` parameter. The hook should
create a state value for an input--initialized to `initialValue`--and return the
current state value along with an `e => setValue(e.target.value)` change
handler. (How can your hook return more than one thing? **Hint:** How do regular
hooks return more than thing?)

If you have set up your `useInput` hook correctly, then you should be able to do
something like this:

```js
const [title, onTitleChange] = useInput('');

// ...

<input type="text" value={title} onChange={onTitleChange}>
```

Next you're going to extract the submit handler functionality. Back in
__hooks/index.js__, export a `useSubmit` function that takes two callbacks: an
action creator and a success callback. (Pass these callbacks as part of an
options object so that you can easily add other parameters later.) `useSubmit`
should then create and return `errors` state and an `onSubmit` callback that

- calls `preventDefault`
- dispatches the thunk action returned by the provided callback
- sets errors in the case of an error response
- calls the success callback if no errors

(That looks like a lot to implement, but remember that you are simply
**extracting common code**; you shouldn't need to figure out how to do all of
that again. You just need to figure out how to move the code that already does
those things to its new location so that the functionality remains the same.)

Again, if you have set up your `useSubmit` hook successfully, then you should be
able to do something like this:

```js
const [errors, onSubmit] = useSubmit({
  createAction: () => {
    const bench = {
      // ...
    };

    return createBench(bench);
  },
  onSuccess: () => history.push('/')
});

// ...

return (
  <form onSubmit={onSubmit}>
    {/* ... */}
  </form>
);
```

Finally, extract `Input`, `TextArea`, and `FormErrors` components to clean up
your jsx even more. To do this, create a __Forms__ folder in __src/components__
and add an __index.js__ file inside. Create and export a functional component
for each of these three elements. The `Input` and `TextArea` components should
receive as props an object consisting of the usually assigned attributes plus a
`label` and return a labeled `input` or `textarea` element.

As an example, your `Input` component could look like this:

```js
export function Input({ label, ...inputProps }) {
  return (
    <label className="input">
      {label}
      <input {...inputProps} />
    </label>
  );
}
```

> **Tip:** Since most of your inputs will be of type `text`, make that a default
> value for `type`.

`FormErrors` should take in `errors` and map them into an unordered list.

When you finish, you should be able to construct your form elements like this:

```js
<form onSubmit={onSubmit}>
  <FormErrors errors={errors}/>

  <Input 
    label="Title"
    value={title}
    onChange={onTitleChange}
    required
  />
  {/* ... */}
</form>
```

Much cleaner! Go ahead and refactor your new bench form to take advantage of
these new hooks and components.

Conclude this phase by refactoring your `LoginForm` and `SignupForm` to use the
new hooks and components too. Your `SignUpForm` has an additional check that
`password === confirmPassword`; incorporate this check into a new callback and
pass it as an additional argument to `useSubmit` under the key of `validate`.
(Make sure to call `validate` at an appropriate point in the hook if it is
present!) Also add an `action` key to `useSubmit`'s object parameter that will
allow you to pass in an actual action instead of an action creator. Adjust your
code to handle this situation as well. Have `LoginForm` and `SignupForm` pass in
an already-created action instead of an action creator.

Great job! You know what you need to do now: **Commit your code!**

## Phase 6: Reviews

Following a process similar to the one used for `Bench`, create a `Review`. You
will need a model and table, controller and routes, Jbuilder views, thunk action
creators, and a reducer. More precise specs are listed below, along with a few
tips and hints. Most of the implementation details, however, are left for you to
determine.

### Desired behavior

Reviews should be rendered on the `BenchShowPage`. Each review should show the
author's `username`, the `rating`, and the `body`. Unless the current user has
already written a review for the current bench, show a button to leave a review.
Clicking the review button should toggle a state boolean value that causes a
review form to be rendered on the `BenchShowPage` as well. (Don't forget to use
your new hooks and components when constructing your form!) Finally, users
should be able to delete (only) reviews that they have written. (Consider using
a [Font Awesome icon] for your delete button.)

Do not worry about editing reviews.

### Model and table

A `Review` should include a `rating` that goes from 1 to 5 and a `body` that
allows for lengthy comments. Think about what other columns your `Review` should
have. **Hint:** You'll need some `references`.

Add indexes--you should only need 2!--validations, and associations (with
appropriate options). You should ensure at both the database level and the
model-validation level that a user cannot leave more than one review for a
bench. While you could do the model check with a scoped uniqueness validation,
try writing a custom `not_a_duplicate` [validation] to check it instead. (It
will be good practice.) Remember to include an apt error message on failure!

### Controller and routes

With regard to actions and routes, you only need to define `create` and
`destroy`. (Why do you not need `show` or `index`?)

If you have trouble with `create`, look back at your `BenchesController`. You
should also check your parameters in the server log to see what is being
passed in. (You might find [`wrap_parameters`] a helpful method to employ.)
Make sure that a user can only create reviews for which they are the author.

Similarly, you should ensure that users can only `destroy` reviews that they
have written. Have `destroy` return the destroyed review.

### Jbuilder views

You didn't define a `show` route, but you will want to define a `show` template.
(Why?) Think about what information you want to send back and what key(s) you
want to nest it under.

You will also likely want to adjust some of your previously defined Jbuilder
views so that they return information on reviews. (Which ones?)

Make sure to avoid any N+1 queries! (**Hint:** See [here][includes].)
  
### Thunk action creators and reducers

Think about what actions you will need for processing your reviews. You of
course want to update the store in response to information returned from the two
review routes. Remember, though, that you will also be sending up review
information from other routes. What type(s) of action do you need to process
review information returned from those other routes?

In your reducer, recall that you want to create a normalized state. What would
that look like for a review? Take a minute to sketch out how you want the review
slice of state structured, then test your implementation against your sketch.
(Examine the bench slice of state in the browser if you need help figuring out
what a normalized slice of state should look like.)

If you can explain what the following line of code does, feel free to use it
in an appropriate place:

```js
const { [review.id]: _remove, ...newState } = state;
```

Finally, if you changed what any of your non-review Jbuilder views return,
adjust the corresponding thunk action creators and/or reducers to make sure you
store the additional review information from the backend.

Once you get the reviews up and running, **commit your code.**

Congratulations, you've finished Part 1!

[`wrap_parameters`]: https://api.rubyonrails.org/v7.0.3/classes/ActionController/ParamsWrapper.html
[validation]: https://guides.rubyonrails.org/active_record_validations.html#custom-methods
[includes]: https://guides.rubyonrails.org/active_record_querying.html#includes
[Font Awesome icon]: https://fontawesome.com/icons?d=gallery&m=free