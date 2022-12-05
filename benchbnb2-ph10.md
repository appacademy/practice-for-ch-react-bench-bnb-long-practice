# BenchBnB, Phase 10: Enhancing The User Experience

[Live Demo]

In this final Phase, you will enhance the user experience by using a modal for
log in / sign up to preserve context and by highlighting a bench list item when
a user hovers over its marker.

[Live Demo]: https://aa-bench-bnb.herokuapp.com/

## Require current user with modal

Rather than redirecting to the sign up page when there isn't a current user,
(for instance on the `BenchFormPage`), it would be nice if there was a session
modal that just overlaid the page. Then, after signing up or logging in, the
modal could close and the user's context would not be lost.

To create reusable session components, create a __components/SessionForms__
directory. Move the `LoginFormModal` component to __SessionForms/index.js__. It
should now render only a Modal: the containing button and the logic around
toggling its visibility was hardcoded for the `Navigation` component context.
Move this logic to the `Navigation` component.

Move the `LoginForm` component file to this directory as well. Extract the
signup form itself from the `SignupFormPage` component and save it to a
__SignupForm.js__ file in `SessionForms`.

Create a generic `SessionModal` in __SessionForms/index.js__. This should render
either the `LoginForm` or the `SignupForm`, with a button at the bottom that
toggles between the two.

Render a `SessionModal` on the `BenchFormPage` if there is no current user.

## Highlight bench list items when hovering over markers

The final feature that you will add to your BenchBnB app--at least for now!--is
to highlight a bench list item when a user hovers over its marker.

First, add a `highlightedBench` state value to the `BenchIndexPage`. Then add
handlers to `markerEventHandlers` for `mouseover` (set `highlightedBench` to
marker's associated bench) and `mouseout` (clear `highlightedBench`).

Pass `highlightedBench` to `BenchMap` as a prop. Create a different marker style
for highlighted benches (e.g., inverted colors). Add another effect to
`BenchMap`: whenever the `highlightedBench` changes, ensure every marker whose
bench is not the `highlightedBench` has standard styling, and any marker whose
bench is the `highlightedBench` has the custom styling. (You can use `setLabel`
and `setIcon`.)

Pass `highlightedBench` to the `BenchList` as well. Add custom styling to
whichever `BenchListItem` corresponds to the `highlightedBench`.

Finally, pass `setHighlightedBench` to `BenchList`. On hovering over a
`BenchListItem`, set `highlightedBench` to that bench.

Congratulations! You finished! Give yourself a huge pat on the back and
celebrate by **committing your code!**

## Bonus

Congratulations again on making it to the end!

This is a large project, and you probably need a well-deserved break from
BenchBnB at this point. But if you ever feel the urge to revisit it and tinker
some more, here are a few ideas for further development:

1. Style and structure the site as closely to AirBnB as you can

2. Add multiple photos per bench; display them in an image carousel

3. Allow users to mark favorite benches

4. Track users' bench visits