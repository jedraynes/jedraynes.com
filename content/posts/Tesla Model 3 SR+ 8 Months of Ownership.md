---
author: "Jed Raynes"
# name: "Jed Raynes"
date: 2020-12-29
linktitle: Tesla Model 3 SR+ 8 Months of Ownership
title: Tesla Model 3 SR+ 8 Months of Ownership
type:
- post 
- posts
weight: 10
# series:
# - Hugo 101
aliases:
- /posts/tesla-model-3-sr+-8-months-of-ownership/
---

![My Car](/images/tesla.JPG)

## Introduction

---

This past year, I became the proud owner of a 2020 Tesla Model 3 SR+. Picking the Tesla was a seemingly easy decision given it was an amazing car after test driving. How much has it incrementally cost me over the ownership period? What does the data say?

## Background and Assumptions

---

### Charging

I took delivery on April 28th after ordering the car on April 26th. This post won't dive into the details about the order-to-delivery process, but it was a slight hassle to say the least. After taking delivery, I mostly supercharged the vehicle as I was splitting time between middle Nashville, TN and Cincinnati, OH. At home in Nashville, I'd rarely supercharge and mostly charge via the included trickle charger from a regular wall outlet. Yeah...the outlet you normally use to charge your phone or computer, I was also using to charge my car. Conversely, in Cincinnati, I only had access to a supercharger as my apartment at the time had only outdoor, lot parking with no outlet access. For other energy sources, I've used the destination Tesla chargers and third-party chargers (e.g., ChargePoint and Blink).

For this analysis, I have all of the expenses for supercharging and third-party chargers in actual dollars as they are charged directly to my credit card. At-home trickle charging, equated to an assumed trivial amount ,I'm guessing around $15-$20 in total over the ownership period, which isn't worth calculating.

If you're curious, the 2020 Tesla Model 3 SR+ (pre-refresh) is rated for 250 miles with an efficiency rate of 239 Wh/mi as sourced from [here](https://insideevs.com/news/381209/2020-tesla-model-3-sr-most-efficient/).

### Other Costs

For the analysis, I'll also look at other costs to get a more complete view of ownership expenses. These costs are directly tracked and include the following:

- One-time order fee,
- Various accessories,
- Premium connectivity, and
- Maintenance (tire rotation).

### Overall

It's important to note that this analysis only includes incremental costs of ownership and not the largest expense, monthly payments. Given that the cost of the car in terms of sticker price is something easily calculated and considered prior to purchase, I'm ignoring it for the purposes of this post. As of this article, my car odometer is reading 7,862 miles.

## Analysis

---

### Charging

First, let's take a look at the charging costs only. As mentioned before, these include supercharging costs plus third-party charging costs. At-home costs are not included as they are trivial for me given my circumstances. At the onset of COVID, I decided to move back home to Nashville from Cincinnati to be with family. Then, getting the car, I found myself driving back and forth between Nashville and Cincinnati for work, see friends, and eventually move-out of my apartment. Additionally, I went on a trip to Holland, MI (insanely fun place and great beaches) which meant more driving. The largest spike is from my trip from Nashville to Philadelphia. The most consistent period is from mid-October to mid-December, when I stayed temporarily in Cincinnati (and driving back to Nashville for breaks) having access to only superchargers.

Here's a mapping of the key charging expense spikes by week (to reference while viewing the Tableau visualization):

- Week of June 28: Trip from Cincinnati, OH to Holland, MI,
- Week of August 16: Trip from Nashville, TN to Philadelphia, PA, and
- Weeks of October 18 to December 13: Cincinnati, OH so no access to home charging + trips from Cincinnati, OH to Nashville before, after, and during.

### Other Costs

First, let's take a look at the non-charging costs. The section above itemizes these costs and they're fairly straightforward. One I'd like to highlight is the Jul-20 $30 accessory charge. This was for a matte black center console wrap...it didn't work out. I must lack the patience and steady hands needed to make it work and I ended up messing it up. Turned out to be a sunk cost, I ended up throwing it out and sticking to the shiny, fingerprint-magnet plastic.

### Overall

For those that want to see it all in one place, see the Tableau story below.

<iframe src="https://public.tableau.com/views/TeslaModel3SR8MonthsofOwnership/Story1?:showVizHome=no&:embed=true" height="750" width="100%" allowfullscreen="allowfullscreen" frameborder="0" scrolling+"0"></iframe>

## Conclusion

---

My YTD Dec-20 costs totaled $677.34. Remember, this doesn't include at-home charging costs so maybe the total is around $700 or so. Had I gotten another car, I'm not entirely sure if I would've spent more or less without diving into the data. Maybe that's something for another post...

---

<div id="disqus_thread"></div>
<script>
    /**
    *  RECOMMENDED CONFIGURATION VARIABLES: EDIT AND UNCOMMENT THE SECTION BELOW TO INSERT DYNAMIC VALUES FROM YOUR PLATFORM OR CMS.
    *  LEARN WHY DEFINING THESE VARIABLES IS IMPORTANT: https://disqus.com/admin/universalcode/#configuration-variables    */
    /*
    var disqus_config = function () {
    this.page.url = PAGE_URL;  // Replace PAGE_URL with your page's canonical URL variable
    this.page.identifier = PAGE_IDENTIFIER; // Replace PAGE_IDENTIFIER with your page's unique identifier variable
    };
    */
    (function() { // DON'T EDIT BELOW THIS LINE
    var d = document, s = d.createElement('script');
    s.src = 'https://jedraynes.disqus.com/embed.js';
    s.setAttribute('data-timestamp', +new Date());
    (d.head || d.body).appendChild(s);
    })();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>