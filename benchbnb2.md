# BenchBnB: Part 2

**You must finish BenchBnB Phases 1-6 before proceeding to BenchBnB Part 2. If
you have not yet finished the earlier Phases, return to the point where you
stopped and continue working from there.**

[Live Demo]

Part 2 of BenchBnB comprises Phases 7-10. In this part, you will finally create
your map (Phase 7), add features like filters (Phase 8) and photos (Phase 9),
and then polish your app with little touches to enhance the user experience
(Phase 10).

[Live Demo]: https://aa-bench-bnb.herokuapp.com/

## Phase 7: Bench map

It's time to create the map!

### Creating bench map component

Begin by creating a __components/BenchMap__ directory containing an index and
CSS file. The index file should define two functional components: a
`BenchMap`--this component will contain the actual map logic--and a
`BenchMapWrapper`. Add skeletons for these two components and make
`BenchMapWrapper` the default export.

Next install the `@googlemaps/react-wrapper` npm package in your __frontend__
folder. This [package][gmaps-react-wrapper] exports a (named) `Wrapper`
component that will load the Google Maps JavaScript API for use by its children.
Your `BenchMapWrapper` component should accordingly return your `BenchMap`
component wrapped in the `Wrapper` component. Pass any `props` that were passed
to `BenchMapWrapper` through to your `BenchMap`. The `Wrapper` component should
take an `apiKey` prop set to the `REACT_APP_MAPS_API_KEY` environment variable
that you defined in __frontend/.env.development.local__. (If you forget how to
access environment variables in JS, see yesterday's "How to Store API Keys
Securely" reading.)

In your `BenchMap` component, create a `map` state value where a `Map` object
from the Google Maps API will be stored. Initialize it to `null`. Next, use the
[`useRef` hook] to create a ref named `mapRef`. Refs store their value under the
key `current`, so to retrieve or update the value of `mapRef`, you will access
`mapRef.current`.

Like `useState`, `useRef` creates a variable whose value will persist between
renders. Unlike `useState` values, the value of a ref will **NOT** trigger a
re-render when it is updated. Refs thus offer a good option when you want to
store mutable values in state (i.e., values that persist) that do not directly
affect the page's appearance.

Refs are commonly used to point to DOM elements, providing instant access. In
React, you can associate a ref with a DOM element by assigning the ref as a
`ref` attribute on the element. E.g., to assign an `inputRef` to a particular
input field on a form, you could do this:

```js
const inputRef = useRef(null);

// ...

<input ref={inputRef} type="text" ... />
```

Whenever React renders a DOM element with `ref={inputRef}`, it will assign
`inputRef.current` to that element. `inputRef` will thus always point to the
currently rendered version of that input element.

Set your `BenchMap` component to return a single `div` whose `ref` attribute is
assigned to `mapRef`. Since you will eventually use `mapRef` to grab this `div`
and put the map inside it, fill the `div` with the simple word `Map` as a
placeholder.

Still inside `BenchMap`, add a `useEffect` to set up the map. If `map` is not
yet set, create a new `google.maps.Map` object, with the `mapRef` as its first
argument. The second argument should provide good default values for the map,
such as `zoom` level and `center`. However, `BenchMap` should also receive a
`mapOptions` prop to override these defaults. Spread these in. Don't forget to
fill in your dependency array!

`BenchMap` will render markers for whatever `benches` it receives in its props.
To keep track of which benches have been rendered as the `benches` prop changes,
create a `markers` ref. This will store an object whose keys are bench `id`s and
whose values are [`google.maps.Marker` objects][gmaps-marker].

In another `useEffect`, go through each bench in the `benches` prop. If no
marker currently exists for that bench, create one with a `position` pointing to
a [`google.maps.LatLng` object][gmaps-latlng] created from the bench's `lat` and
`lng`. (No custom marker `label` or `icon` at this point.) Any markers that do
not correspond to a bench in `benches` should be removed
(`marker.setMap(null)`).

What about event handlers on these markers? On the bench index page, clicking
on a marker should go to that bench's show page. But clicking on the marker
within the map rendered on the show page should do nothing. Thus, `BenchMap`
needs to be flexible. It should receive a `markerEventHandlers` prop, which is
an object whose keys are event types and whose values are event handlers.

When creating a `Marker`, each handler in `markerEventHandlers` should be
attached to it via `marker.addListener`. The event handler passed as a second
arg to `addListener` should invoke the provided handler in
`markerEventHandlers` with the marker's corresponding `bench` object as an
argument.

Similarly, `BenchMap` should also accept a `mapEventHandlers` prop. In a third
effect, each handler in `mapEventHandlers` should be attached to the `map`
object itself using [`google.maps.event.addListener`][gmaps-addlistener]. Any
arguments passed to the native event handler should be passed to the provided
handler from `mapEventHandlers` -- sometimes, this is an `event` object,
sometimes it is nothing. The `map` object should be passed in as well.

### Render map on bench index and show pages

Now that you've done all the hard work of creating your maps, let's add them to
your pages so you can actually see them!

Render a `BenchMap` on the `BenchIndexPage`. Pass in the following props:

* `benches`, which contains the benches already being selected from the Redux
  store
* `markerEventHandlers`, which contains a `click` handler that navigates to that
  bench's show page via `history.push`
* `mapEventHandlers`, which contains a `click` event handler that navigates to
  the new bench page with a query string containing the selected latitude and
  longitude

To implement that last `click` handler for the map, note that the event object
itself will contain a `latLng` property. Convert this to an object (`toJSON`),
then into a query string (`URLSearchParams`). Use `history.push` with an object
argument to navigate to the new bench page. The object argument should contain a
`pathname` property and a `search` property pointing to the query string.

Also render a `BenchMap` on the `BenchShowPage`. Pass in the following props:

* `benches`, which contains only the current page's bench
* `mapOptions`, which contains a `center` property using the bench's `lat` and
  `lng`

### Map bounds

Instead of rendering every bench in the database on the bench index page (with
some markers off screen), only render those in the current map view. To complete
this feature, you will need to make the following adjustments.

#### __app/controllers/api/benches_controller.rb__

The `benches#index` backend action should accept a `bounds` parameter that will
be passed in the query string. This will be a comma separated list of lat/lng
values (SW lat, SW lng, NE lat, NE lng). If a `bounds` parameter is present,
query only for benches whose lat/lng fall within these bounds.

Remember that you want to keep your controllers lean. Find a way to move the
bulk of the logic of finding benches in bounds to your `Bench` model.

#### __frontend/src/store/benches.js__

The `fetchBenches` now needs to accommodate a query string. It should accept a
`filters` object argument, which should be converted to a query string and
appended to the request url. (If you don't remember how to form a query string,
see the instructions on rendering a `BenchMap` on the `BenchIndexPage` above.)

#### __frontend/src/components/BenchIndexPage/index.js__

The bench index page now needs a `bounds` state value. In `mapEventHandlers`,
add an `idle` event handler. `idle` gets called after each time a user pans the
map. In this handler, set the `bounds` state to `map.getBounds().toUrlValue()`.

In the effect that dispatches `fetchBenches`, add `bounds` as a dependency and
pass it in the `filters` object argument to `fetchBenches`. Then, whenever
`bounds` changes, new benches will be fetched.

### Customizing markers

Finally, customize the markers so they show the price of their respective bench.
See [here][gmaps-marker-options] for `label` and `icon` properties that can be
set.

To customize the marker shape itself, use SVG `path`. See [here][svg].

Great job! You have now incorporated Google Maps into your project. That wasn't
so bad, was it?

**Commit your code,** then proceed to Phase 8 for further enhancements!

[gmaps-react-wrapper]: https://www.npmjs.com/package/@googlemaps/react-wrapper
[`useRef` hook]: https://reactjs.org/docs/hooks-reference.html#useref
[gmaps-marker]: https://developers.google.com/maps/documentation/javascript/markers
[gmaps-latlng]: https://developers.google.com/maps/documentation/javascript/reference/3.47/coordinates?hl=en
[gmaps-addlistener]: https://developers.google.com/maps/documentation/javascript/reference/3.47/event?hl=en#event.addListener
[gmaps-marker-options]: https://developers.google.com/maps/documentation/javascript/reference/marker?hl=en#MarkerOptions
[svg]: https://developer.mozilla.org/en-US/docs/Web/SVG/Attribute/d