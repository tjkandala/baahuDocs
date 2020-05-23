---
id: ssr
title: SSR
---

## Why is there no SSR support?

### Argument

baahu was built with the SPA architecture in mind. It prioritizes [Time to Interactive](https://web.dev/interactive/) over First Contentful Paint. If an app is visible in 3 seconds, but _none_ of your interactions with it register until 10 seconds, that is a frustrating experience.

The thin client vs fat client cycle and debates have been going on for decades. See [this blog post](http://www.onstartups.com/tabid/3339/bid/161/The-Thin-Client-Thick-Client-Cycle.aspx) from FOURTEEN years ago (2006). There likely isn't a universally ideal approach web application architecture.

Read these supplements about SSR + hydration:

[By Google DevRels](https://developers.google.com/web/updates/2019/02/rendering-on-the-web#server-vs-static)

[By the CEO of vercel/author of Next.js](https://twitter.com/rauchg/status/1226353359759634432)

### Counterargument

Evidently, it _is_ possible to get extremely good performance with SSR + hydration: [read more](https://old.reddit.com/r/javascript/comments/ghfyd2/secondguessing_the_modern_web/fq8th3k/)

#### "I suppose it is tempting, if the only tool you have is a hammer, to treat everything as if it were a nail." - Abraham Maslow

When you are building a blog or e-commerce site, don't use a framework like baahu. For example, these docs are made with docusaurus, not baahu.

Regardless of which side of the debate you align with, you will benefit from Baahu's virtually [best-in-class startup metrics](/docs/performance#startup-performance-lighthouse).
