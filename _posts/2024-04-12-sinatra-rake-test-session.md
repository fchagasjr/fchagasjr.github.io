---
layout: default
title: Setup a Session in Sinatra for RakeTest
---
Have you ever estabilish a mock section for testing a Sinatra application? There is a great chance you will never do it. To be honest, I would think the same way and here I'm writing about it. So, you never know...

My project is built using Sinatra and, to be fair, it became bigger than I originally expected. I was using RakeTest for testing then eventually I needed to test some functionalities that required a user to be logged in or logged out.

That was easy, I thought, I just need to create reproduce on the test environment what the application uses in production to assure the user is logged, let's use section!

Easier said than done, I came to realize that doing that on Sinatra & RakeTest is not really intuitive.
