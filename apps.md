# Lessonly App Overview

# Creating a Lessonly App

Creating a new app is a manual process right now. You first need to open a console in the environment where you want to create the app, then use the following command:

```Ruby
Demux::App.create(
  name: "Acme App",
  description: "A helpful description of my app",
  signal_url: "https://acme.com/webhooks",
  entry_url: "https://acme.com/configure",
  signals: ["test", "lesson_editing"]
)
```

Let's break down the arguments that we supplied.

***name*** - A user facing name that will be displayed for your app.

***description*** - A user facing description of what your app does.

***signal_url*** - The URL where signals will be posted for this app. See more on signals [here](signals.md)

***entry_url*** - URL where a user will be directed with a signed token in order to complete configuration of your app.

***signals*** - An array of the names of signals your app wants to listen for (if any).

You now have a new Lessonly App!

# Installation and Configuration

A Lessonly App will be listed on the integrations page when a company is authorized to install it. Here is how they appear on that installation page:

![](assets/lessonly-apps-list.png)


If a app has not been installed on this company before, the use will be presented with the "Install" button. When they click the install button, a `Demux::Connection` will be created between that company and the app. The user will then be redirected to the `entry_url` of the app along with a JWT. The URL they are redirected too for the app in our create example will look like this with the token attached: https://acme.com/configure?token=arst15earstne435irstie34arst4

The JWT is signed using the apps `secret`. When your app receives the request to this entry point, you should verify the token before using it's contents. Here is an example of how to do that in Ruby using the [ruby-jwt gem](https://github.com/jwt/ruby-jwt):

```Ruby
token_contents = JWT.decode(
  token,
  secret_from_app,
  true,
  algorithm: "HS256"
)
```

Another good example would be to look at how this is done in our [test app](https://github.com/lessonly/lessonly_apps_test_dummy)

The contexts of the token once decoded will look something like this:
```JSON
[
  {"data": {"account_id":9}, "exp": 1594737582},
  {"typ": "JWT", "alg": "HS256"}
]
```

Most of the contents of that token will be used to verify the JWT and make sure it hasn't expired. The "data" key contains the account_id, which is the lessonly company ID for the company that is installing the app. You can use the information in this data field to provision the newly installed account.
