# BenchBnB: Part 2

## Phase 7: Bench Map

### Creating Bench Map Component

- Install the `@googlemaps/react-wrapper` npm package. This provides a `Wrapper`
  component that will load the Google Maps API only when it is rendered. The API
  key is provided as a prop.
- Create a `components/BenchMap` directory with an index and CSS file. The index
  file should define two components: a `BenchMap` (with the actual map logic),
  and a `BenchMapWrapper`, which renders a `Wrapper` from
  `@googlemaps/react-wrapper` with an `apiKey` prop pointing to the
  `REACT_APP_MAPS_API_KEY` environment variable. In
  `frontend/.env.development.local`, this environment variable should be set to
  an actual API key.
- Inside the `Wrapper`, `BenchMapWrapper` should render `BenchMap` with whatever
  props were passed to `BenchMapWrapper`.

- `BenchMap` should have a `map` state value where a `Map` object from the
  Google Maps API will be stored. It should also have a `mapRef` ref for the
  `div` in which the map will be rendered. It should render this `div` with
  `ref={mapRef}`.
- A `useEffect` should be added for setting up the map. If `map` is not yet set,
  a new `google.maps.Map` object should be created, with the `mapRef` as its
  first argument. The second argument should provide good default values for the
  map, such as zoom level and center. However, `BenchMap` should also receive a
  `mapOptions` prop to override these defaults. Spread these in.
- `BenchMap` will render markers for whatever `benches` it receives in its
  props. To keep track of which benches have been rendered as the `benches` prop
  changes, a `markers` ref should be created. This will store an object whose
  keys are bench `id`s and whose values are `google.maps.Marker` objects.
- In an effect, go through each bench in the `benches` prop. If there is not
  already a marker for that bench, create one with a `position` pointing to a
  `google.maps.LatLng` object created from the bench's `lat` and `lng`. (No
  custom marker `label` or `icon` at this point.) Any markers that do not
  correspond to a bench in `benches` should be removed (`marker.setMap(null)`).
- What about event handlers on these markers? On the Bench Index page, clicking
  on a marker should go to that bench's show page. But clicking on the marker
  within the map rendered on the show page should do nothing. Thus, `BenchMap`
  needs to be flexible. It should receive a `markerEventHandlers` prop, which is
  an object whose keys are event types and whose values are event handlers.
- When creating a `Marker`, each handler in `markerEventHandlers` should be
  attached to it via `marker.addListener`. The event handler passed as a second
  arg to `addListener` should invoke the provided handler in
  `markerEventHandlers` with the marker's corresponding `bench` object as an
  argument.
- Similarly, `BenchMap` should accept a `mapEventHandlers` prop. In an effect,
  each handler should be attached to the `map` object itself. Any arguments
  passed to the native event handler should be passed to the provided handler
  from `mapEventHandlers` -- sometimes, this is an `event` object, sometimes it
  is nothing. The `map` object should be passed in as well.

### Render Map on Bench Index and Show Pages

- Render a `BenchMap` on the `BenchIndexPage`. Pass in the `benches` that
  already are being selected from the Redux state.
- Pass in a `markerEventHandlers` with a `click` handler that navigates to that
  bench's show page via `history.push`.
- Pass in a `mapEventHandlers` with a `click` event handler. The event object
  itself will contain a `latLng` property, which should be converted to an
  object (`toJSON`), then into a query string (`URLSearchParams`). Then, use
  `history.push` with an object argument to navigate to the new bench page, with
  a `pathname` property and a `search` property pointing to the query string.
- Render a `BenchMap` also on the `BenchShowPage`. Pass in a `benches` prop
  containing only the current page's bench. Pass in a `mapOptions` prop, with a
  `center` property using the bench's `lat` and `lng`.

### Map Bounds

- Instead of rendering every bench in the database on the Bench Index Page (with
  some markers off screen), instead you should only render those in the current
  map view.
- The `benches#index` backend action should accept a `bounds` parameter which
  will be passed in the query string. This will be a comma separated list of
  lat/lng values (SW lat, SW lng, NE lat, NE lng). If a `bounds` parameter is
  present, query only for benches whose lat/lng fall within these bounds.
- The `fetchBenches` now needs to accommodate a query string. It should accept a
  `filters` object argument, which should be converted to a query string and
  appended to the request url.
- The Bench Index Page now needs a `bounds` state value. In `mapEventHandlers`,
  add an `idle` event handler. `idle` gets called after each time the user pans
  the map. In this handler, set the `bounds` state to
  `map.getBounds().toUrlValue()`.
- In the effect that dispatches `fetchBenches`, add `bounds` as a dependency and
  pass it in the `filters` object argument to `fetchBenches`. Then, whenever
  `bounds` changes, new benches will be fetched.

### Customizing Markers

- Customize markers so they show the price of their respective bench.
- See
  https://developers.google.com/maps/documentation/javascript/reference/marker?hl=en#MarkerOptions
  for `label` and `icon` properties that can be set.
- For customizing the marker shape itself, you can use SVG `path`. See
  https://developer.mozilla.org/en-US/docs/Web/SVG/Attribute/d

## Phase 8: Bench Index Filters

- Add a `FilterForm` component inside __components/BenchIndexPage/__. Render
  above the `BenchList`.
- Create min and max seating state in the bench index component. Pass the
  current values and setters to `FilterForm`.
- Include the current `minSeating` and `maxSeating` as properties within the
  `filters` argument passed to `fetchBenches`. Add those values as dependencies
  of the effect.
- Filter form should render two number inputs for min and max seating.
- `benches#index` controller action should be refactored to accommodate
  `minSeating` and `maxSeating` query string parameters.

## Phase 9: Bench Images

- In __config/application.rb__, uncomment `require "active_storage/engine"`
  (around line 8).
- Follow the Active Storage installation instructions under 'Setup'
  [here][active-storage]. Migrate. Add storage.yml. Update config/environments/development.rb.
- Add storage to .gitignore
- Add a `has_one_attached :photo` Active Storage association to the Bench model.
- The `benches#create` action now needs to accommodate uploading a photo file.
  As a result, the request body will no longer be JSON, but instead
  `multipart/form-data`
  Need to change csrf.js; headers need to be set automatically (include multipart boundary)
  - Add `wrap_parameters include: Bench.attribute_names + [:photo], format:
    :multipart_form` to the Benches controller.
- In the `BenchFormPage` component, add `photoFile` and `photoUrl` state values.
  `photoFile` will be passed in the create bench request. `photoUrl` will be
  used to render the photo that is meant to be uploaded as a preview.
- Add a input of type `file` for uploading a photo. If `photoUrl` state is
  truthy, render an image of that photo with a heading of `Image preview`.
- In the change handler for the file input, save `e.target.files[0]` (the first,
  i.e. only, file uploaded to that input) under a variable `file`. To generate a
  URL for the preview, create a `FileReader` instance, then invoke
  `readAsDataURL` with the file. This will trigger an async action. Define an
   `onload` property on the `FileReader` instance that points to a callback
   which runs after `readerAsDataURL` completes. Set the `photoFile` state to
   the `file` and the `photoUrl` state to `fileReader.result` (the result of
   `readAsDataURL`).
- In the submission event handler, create a `FormData` object. Append every
  input state value to this `FormData` instance. Also append the `photoFile` if
  it is present. Pass the `FormData` object to `createBench`.

## Phase 10: Polish

### Require Current User With Modal

- Rather than redirecting to the sign up page when there isn't a current user,
  (for instance on the BenchFormPage), it would be nice if there was a session
  modal that just overlaid the page. Then, after signing up or logging in, the
  modal could close and the user's context would not be lost.
- To create reusable session components, create a `components/SessionForms`
  directory. Move the LoginFormModal component to `SessionForms/index.js`. It
  should now render only a Modal: the containing button and the logic around
  toggling its visibility was hardcoded for the Navigation component context.
  Move this logic to the `Navigation` component.
- Move the `LoginForm` component file to this directory as well. Extract the
  signup form itself from the `SignupFormPage` component and save it to a
  `SignupForm.js` file in `SessionForms`.
- Create a generic `SessionModal` in `SessionForms/index.js`. This should render
  either the `LoginForm` or the `SignupForm`, with a button at the bottom that
  toggles between the two.
- Render a `SessionModal` on the `BenchFormPage` if there is no current user.

### Highlight Bench List Items When Hovering Over Markers

- Add a `highlightedBench` state to the `BenchIndexPage`.
- Add handlers to `markerEventHandlers` for `mouseover` (set `highlightedBench`
  to marker's associated bench) and `mouseout` (clear `highlightedBench`).
- Pass `highlightedBench` to `BenchMap` as a prop. Create a different marker
  style for highlighted benches (e.g., inverted colors). Add an effect to
  `BenchMap`: whenever the `highlightedBench` changes, ensure every marker whose
  bench is not the `highlightedBench` has standard styling, and any marker whose
  bench is the `highlightedBench` has the custom styling. (Can use `setLabel`
  and `setIcon`.)
- Pass `highlightedBench` to the `BenchList` as well. Add custom styling to
  whichever `BenchListItem` corresponds to the `highlightedBench`. 
- Pass `setHighlightedBench` to `BenchList`. On hovering over a `BenchListItem`,
  set `highlightedBench` to that bench.

## Bonus Phase

### Option 1: Style and Structure like AirBnB

### Option 2: Multiple Photos + Image Carousel

### Option 3: Favorites

[maps-api]: https://developers.google.com/maps/documentation/javascript/overview
[active-storage]: https://guides.rubyonrails.org/active_storage_overview.html
[maps]: https://www.google.com/maps/place/San+Francisco,+CA
[`Faker`]: https://github.com/faker-ruby/faker
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