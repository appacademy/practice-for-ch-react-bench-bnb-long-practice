# BenchBnB, Phases 3-4

[Live Demo]

It's time to build (Phase 3) and style (Phase 4) components that render your
bench data!

[Live Demo]: https://aa-bench-bnb.herokuapp.com/

## Phase 3: Bench `index` and `show` pages

Create new directories __BenchIndexPage/__ and __BenchShowPage/__ in
__frontend/src/components/__. Inside each, create an __index.js__.

### Bench `index` page

Within __frontend/src/components/BenchIndexPage/index.js__, define and `export
default` a `BenchIndexPage` component. For now, simply return a `div` with the
text `Bench Index Page`.

Before filling out `BenchIndexPage`, let's connect it with the rest of your app.

Head to __frontend/src/components/App.js__ and import the `BenchIndexPage`
component. Since BenchBnB will not have a separate splash page as its home page,
you'll use `BenchIndexPage` as your home page.

Add a new `Route` component with a `path` of `/` and an `exact` prop, within
which you should render the `BenchIndexPage` component.

Refresh your browser. Test that you can see the `BenchIndexPage` component at
the root path of `/`, but not at any other path. Nice!

Head back to __frontend/src/components/BenchIndexPage/index.js__. Eventually,
you will render a list of benches, a control panel for filtering the list, and a
map showing each bench's location. For this phase, you will only render the list
of benches.

Select every bench object from the `benches` slice of Redux state with
[`useSelector`] and put them into an array called `benches`.

In order to ensure your Redux state is populated with bench data, add a
`useEffect` from which you should `dispatch` the `fetchBenches` thunk action.

For rendering the bench list, define two new components, `BenchList` and
`BenchListItem`, each within their own file inside
__frontend/src/components/BenchIndexPage/__. Your __BenchIndexPage/__ directory
should now look like so:

```text
BenchIndexPage
  ├── BenchList.js
  ├── BenchListItem.js
  └── index.js
```

Within __index.js__, import the `BenchList` component and render it, passing in
the `benches` array selected from your Redux state as a prop of the same name.

Within the `BenchList` component, import `BenchListItem`. Within a root element,
render a `BenchListItem` for each bench object in the `benches` array, passing
in the bench object as a prop named `bench`. Don't forget to add a unique `key`
prop to each `BenchListItem`! Add a heading with the text `Benches` to the top
of the `BenchList` root element.

Within `BenchListItem`, render a `div`, within which you should render the
`title` and `price` of the bench, using any HTML structure you see fit to use
(e.g., the `title` might be in a heading element).

Clicking the outer `div` should navigate to the bench show page:
`/benches/[BENCH_ID]`. Use the `push` method on the [`history`] object
returned by [`useHistory`] to imperatively navigate to this frontend route.

Let's review what you've implemented thus far for the `BenchIndexPage`:

- `BenchIndexPage`:
  - fetches bench data from the backend in a `useEffect` function, then selects
    that bench data from the Redux state after a `useSelector`-triggered
    re-render
  - renders a `BenchList` passing in the `benches` array as a prop
- `BenchList` renders a `BenchListItem` for each bench in `benches`
- `BenchListItem` renders a `div` displaying data from the `bench` provided in
  prop; clicking this `div` navigates to the bench show page

Refresh your browser again to test that your benches are successfully rendering.
Nice!

### Bench `show` page

Within __frontend/src/components/BenchShowPage/index.js__, define and `export
default` a `BenchShowPage` component. Return a `div` with the text `Bench Show
Page`.

Head to __frontend/src/components/App.js__, import `BenchShowPage`, and render
it within a new `Route` with a path of `/benches/:benchId`.

Once again, refresh your browser and test that your `BenchShowPage` component is
rendering only at paths that match `/benches/:benchId`--e.g., `/benches/1` or
`/benches/2`, but not `/` or `/benches`.

Head back to __frontend/src/components/BenchShowPage/index.js__.

Grab the `benchId` param from the URL with [`useParams`], then retrieve the
bench object having the same `id` from your Redux state. In a `useEffect`
function, fetch that batch.

Render `null` if there was no `bench` with `id` of `benchId` in your Redux
state. Otherwise if `bench` does exist, render a `div` containing the following
two elements:

- a header with the `bench`'s title and a [`Link`] back to the bench index page
  (`/`)
- a details section with a heading of `Details`, a paragraph containing the
  `bench`'s `description`, and finally a `ul` displaying the `bench`'s `Seats`,
  `Latitude`, and `Longitude`.

Refresh your browser. Test that clicking on a particular `BenchListItem` on your
bench index page brings you to that bench's show page. Confirm that everything
is rendering as expected on the bench show page. Well done!

## Phase 4: Styling

It's time to style the components you just created! The design is up to you. You
can use the [real AirBnB][airbnb-sf] as inspiration as you build out BenchBnb,
but you are not required to. Focus on layout, spacing, and sizing over small
visual flourishes.

### Add a CSS reset file

If you haven't already, create a CSS reset file: __frontend/src/reset.css__. As
a reminder, CSS reset files override most default browser styles (known as `User
Agent` styles) so that your project starts as a blank slate.

Feel free to use [this CSS reset written by Eric Meyer][css-reset] (be sure to
include the attribution!).

### Create CSS files

- Ensure you have a way of selecting the parent or container elements of each
  section / box in your app.
  - At the very least, this usually means adding a `className` to the root
    element of each component.
  - You'll often add `className`s to nested elements in a component that contain
    distinct subsections or boxes (e.g., `ul` and `form` elements often get
    `className`s).

- Start with layout, relying primarily on [`flexbox`] or [`grid`]. Remember to
  apply `display: flex` or `display: grid` to the parent or container of the
  elements you want to lay out.  
  - `flexbox` works best when you are laying out child elements either
    vertically *or* horizontally (e.g., spacing out buttons horizontally in a
    navigation bar).
  - `grid` is a good option when you are laying out child elements in two
    dimensions (e.g., tiling cards).
  - `flexbox` is a good go-to, whereas `grid` is helpful in more niche
    circumstances.
  - Reference the excellent CSS Tricks guides for [`flexbox`][flex-guide] and
    [`grid`][grid-guide]

- Remember that putting `display: flex` on an element only affects the layout of
  its direct children. However, you can nest one `flexbox` inside another: just
  add `display: flex` to a `flex` child to make it a `flex` parent as well!
  Consider the following example:
  - You have a navigation bar with two sections: general navigation links on the
    left and account-related links on the right.
  - You might put `display: flex` with `justify-content: space-between` on the
    whole navigation bar to horizontally separate the two link sections.
  - Then, each section might get `display: flex` with `justify-content:
    space-around` to horizontally spread out the links they contain.

- Sometimes it's best to change your HTML structure to implement a particular
  layout. Don't be afraid to add wrapper or container `div`s!
  - For example, suppose two `button`s at the bottom of a `form` need to appear
    in the same row. You could create a wrapper `div` around them, and put
    `display: flex` on the wrapper.

- For recurring UI building blocks, such as buttons, headings, and links, try to
  stay DRY; the default styling for each can be defined in a single rule.
  - Be careful about relying on HTML tags for your selectors. You might want a
    particular `Link` to be styled like a `button`, even though it's actually an
    `a` tag. Instead, prefer a shared `className`.

These are not absolute rules; your own approach may differ, and that's fine if
it works well! Whatever approach you use, though, it's important to:

1. Choose an approach and stick with it. Consistency reduces decision fatigue.

2. Keep things as organized and as simple as possible.

3. Use your browser's developer tools heavily. Experiment and debug there. Once
   you know which elements need which styles, apply that to your code (adding
   `className`s as necessary).

4. Remember that styling is a game of trial and error. Work incrementally and
   don't expect anything to look right on your first try.

Go ahead and style your `BenchIndexPage` and `BenchShowPage`. If your existing
CSS (for the navigation bar and session forms) was disorganized or incomplete,
refactor/complete that as well.

Spend no more than an hour on styling for now.

**Now would be a good time to commit your repository.**

[`history`]: https://v5.reactrouter.com/web/api/history
[`useSelector`]: https://react-redux.js.org/api/hooks#useselector
[`useHistory`]: https://v5.reactrouter.com/web/api/Hooks/usehistory
[airbnb-sf]: https://www.airbnb.com/s/San-Francisco--CA/homes
[css-reset]: https://meyerweb.com/eric/tools/css/reset/
[`flexbox`]: https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Flexible_Box_Layout
[`grid`]: https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Grid_Layout
[flex-guide]: https://css-tricks.com/snippets/css/a-guide-to-flexbox/
[grid-guide]: https://css-tricks.com/snippets/css/complete-guide-grid/
[`useParams`]: https://v5.reactrouter.com/web/api/Hooks/useparams
[`Link`]: https://v5.reactrouter.com/web/api/Link