# SameSite Cookies and Shibboleth

[Google Chrome v.80](https://www.chromestatus.com/feature/5088147346030592) is [slated](https://www.chromestatus.com/features/schedule) to be deployed to the stable channel on February 4th, 2020. (Note: [some sources](https://www.chromium.org/updates/same-site) indicate Feb. 17th as the release target.) With this update comes a fundamental shift in the default handling of cookies within Chrome. Starting with this release, cookies will by default be treated as though they have the property `SameSite=lax`, instead of this property being unset.

The `SameSite` cookie attribute is a [IETF draft written by Google Inc.](https://tools.ietf.org/html/draft-west-first-party-cookies-07) which instructs the user-agent not to send the `SameSite` cookie during a cross-site HTTP request. The  aim of the `SameSite` property is to help prevent certain forms of cross site request forgery. Cross-site HTTP requests are those for which the top level site (i.e. that shown in an address bar) changes during navigation.

The `SameSite` attribute can take three values:

  - `strict` - only attach cookies for ‘same-site’ requests.

  - `lax` - send cookies for ‘same-site’ requests, along with ‘cross-site’ top level navigations using safe HTTP methods e.g. (GET HEAD OPTIONS TRACE).

  - `none` - send cookies for all ‘same-site’ and ‘cross-site’ requests.

The previous behavior of the Chrome browser would be equivalent to `SameSite=None`.

---

## Ramification for Shibboleth Identity Provider

Per the [Shibboleth Consortium](https://wiki.shibboleth.net/confluence/display/DEV/IdP+SameSite+Testing#IdPSameSiteTesting-Conclusion), which conducted extensive testing of the Identity Provider software:

> the IdP should continue to function when its cookies are being defaulted to `SameSite=Lax` by browsers (currently tested on Chrome 78-81 and Firefox 72 with the same-site default flags set). Typically, we have only seen the IdP itself break when the JSESSIONID is set to `SameSite=strict`, which should not happen apart from when explicitly trying to set `SameSite=none` with older versions of Safari on MacOS <=10.14 and all WebKit browsers on iOS <=12 [(Source)](https://wiki.shibboleth.net/confluence/display/DEV/IdP+SameSite+Testing#IdPSameSiteTesting-Conclusion)

They go on to list the following scenarios wherein SSO breaks, namely:

  - When using client side session storage, with `htmlLocalStorage` set to false, *HTTP-POST* SSO will not work (show login page again) with defaulted `SameSite=Lax` IdP cookies. However, when using client side session storage, with `htmlLocalStorage` set to true, and all bean references in `shibboleth.ClientStorageServices` are left as they are, HTTP-POST SSO will work with defaulted `SameSite=Lax`.

  - When using server side session storage, if either `htmlLocalStorage` or the bean references in `shibboleth.ClientStorageServices` are commented out, HTTP-POST SSO will not work (show login page again) with defaulted `SameSite=Lax`. Once again, however, when using sever side session storage, with `htmlLocalStorage` set to true, *or* all bean references in `shibboleth.ClientStorageServices` are left as they are, HTTP-POST SSO will work with defaulted `SameSite=Lax`.

Therefore, to take the relevant IdP-side steps necessary to guarantee SSO on existing installations of the IdP v3, you should enable the HTML Local Storage plugin whether you use client-side storage or server-side storage. This is achieved by setting the `idp.storage.htmlLocalStorage` property to true in `idp.properties`.

See the Shibboleth Wiki section re: [StorageConfiguration](https://wiki.shibboleth.net/confluence/display/IDP30/StorageConfiguration) for more information and the implications of this setting.

For more details on the Consortium's investigatory work related to `SameSite` please visit [this wiki page](https://wiki.shibboleth.net/confluence/display/IDP30/SameSite).

---

## Ramification for Shibboleth Service Provider

Unfortunately, as [noted](https://wiki.shibboleth.net/confluence/pages/viewpage.action?pageId=72712631) by the Shibboleth Consortium, the bulk of the issues likely to arise within service provider deployments lie solely within the application space, and are likely outside of the Shibboleth domain.

> Nor can we provide any sort of yes/no or good/bad conclusion for anybody as to whether "their system is affected". That is going to depend entirely on the individual case and the only real answer is to test. [(Source)](https://wiki.shibboleth.net/confluence/pages/viewpage.action?pageId=72712631)

Effectively, SPs should really just test their systems prior to the expected [launch](https://www.chromestatus.com/features/schedule) of the `SameSite` changes in Chrome. You can test in either Firefox or Chrome:

### Firefox

  - Enter about:config in the URL bar, Accept Risk and Continue
  - Type samesite to filter options to display: network.cookie.sameSite.laxByDefault
  - Set network.cookie.sameSite.laxByDefault to true

### Chrome

  - Enter chrome://flags in URL bar
  - Type SameSite
  - Enable “SameSite by default cookies”

If your application fails to work under the test conditions, you can adjust the relevant (blocked) cookies in order to specify the `SameSite=none` directive. None of the cookies set by Shibboleth SP *should* require this directive, however, it is quite likely that you will need to adjust cookies within your application.

In particular, the notes from the Shibboleth Consortium further state that

> ... a typical source of problems for most applications is going to be load balancer behavior. If you're using cookies for node affinity, you're going to have problems with `SameSite` unless you do something about it.

and in particular, you should adjust your load balancer to specify `SameSite=none` for these affinity cookies.
