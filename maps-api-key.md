# Google Maps API Key

For the BenchBnB project, you'll need a Google Maps API key for the
maps feature to work. The Maps API requires that you have a Google account with
billing enabled (i.e., your credit card saved). In this practice, you will set
up your Google account and create an API key so you will be able to hit the
ground running when you get to BenchBnB.

As developers, there will be several occasions where you'll be billed for the
usage of third-party APIs. You are almost certainly not going to use the Maps
API enough in BenchBnB for you to actually be billed anything--especially if you
properly safeguard your API key--so this is a fantastic opportunity to learn
more about responsible API-key stewardship. Specifically, you'll learn more
about how to protect your API keys from bad actors.

## Setting up an API key

To generate a working API key, you need to do the following:

1. Create a Google developers project.
2. Enable Maps Javascript API in that project.
3. Go to credentials to generate an API key.
4. Add billing info to unlock the full set of Maps functionality.
5. Add the API key to your BenchBnB project.

In this practice, you will complete steps 1-4. You will complete step 5 as part
of the BenchBnB project.

### Create a Google developers project

To get started, first navigate to the [Google Cloud console][gcp-console].

You will need a Google account to be able to log in to the Google Cloud console.

Once you are logged in, click on the `Select a Project` button in the nav bar.
In the newly opened modal, select `New Project`.

![New GCP Project example][maps-api-1]

Choose a name for your project, something like `benchbnb`. Leave the `Location`
set to `No organization`. Then, click `Create`.

![New GCP Project example 2][maps-api-2]

[gcp-console]: https://console.cloud.google.com/
[maps-api-1]: https://assets.aaonline.io/fullstack/react/projects/bench_bnb/maps_api_1.png
[maps-api-2]: https://assets.aaonline.io/fullstack/react/projects/bench_bnb/maps_api_2.png

### Enable Maps JavaScript API

Once you've created your project, make sure you have selected your newly created
project. Then, open the side menu, select `APIs & Services` > `Library`.

Search for and select `Maps JavaScript API`. Now, `Enable` this API for your
project. (Again, be sure you have selected your newly created project when
enabling this API.)

![Enable JavaScript API][maps-api-3]

[maps-api-3]: https://assets.aaonline.io/fullstack/react/projects/bench_bnb/maps_api_3.png

### Generate an API key

Open the side menu navigation again. This time, select `APIs & Services` >
`Credentials`.

Then, click `Create credentials` and select the `API key` option.

Once you've generated a new API key, click `Edit key`. This will take you to a
new page where you can now rename the API key to something more specific (e.g.,
'BenchBnB Maps API Key'). You should also restrict your API key so that it can
only call on the `Maps JavaScript API`.

![Restrict API Access][maps-api-4]

In general, you want to follow the
[Principle of Least Privilege][principle-least-privilege] when it comes to
managing your API keys and what they have access to.

Access Google's official docs for the [Maps JavaScript API][maps-javascript-api]
for more info on this particular API.

[principle-least-privilege]: https://en.wikipedia.org/wiki/Principle_of_least_privilege
[maps-api-4]: https://assets.aaonline.io/fullstack/react/projects/bench_bnb/maps_api_4.png
[maps-javascript-api]: https://developers.google.com/maps/documentation/javascript/get-api-key

### Add billing info

Finally, for your Maps API to work without restrictions, you'll need to
add billing info to your Google account. To do this, open the side navigation
menu again and go to `Billing` to add your credit card info.

## Final thoughts

Congratulations! You've successfully set up your Maps API key. You'll get the
opportunity to add it to an actual project when you get to BenchBnB.

**Important Note**: Be very careful with your API key! There have been multiple
anecdotes from students who accidentally pushed API keys to a public GitHub
repo. Those keys were then scraped by bad actors who racked up tens of thousands
of dollars in charges using those keys. **Never put your key in a file that will
get pushed to GitHub!**