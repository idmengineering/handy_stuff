# The Future of Shibboleth SP

Two days ago the Shibboleth Consortium [quietly released](http://shibboleth.net/pipermail/announce/2021-November/000252.html) the latest version of Shibboleth Service Provider for all platforms. 

> The Shibboleth Project has released a small update to the SP software, V3.3.0, and it is now available from the [download site](http://shibboleth.net/downloads/service-provider/latest/) and [packages for supported platforms on the mirrors](http://shibboleth.net/downloads/service-provider/latest/RPMS/).

This release, while acknowledged by the Consortium as a "small update" containing mostly small fixes and library updates, as well as a "sweep of the code to add deprecation warnings to more at risk features."

was notable primarily for the fact that there's a remarkable amount of commentary regarding the future of Shibboleth SP. 

This includes commentary on the build process changing to move away from the OpenSUSE Build Service to a local, Docker-based process, which (according to the Consortium):

> is a much faster process for us but it expands and constrains what we can support at the same time. As a result, a number of older platforms for which we have been unofficially producing packages but not supporting for some years will not see further package updates starting with this release.

The older platforms that appear to no longer be officially supported include macOS and SuSE. Meanwhile new support has been added for Amazon Linux and Rocky Linux. They go on to state that CentOS will no longer be officially supported later this year. This is due to the fact that CentOS is fundamentally changing it's nature:

> Some of this is also in response to the CentOS changes coming next year, and due to CentOS 8 no longer representing a fixed OS target, we will be dropping official support for it as of the end of this year, though it's possible packages may still be produced for it in the future as part of our process.

Importantly, there's also this comment regarding the future of Shibboleth SP:

> Lastly, we want to note that this is probably the final minor version of the software in its current form and new features are unlikely. Attention will be shifting in 2022 and 2023 to redesigning the SP into a much smaller native footprint alongside a Java-based appliance that performs the work of `shibd` today. This will represent a large and likely breaking change to much of the way the SP is configured and works, so if you find that non-appealing for your needs, this is a good time to be evaluating alternatives.

## A Java-based SAML SP

While it appears that there is no concrete plan at this time for the exact nature of a future "Service Provider ver. 4", according to the design notes any such software is likely to be Java-based, so that all or most of the components that dictate the SAML- and XML-handling can be done with existing code leveraged by the Shibboleth Identity Provider.

This likely means a standalone Java appliance application that can be deployed alongside Apache, IIS, etc. in order to host the "SAML core" while minimizing the C code that's necessary for the consortium to maintain.

According to the draft [Design Notes](https://shibboleth.atlassian.net/wiki/spaces/SP3/pages/2110391683/DesignNotes):

> there are some key requirements we probably have some consensus on that a new design would have to maintain:
>  1. Support for Apache 2.4 and IIS 7+, ideally in a form that leads more easily to other options in the future.
>  2. Support for the current very general integration strategy for applications that relies on server variables or headers.
>  3. Some relatively straightforward options for clustering.

which means that the new SP is likely to be largely functional for most deployers in the future as it is today.

This represents such a massive shift that those designing and building new SAML deployments may wish to avoid new deployments of Shibboleth due to the uncertain future. 

## Choosing a SAML Stack

That said, there's effectively a *large* amount of calculus to perform when choosing the proper SAML stack, and the long-term future of a given platform is an element that's always entered into that problem. The recent changes that'll be occurring in the Shibboleth sphere should *not* be taken as the sole factor in determining whether to use or not use a given SAML stack.

Shibboleth continues to be a stable platform, in both the SP and IdP components, and we expect the new SP (when it is ultimately released) will be a good, solid choice for a service provider. It is just worth consideration of the fact that it's likely to change in the (relatively) near future, perhaps in a major way.

As always, IDM Engineering stands ready to guide our clients toward the most stable, most cost-effective, and most easily-supported SAML solution for any given SAML stack. If you're looking to deploy Shibboleth or another SAML implementation, and don't want the headache of researching the nuances of choosing which stack or library to use, [reach out to us today](https://idmengineering.com/contact/).