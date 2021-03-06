Title: Reading a Twitter account from Clojure
Date: 2011-05-09 06:52
Author: gmwils
Category: technology

For my [photography news][] site, I want to use Twitter as a data source
for popular links. This involves getting tweets out of Twitter and into
MongoDB.

The first step is to be able to read the tweets from an account using
Clojure. The library that holds the key is [clojure-twitter][].

To try this out, I created a new lein project, and added the following
to project.clj:

    :::clojure
    (defproject twitter "1.0.0-SNAPSHOT"
      :description "Sample project to read in a tweet stream"
      :dependencies [[org.clojure/clojure "1.2.1"]
                     [clojure-twitter "1.2.5"]]
      :dev-dependencies [[swank-clojure "1.3.0"]])

Then run `lein deps` and start working out the code.

The [example][clojure-twitter] is a good start, however it the various
authentication keys didn't quite make sense to me on first look.

After [registering][] an [application][], you will need to copy four
strings from your Twitter account:

-   **Consumer key** - identifier for your [application][], found with
    your app details.
-   **Consumer secret** - shown with your consumer key and almost twice
    as long. Keep this secret.
-   **Access Token** (oauth\_token) - identifier for your user, found
    under the "My Access Token" button next to your application details.
-   **Access Token Secret** (oauth\_token\_secret) - shown with your
    Access Token

The access token took me a little while to find. Yours will be at a URL
similar to:

    https://dev.twitter.com/apps/123456/my_token

Where 123456 is the identifier of your application.

Armed with these four pieces of information, the code is quite simple.
First include the API:

    :::clojure
    (ns twitter.core
      (:require [twitter :as twitter]
                [oauth.client :as oauth]))

Then setup the various keys and secret phrases:

    :::clojure
    ;; Make a OAuth consumer
    (def oauth-consumer
      (oauth/make-consumer "N0tr3a11ym1k3y" ;; consumer key
                           "R23mnasdfphsfjas08ujhasf08aslfkjasfd234asfd" ;; consumer secret
                           "https://api.twitter.com/oauth/request_token"
                           "https://api.twitter.com/oauth/access_token"
                           "https://api.twitter.com/oauth/authorize"
                           :hmac-sha1))
    (def oauth-access-token
      "1234456789-asdfjklkjasdfjkllASDFASDF")
    (def oauth-access-token-secret
      "CompletelyRandomStringWithNumbersAndLetters")

Obviously, include your own keys and tokens.

The Clojure API is easy to read from the [source][] and follows closely
to Twitter's API.

I chose to use the https version of the API and also scope the
authentication. To read the most recent tweets from my followers:

    :::clojure
    (defn latest-tweets [count]
      (twitter/with-https
        (twitter/with-oauth
          oauth-consumer
          oauth-access-token
          oauth-access-token-secret
          (twitter/home-timeline :count (str count)))))

I'm not sure why the limit parameter only takes a string. Now after
running `lein swank` and connecting up Emacs, I can run:

    :::clojure
    user> (use :reload 'twitter.core)
    nil
    user> (first (map :text (latest-tweets 1)))
    "OK, that's 20 videos this weekend alone I posted at http://t.co/Vm2sVQV Hope you like (please subscribe if you do!)"

There is quite a lot of information returned for each tweet, such as
conversation threads, user info and retweet data. Now I just need to
decide how much of it to store for later analysis.

  [photography news]: http://photozeit.org/
  [clojure-twitter]: https://github.com/mattrepl/clojure-twitter
  [registering]: https://dev.twitter.com/apps/new
  [application]: https://dev.twitter.com/apps
  [source]: https://github.com/mattrepl/clojure-twitter/blob/master/src/twitter.clj
