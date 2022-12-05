# BenchBnB, Phases 1-2

[Live Demo]

In Phases 1-2, you will create your Bench model on the backend (Phase 1) and set
up your frontend to handle benches in the Redux store (Phase 2).

[Live Demo]: https://aa-bench-bnb.herokuapp.com/

## Phase 1: Bench backend

Create a `Bench` model and table. It should have the following fields:
  
|        column | data type | details              |
| ------------: | --------: | -------------------- |
|       `title` |  `string` | not null             |
| `description` |    `text` | not null             |
|       `price` | `integer` | not null             |
|     `seating` | `integer` | not null; default: 2 |
|         `lat` |   `float` | not null             |
|         `lng` |   `float` | not null             |

Be sure to include `presence` validations for each field; in addition, include a
validation that `price` is between 10 and 1000.

Next, create an `Api::BenchesController` to handle your API requests. It will
need `index`, `create`, and `show` actions:

- `index` should render all benches as JSON under a key of `benches`
- `create` should use strong params to create a new `Bench`
  - If it saves successfully, render it as JSON under a key of `bench`
  - If it fails any validations, render the full error messages under a key of
    `errors` with a status of `422`
- `show` should render the bench specified by `params[:id]` as JSON under a key
  of `bench`
- Use Jbuilder views to structure your API's JSON responses.

Next, add routes for your `BenchesController` actions. These should be
namespaced under `api` and return JSON by default.

Finally, populate __seeds.rb__ with at least five benches. Use real longitudes
and latitudes for locations within a city of your choice. If you right click a
spot in [Google Maps][maps], the top option in the context menu will copy that
spot's latitude and longitude to your clipboard. You can use [`Faker`] to create
randomized descriptions or make up your own.

Test your benches API in your browser console using `csrfFetch`.

**Now would be a good time to commit your repository.**

## Phase 2: Bench Redux

Next, you will build the pieces necessary for fetching, updating, and managing
bench data on the frontend.

### Goal

Your goal is to build a bench slice of state that has the following shape:

```js
benches: {
  1: {
    id: 1,
    title: "...",
    description: "...",
    price: 100,
    seating: 2,
    lat: 0.0,
    lng: 0.0
  },
  2: {
    id: 2,
    title: "...",
    description: "...",
    price: 100,
    seating: 2,
    lat: 0.0,
    lng: 0.0
  },
  // ...
}
```

Note that the bench slice of state is normalized: it is an object where each key
is the `id` of a particular bench, pointing to a shallow (non-nested) object
containing the data for that bench.

### `fetchBenches`

Begin by creating a file __frontend/src/store/benches.js__.

Write and export a thunk action creator `fetchBenches`, which should use
`csrfFetch` to hit the `benches#index` action of your backend API (run `rails
routes` if your forget the necessary path and verb to create this request).

For now, simply `console.log` the data you get back, after parsing it as JSON
(you'll `dispatch` an action with the response data once you've built a POJO
action creator). To test that your thunk action creator is successfully fetching
your bench data, import it in __frontend/src/index.js__ and add it as a property
to the window:

```js
// frontend/src/index.js

// ... other imports
import * as benchActions from './store/benches';

// ...

if (process.env.NODE_ENV !== "production") {
  // ...
  window.benchActions = benchActions;
}
```

Then, run the following in your browser console:

```js
> await store.dispatch(benchActions.fetchBenches())
```

You should see your bench data logged in the console. Take special note of the
structure of the data that is coming in. What are the top-level keys? Which data
attributes are included for each bench? What determines the shape and content of
this JSON response?

Once you've determined you're successfully fetching bench data, write a POJO
action creator, `setBenches`, that takes in a `benches` argument--an object
containing normalized `benches`--and returns a POJO with a `type` of
`benches/setBenches` and a `payload` pointing to the `benches`. You should save
the string `benches/setBenches` to a variable (e.g. `SET_BENCHES`) and use that
variable in your action creator.

> **Note:** If you don't remember why you should create a variable for each
> action `type`, imagine the following situation: in your reducer, you made a
> typo in your `case` for `benches/setBenches` and wrote `benches/setBences`
> instead. What would happen? On the other hand, if you made a typo while
> referencing the variable `SET_BENCHES`, writing `SET_BENCES` instead, what
> would happen?

A typical action created by this POJO action creator should be structured like
so:

```js
{
  type: 'benches/setBenches',
  payload: {
    1: {
      id: 1,
      title: "...",
      description: "...",
      price: 100,
      seating: 2,
      lat: 0.0,
      lng: 0.0
    },
    2: {
      id: 2,
      title: "...",
      description: "...",
      price: 100,
      seating: 2,
      lat: 0.0,
      lng: 0.0
    },
    ...
  }
}
```

From your `fetchBenches` thunk action creator, create and `dispatch` an action
using `setBenches`, passing in the benches you received from your backend API's
response.

Once again, use `store.dispatch` on the window to test `fetchBenches` in the
browser console. This time, check to make sure a POJO action with the right
`type` and `payload` is being dispatched.

Next, create a function `benchesReducer` and make it your `export default`. The
initial `state` should be an empty object. As usual, create a `switch` block
based on `action.type` with a `default` case where you return the previous
`state`. Add a case for `SET_BENCHES`, where you should simply return the
`action`'s `payload` as the new benches slice of state.

Finally, head to  __frontend/src/store/index.js__, import `benchesReducer`, and
add a `benches` slice of state to `combineReducers`. Use `store.dispatch` on the
window to test `fetchBenches` again; this time, you should see your Redux state
populated with a normalized `benches` slice of state.

### `fetchBench` and `createBench`

Next, you will add two more thunk action creators, `fetchBench` and
`createBench`:

- `fetchBench` should take in a `benchId` and make a request to `GET
  /api/benches/:id` with `benchId` interpolated into the `:id` dynamic segment.
- `createBench` should take in a `benchData` object and make a request to `POST
  /api/benches`, with `benchData` as the request body.

Begin by `console.log`-ing your backend's response in each thunk action. Test by
`dispatch`-ing each thunk action in your browser console. Be sure to supply the
appropriate arguments. For `createBench`, this means you'll need a dummy bench
object--to save time, you can hardcode one and save it on the `window`.

Next you'll write an `addBench` POJO action creator and an `ADD_BENCH` reducer
`case` statement. Both`fetchBench` and `createBench` should `dispatch` an
`addBench` action with the bench received from the backend.

Write an `addBench` POJO action creator with a `type` of `benches/addBench` and
a `payload` pointing to a single `bench` object. Test that you are creating a
POJO action with the right `type` and `payload` before continuing.

Add an `ADD_BENCH` case statement to your reducer. This time, merge the new
bench into your existing benches slice of state, under a key of its `id`. Test
that the fetched/created bench is being properly merged into the benches slice
of state!

**Now would be a good time to commit your repository.**

[maps]: https://www.google.com/maps/place/San+Francisco,+CA
[`Faker`]: https://github.com/faker-ruby/faker