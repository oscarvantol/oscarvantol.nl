---
date: 2020-09-24
title: The state of the cloud, serverless
aliases:
    - /blog-serverless
---

It looks like the world is changing faster than ever before, disrupting start-ups come from nowhere and challenge big companies. Technology provides connectivity as never before and the possibilities seem endless. This year the world got challenged like never before and we are using technology to overcome issues in our daily life that would not have been possible 10 or even 5 years ago. In the past decade I have talked to a lot of development teams, CTOs and decision makers about the cloud, mainly Azure. From just grasping the concepts to helping architect specific solutions. If I look back, the term that was defining was 'As A Service'. 

## As a service

Most of the discussions were about scale and cost control, for start-ups it was about not having to make investments upfront. In the end it is about not owning a big part of the stack. Not owning something also meant having a different level of control. To choose your comfort zone we always had a good analogy, Pizza as a Service. Still today I(nfrastructure)aaS, P(latform)aaS and S(oftware)aaS exist side by side for a reason. With IaaS you still get the maximum control over your environments, with PaaS you get to focus on your own code and with SaaS you are just a consumer.

## Evolution
 Within those groups there has been a lot of evolution though. The most difficult to place for me are Containers, I would arguably place them as the pinnacle of IaaS because of the amount of control you have. Platform as a Service was always about not caring about anything but your own code, providing out of the box scale and cost control options. The ultimate version of this is SERVERLESS. So is this just PaaS rebranded? Well, there are definitely some services that have been around for some years but suddenly showed up in lists of Serverless offerings. Some of these services did get some new tiers that would classify better. We can discuss that Serverless is a horrible name but most offerings will scale back to zero hosts and cost nothing for those seconds. Most people would associate the term with compute and F(unctions)aaS but we should also count several storage and database solutions as Serverless.

## Cold start

Every application needs some start-up time and the so called 'cold-start' is nothing new. After deploying or even after a certain idle time most applications need to spin up for a short amount of time. Also when you are scaling out the new hosts need time to become responsive. So why is this different? As mentioned before in Serverless you will go back to zero hosts if there are no requests. The idea is that you run a tiny service on a part of infrastructure that is assigned on the fly and not booked ahead. For asynchronous tasks this won't be a big issue but if it is a user doing an http call a 10 second wait is not the best experience. Serverless offerings are adding all kinds of control and tricks to overcome this issue like predictive scaling and giving the owner control to have a minimum amount of hosts running. Is this still Serverless then? The real answer is: Do you care? The real power of Serverless compute is in something else.

## Vendor lock in

This power that I mentioned is also an interesting subject. The power is in abstraction, everything you should not care too much about is not in the way. But this is 1-to-1 connected to the heavily discussed 'drawback' of Serverless that is called vendor-lockin. This means that if you start using serverless you will be heavily tied into the specific cloud service and the SDK's. But these abstractions that they provide actually help you to focus on your code, you don't have to write listeners or pull in all kinds of packages to make something work. Your code starts exactly where you are adding value, right after the framework gave you an event with all the context you need. The logic that you write is still yours and of course moving to a different cloud will be some work but all the stuff that creates the 'vendor lock in' you did not write and you do not have to maintain!

## Now what?

Serverless is here to stay, we will definitely see more evolution and maybe different naming in the future. It is not that you should start using Serverless for everything but it is a mature option to consider. Picking the right tool for the job is really important, this means that you should have multiple tools in your belt and you should know how they work. Thanks to the cloud we moved far away from having just a database and a webserver as tools to create.


### Resources:

For #ServerlessSeptember we created and collected a lot of content. It can be found [here](https://www.betabit.nl/nl/serverless-september).

---

[oscarvantol.nl](https://oscarvantol.nl) 