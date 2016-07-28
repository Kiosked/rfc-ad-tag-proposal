# RFC: <Ad> tag proposal
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


[1]: http://stackoverflow.com/questions/10781880/dynamically-crated-iframe-triggers-onload-event-twice
[2]: https://msdn.microsoft.com/library/hh180173(v=vs.85).aspx
[3]: https://www.experts-exchange.com/questions/21975584/IFRAME-loads-twice-on-refresh.html
[4]: http://stackoverflow.com/questions/24528797/how-to-check-if-a-dfp-ad-unit-has-content
