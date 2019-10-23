<h1>Setting up a PHP OAuth Client</h1>

Our firm usually recommends [PHPLeague's OAuth2 client](https://github.com/thephpleague/oauth2-client) for PHP integrations. There's is sample code in the README.md, but basically

 - Include the relevant Client Key and Secret in the `$Provider` element, i.e.

    ~~~~
      $provider = new \League\OAuth2\Client\Provider\GenericProvider([
      'clientId'                => 'fake-id-741648bdf4e3ffc8e1e3607898e16870',
      'clientSecret'            => 'fake-secret-548755e9e3717974f7a378cd25b24e85',
      'redirectUri'             => 'https://app.example.com/callback.php',
      'urlAuthorize'            => 'https://oauth.server.com/oauth2/authz/',
      'urlAccessToken'          => 'https://oauth.server.com/oauth2/access/',
      'urlResourceOwnerDetails' => 'https://oauth.server.com/userinfo/',
    ]);
    ~~~~

 - Make the call to getAuthorizationURL() with the $Provider, which will redirect the user to the login page, and return back to your code with `?code=[stuff]`. To include OIDC support, make sure you specify the OIDC scope (as well as any [others](https://openid.net/specs/openid-connect-core-1_0.html#ScopeClaims) that you care about):

    ~~~~
    $options = [
      'scope' => ['openid profile email']
    ];
    ~~~~

   and call `getAuthorizationUrl` with the $options array, like:

    ~~~~
    $authorizationUrl = $provider->getAuthorizationUrl($options);
    ~~~~

 - You can then use the value in `$_GET['code']` to get an access token from the identity provider, like:

    ~~~~
    $accessToken = $provider->getAccessToken('authorization_code', [
      'code' => $_GET['code']
    ]);
    ~~~~

 - using that token you'll be able to use `getAuthenticatedRequest`'s along with the `$accessToken` as needed if you need to pull something from a secured API on the identity server:

    ~~~~
    $request = $provider->getAuthenticatedRequest(
      'GET',
      'https://oauth.server.com/some-api-endpoint',
      $accessToken
    );
    $entitlementResponse = $provider->getParsedResponse($request);
    ~~~~~

 - you can also just user information about the `$resourceOwner` that is returned (because of the scopes that we chose:

    ~~~~
    $resourceOwner = $provider->getResourceOwner($accessToken)->toArray();
    ~~~~

The access token has expiry information, etc. and there are functions for refreshing the token as needed. All in all it's really simple to do with PHP.

I avoid JS for OAuth, but I'm pretty sure that's just a personal preference. There are [good javascript libraries](https://github.com/andreassolberg/jso) as well.

*Note: Loosely based off of [this](https://stackoverflow.com/questions/58438389/openid-connect-with-php-or-java-script/58452437#58452437) StackOverflow answer from the author.*
