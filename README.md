# Place

This is a pretty basic clone of the popular reddit experiment, r/place, that lets users place 1 pixel on a 1000x1000 grid every 5 minutes. Mine lets you place 1 pixel every half second, and is overall a lot more basic. It was still a really fun project, my biggest to date and I learned a lot.

# Features

- Zoomable and pannable 1000x1000 pixel canvas
- 32 different colors!
- Each user can place 1 pixel every half second. This value was kinda arbitrarily selected
- It updates in livetime! No refreshing needed. The latency appears to be pretty good

# Current limitations

Because this was my first real project with any kind of server side anything, there are a bunch of limitations currently

- The loading time for an initial page load is sloooow. This is because I don't blobify the data and the client needs to make a bunch of requests at once on the pageload
- The latency isn't great, because I don't use websockets or anything
- There are in theory some ratelimits due to my relatively unoptimized firebase implementation
- The pixel/second restriction is only enforced client-side
- Probably a bunch of other ones I don't know about

I want to clarify that these are all solvable problems, and probably with not a lot of work, I just wanted to get this project shipped tbh

## UI limitations

I'm not the best at frontend sometimes, and this is still only my second project in svelte. That being said, the UI is kinda bad

- It just doesn't work on mobile, and I've heard reports that it's a bit weird on firefox too.
- Even on chrome, it's a bit ugly and can sometimes break on small screens
- I didn't spend enough time on the UI...

# Summary

I learned a lot! There are still a bunch of easy improvements I could make to this (beyond just refactoring the dumpster fire that is the codebase) which I might do eventually.

But for now,
Have fun! (and please don't break it...)