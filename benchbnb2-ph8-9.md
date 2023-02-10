# BenchBnB, Phases 8-9

[Live Demo]

In Phases 8-9, you will add filters (Phase 8) and photos (Phase 9) to your
benches.

[Live Demo]: https://aa-bench-bnb.herokuapp.com/

## Phase 8: Bench index filters

Let's face it: your bench needs vary. When you're on a date, you want a quaint,
2-seat bench that will keep the two of you in close proximity. When you're out
with your family, you want a full 6-seater so everyone can have their own spot
with room to spare. Wouldn't it be great if you could specify the bench size you
wanted so you didn't have to waste all that time checking out benches that were
never going to suit your present needs? Let's make it happen!

Filtering out unwanted benches essentially means limiting the number of benches
that your index returns. In fact, you've already implemented one such filter,
having your index omit any benches that fall outside the map
boundaries. Now you just need to add another filter based on seat size.

Add a `FilterForm` component inside the __components/BenchIndexPage/__ folder.
The component should take in values for `minSeating` and `maxSeating`, as well
as setter functions for each. It should return a structure containing two of
your `Input` components, one for updating each of the min and max seating
values.

> **Note:** The wrapping structure does not need to be a `form` because nothing
> needs to be submitted. The `Inputs` already do all the necessary work by
> updating the appropriate values whenever a user changes those values.

Import the `FilterForm` into `BenchIndexPage` and render it above the
`BenchList`. Create `minSeating` and `maxSeating` state in the `BenchIndexPage`
component. Pass the current values and setters to `FilterForm`.

Include the current `minSeating` and `maxSeating` as properties within the
`filters` argument passed to `fetchBenches`. Add those values as dependencies of
the effect.

Finally, refactor the `benches#index` controller action to accommodate
`minSeating` and `maxSeating` query string parameters.

**Commit your code** once you have the new filter working!

## Phase 9: Bench images

The interactive map is great, but the modern web wouldn't be what it is without
images. It's time to upgrade your app to handle bench photos!

### Set up Active Storage

To add image-handling capabilities, you first need to activate Active Storage,
one of the packages that `--minimal` skips. The instructions below will walk you
through that process, or you can read the official Active Storage installation
instructions under 'Setup' in the [Rails docs][active-storage].

To begin, go to __config/application.rb__ and uncomment the `require
"active_storage/engine"` line (around line 8). Then run

```sh
rails active_storage:install
```

This command will create a migration to install the 3 new tables needed by
Active Storage: one for handling blobs, one for variant records, and one for
attachments. Migrate the migration as you would any other (`rails db:migrate').

Next, add a __storage.yml__ file to the __config__ directory. This file declares
your Active Storage services. For example, if you are using Amazon S3 services
for storage, then you would have a section setting up the relevant parameters
for connecting to and accessing Amazon. For now, just declare two services,
`local` and `test`:

```yml
# config/storage.yml

local:
  service: Disk
  root: <%= Rails.root.join("storage") %>

test:
  service: Disk
  root: <%= Rails.root.join("tmp/storage") %>
```

This configuration sets up the `local` storage service to use your `Disk` and
store the files in a __storage/__ directory. Similarly, the `test` service
also uses your `Disk`, but it stores information in a __tmp__ folder, namely,
__tmp/storage__.

Now that you have set up your services, you need to tell Rails which service to
use for each environment. If you one day decide to deploy your Bench Bnb app,
then you will want to set up a production-appropriate service--such as
Amazon--and edit your production environment to use that service. For now,
however, you only need to worry about `development`. Go to
__config/environments/development.rb__ and tell Rails to use the `local` service
in `development`:

```rb
# config/environments/development.rb

Rails.application.configure do
  # ...

  config.active_storage.service = :local
end
```

You don't want to push your local storage to GitHub, so add the following lines
to your top-level __.gitignore__:

```plaintext
/storage/*
!/storage/.keep 
```

> A `!` in a __.gitignore__ file serves to negate the pattern, i.e., it
> signifies that the following pattern should be **included**. It is used when a
> previously specified pattern would call for a necessary file to be ignored. So
> these two lines together say to ignore everything (`*`) in the top-level (`/`)
> `storage/` directory except (`!`) __.keep__. (Top level here is relative to
> the directory of the __.gitignore__.) A __.keep__ file serves simply to keep
> the directory from being empty so that git will include it.

One more issue has to be resolved. By default, Rails loads all the routes
(views, helpers, assets, etc.) for the main application before loading the
routes needed for engines like Active Storage. Normally that's not a problem,
but it will be for this app.

The issue is the catch-all route (`*path`) that you defined in
__config/routes.rb__. As you may remember, Rails uses this route to serve up the
frontend's static pages for any route that does not match one of the defined
Rails routes. (For more information, see the deployment instructions for
Authenticate-Me.) Since Rails loads the main app's routes before the Active
Storage routes, however, the Active Storage routes also get captured by the
catch-all route. If a request comes in for `rails/active_storage/...`, Rails
sees that it doesn't match any of the specified API routes and passes it on to
the frontend. The Active Storage routes can't break through!

You could solve this problem by using a lambda function to explicitly exclude
any route that begins with `rails/active_storage` from your catch-all in
__routes.rb__:

```rb
# config/routes.rb

get '*path', to: "static_pages#frontend_index", constraints: ->(req) {
  req.path.exclude? 'rails/active_storage'
}
```

A more comprehensive solution simply changes the Rails load order in
__config/application.rb__ so that no other Rails routes will be caught by the
catch-all:

```rb
# config/application.rb

config.railties_order = [:all, :main_app]
```

Choose one of these solutions and implement it. (For more on the problem and
these potential solutions, see this [Rails issue][catch-all].)

That's it! Active Storage should be ready to go.

### Adding photos: Backend

If you only need to attach (at most) one file to each record, you can establish
this Active Storage association in your model using the [`has_one_attached`]
macro. Accordingly, add `has_one_attached :photo` alongside the other
associations declared in your `Bench` model.

Your `benches#create` action now needs to accommodate uploading a photo file.
Significantly, this means that requests to `benches#create` will no longer be
formatted as JSON; their `Content-Type` will be `multipart/form-data`. This
change in content type then has implications for your strong params.

Remember that your strong params method (`bench_params`) looks to permit
parameters nested under the model name (`require(:bench)`). Your frontend,
however, sends all of the attribute parameters at the top level; they only
appear under the `:bench` key because **Rails** wraps any attribute parameters
under the model name. By default, however, Rails performs this wrapping service
only for __JSON__ requests. You accordingly need to tell it to wrap the
parameters for `multipart` requests as well. This is easy enough to do. Just
call `wrap_parameters` in your `Api::BenchesController`:

```rb
# app/controllers/api/benches_controller.rb

class Api::BenchesController < ApplicationController
  wrap_parameters format: :multipart_form
  # ...
end
```

Before leaving your benches controller, one other issue needs attention. You
want to permit `:photo` to be passed into your `Bench` constructor through
strong params so it can be attached. The first thing to do is accordingly to add
`:photo` to the list of permitted params in `bench_params`. That step alone,
however, will not suffice to pass `:photo` into your constructor in
`benches#create`. Can you see the problem?

The problem is that `:photo` is not a column/attribute of the `benches` table;
it is an Active Storage association. As a result, Rails will not wrap it under
the `bench` key like the other parameters. Fortunately, the solution requires
just a small tweak to your `wrap_parameters` call:

```rb
# app/controllers/api/benches_controller.rb

class Api::BenchesController < ApplicationController
  wrap_parameters include: Bench.attribute_names + [:photo], format: :multipart_form
  # ...
end
```

Now `benches#create` should correctly attach any `:photo` passed in with a new
bench request.

### Adding photos: Frontend

Begin the frontend adjustments by adding `photoFile` and `photoUrl` state
values to the `BenchFormPage` component. `photoFile` will be passed in the
create bench request. `photoUrl` will be used to render the photo that is meant
to be uploaded as a preview.

Next, in the form itself, add an `Input` of type `file` for uploading a photo.
For the `onChange` prop, pass a callback named `handleFileChange` that you will
define shortly. After this `Input`, if `photoUrl` is truthy, render an image of
that photo with a heading of `Image preview`. Use `photoUrl` as the `src`, and
set an appropriate `height` for a preview.

In `handleFileChange`, save `e.target.files[0]` (the first--i.e., only--file
uploaded to that input) under a variable named `file`. To generate a URL for the
preview, create a [`FileReader`] instance, then invoke [`readAsDataURL`] with
`file` passed as the argument. This will trigger an async action. Define an
`onload` property on the `FileReader` instance that points to a callback that
will run after `readAsDataURL` completes. Set the `photoFile` state to the
`file` and the `photoUrl` state to `fileReader.result` (i.e., the result of
`readAsDataURL`).

If you set up `handleFileChange` correctly, it should look something like this:

```js
const handleFileChange = e => {
  const file = e.target.files[0];
  if (file) {
    const fileReader = new FileReader();
    fileReader.readAsDataURL(file);
    fileReader.onload = () => {
      setPhotoFile(file);
      setPhotoUrl(fileReader.result);
    };
  }
}
```

In `createAction` in the submission event handler, create a [`FormData`] object.
Append every input state value to this `FormData` instance. Also append the
`photoFile` if it is present. Pass the `FormData` object to `createBench`.

Finally, adjust your `csrfFetch` (__frontend/src/store/csrf.js__) so that it
will not set the `Content-Type` header to `application/json` if `options.body`
is an instance of `FormData`.

> **Note:** Why not just set the `Content-Type` header to `multipart/form-data`
> in `createBench` and be done with it? The `multipart/form-data` header cannot
> be set manually because it needs to contain information about the multi-part
> boundaries.

Once you get images working, definitely **commit your code.** That's quite an
accomplishment!

Only Phase 10 remains to be conquered...

[active-storage]: https://guides.rubyonrails.org/active_storage_overview.html#setup
[catch-all]: https://github.com/rails/rails/issues/31228
[`has_one_attached`]: https://api.rubyonrails.org/v7.0.3.1/classes/ActiveStorage/Attached/Model.html#method-i-has_one_attached
[`FileReader`]: https://developer.mozilla.org/en-US/docs/Web/API/FileReader
[`readAsDataURL`]: https://developer.mozilla.org/en-US/docs/Web/API/FileReader/readAsDataURL
[`FormData`]: https://developer.mozilla.org/en-US/docs/Web/API/FormData