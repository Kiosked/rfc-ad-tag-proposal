# RFC: `<Ad>` tag proposal
RFC: Event-driven &lt;Ad> tag to replace legacy &lt;script>'s for ad networks

## About
This proposal details plans for several ad-tech solutions to better the integration between creatives and the technology serving them, including:
 * Custom <Ad> elements to better describe an ad placement
 * New events for in-line ad scripts
 * New events for ad script tags (`<script src="">`)

A successful advertising solution is one that monetises well while minimising user annoyance. Kiosked provides a comprehensive advertising solution that makes use of many UI-based effects to present and hide ads, and these affects are based on an ability to detect (or best guess) **when an ad is ready to show**.

### Detecting when an ad has shown/rendered
Ads don't usually talk to us, so we don't have any real reliable way of detecting when they've finished loading. Most of the time network tags are *nested* and will even load non-friendly iframes in some cases, so checking media load states is unreliable at best. Listening to `<iframe>` onload events is also unreliable, because we have no idea how many iframes there actually are and the event behaves differently on different browsers ([1][1] [2][2] [3][3]). Even [Google hasn't solved this issue][4] when working with 3rd party scripts:

> ...if you have a complex DfP setup and forward certain ad requests to networks, then DfP might simply insert the tag from a network and call the renderEnded event. It will even insert an IFRAME of the expected size around the network tag.

> In such a case, the event is always triggered and it's even triggered before the network has answered. But the network might be unable to fill the slot and the ad unit remains empty.

There seems to be no current *standard* way of approaching the problem of loaded/rendered detection for an ad. This proposal discusses ways of solving this through implementation of new infrastructure for client-side ad tags.

## Structure
As there are several different ways of managing bi-directional events for ad tags, this proposal includes a number of potential solutions. Each solution follows the same basic flow of information and interaction.

Currently ad tags are created by writing a `<script>` to the DOM, which can either request a remote script or execute in-line code. Both of these options result in advertiser code running in the current frame of placement, which means that support for event callbacks should be trivial. Having certain events like when the ad has **filled**, **passed-back** or **errored** would prove to be invaluable to players that manage some kind of user-experience standard for their platforms. Let's call this link **ad-to-page**.

Network ad tags can sometimes fire visibility pixels or scripts that detect how much of the ad is in view, but depending on how and where the ad is shown, this can prove to be inaccurate. Delaying the visibility/impression tracking should be made possible for such complex integrations into advertising systems (such as Kiosked's) so that both paid ad impressions and visibility measurements are accurate. This would allow ads to be presented **in a controlled, designed manner** that would not negatively affect the advertiser or intermediaries. Providing a way to indicate to the ad when it should process it's visibility measurements could be made by sending an event to the ad placement itself. Let's call this link **page-to-ad**.

### Propagation
Because ad tags may be nested, there is a strong necessity for all tags to both support this proposal's event-driven upgrades aswell as the ability to propagate events up and down the chain of ad tags.

For instance, if we had networks A, B and C, and A had been placed on the page (with B and C being nested therein), we'd see propagation similar to this:

```
A <--> B <--> C
```

A passes upwards to B, and B to C, then C downwards to B, and B to A. A would then be the interface with the page or controlling application.

### Ad-to-page
Having events sent from the ad placement to some kind of script in the containing frame or page would allow applications to respond to what the ads are doing. Ads are an element in the UI, and because they can passback or fill, the containing application should be able to react to how the ad resolves. If an ad passes-back, an empty space is left that the application can do nothing about.

There are some events that would be trivial to implement that would allow for powerful detection of ad state:
 * "ad_fill": An ad has filled and rendered content (**visible to the user**)
 * "ad_passback": An ad was unable to fill, and passback functionality should ensue
 * "ad_error": An ad encountered a fatal error (nothing should be shown)

[1]: http://stackoverflow.com/questions/10781880/dynamically-crated-iframe-triggers-onload-event-twice
[2]: https://msdn.microsoft.com/library/hh180173(v=vs.85).aspx
[3]: https://www.experts-exchange.com/questions/21975584/IFRAME-loads-twice-on-refresh.html
[4]: http://stackoverflow.com/questions/24528797/how-to-check-if-a-dfp-ad-unit-has-content
