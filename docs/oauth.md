# Some notes on using OAuth with Databox

see...
an [oauth issues](https://github.com/me-box/databox/issues/193),
[old notes from the strava driver](https://github.com/cgreenhalgh/databox-driver-strava/blob/master/docs/implementation-notes.md#oauth-notes),
a possible [oauth intermediary service](https://github.com/me-box/core-oauth-intermediary),
[oauth support in the twitter driver](https://github.com/me-box/driver-twitter#implementing-oauth-in-databox),
[`databox_oauth_redirect` implementation in core-ui](https://github.com/me-box/core-ui/blob/master/ui/src/app.js#L27),
and [in a version of the app](https://github.com/me-box/databox-app/blob/8b661272064ed94596ebb2cb08670f1813ab6736/src/app.js#L78),
[or here?](https://github.com/me-box/databox-app/blob/8b661272064ed94596ebb2cb08670f1813ab6736/src/app.js#L78)

twitter driver
uses [npm "oauth": "^0.9.15"](https://www.npmjs.com/package/oauth),
defaults to "https://localhost/driver-twitter/ui/oauth" as redirect url,
does `res.render('oauth_redirect', {url: 'https://api.twitter.com/oauth/authenticate?oauth_token=' + settings.requestToken})`,
which uses [message databox_oauth_redirect](https://github.com/me-box/driver-twitter/blob/master/src/views/oauth_redirect.pug#L4),
on `ui/oauth` redirects to [core-ui driver view](https://github.com/me-box/driver-twitter/blob/master/src/main.js#L93).

Not clear atm where `databox:` URL scheme is introduced or handled.

This is changing in the [latest core-ui](https://github.com/ktg/core-ui) -
see [corresponding twitter driver](https://github.com/ktg/driver-twitter/tree/oauth),
which uses `let callbackURL = parent.getOauthCallbackURL();` and
`parent.startOauth(url);`

This is probably not in the 0.5.7 app.
