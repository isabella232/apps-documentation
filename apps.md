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

*name* - A user facing name that will be displayed for your app.

*description* - A user facing description of what your app does.

*signal_url* - The URL where signals will be posted for this app. See more on signals [here](signals.md)

*entry_url* - URL where a user will be directed with a signed token in order to complete configuration of your app.

*signals* - An array of the names of signals your app wants to listen for (if any).

You now have a new Lessonly App!

