---
date: 2019-08-16
title: You are only paying a small part of your dev team 
aliases:
    - /blog-payback-time
---

*The software your team builds relies on a lot of code that is written by people outside your organisation. Are you paying them or thanking them even?*

Building software changes all the time, for me it started with writing desktop apps that you would put on a floppy for someone to use. More than 20 years later I am working with a team at [Virtual Vaults](https://virtualvaults.com) building SAAS by using PAAS and all the serverless goodies in Azure. Not only infrastructure 
and the delivery to the end user changed, but also the way we are able to continuously create, test and ship features with multiple people working closely together.

A big part of being able to build complex systems fast is because we can build on top of layers of abstraction. 
These days you just don’t build everything from scratch, you use services, packages. **If you would look at your entire stack, 
ever wonder how little code that is executed, is actually yours?** And how much of the external code is a library that you are paying for? 
I am betting that just like the software I am working on, most of the external software is pulled into your project as a package via NuGet or NPM. 
The majority of these packages have their source code in a [Github](https://github.com/) project and have an open source license.

**So the foundations of the software we are making money with is made available for free. Build something for free, who would do that?**

Well, there are a few different groups doing this. First of all, enthusiast that started something just to learn or make their life a bit more easy. Some of them grow their career because of the project and become trainers, speakers or specialized consultants. We also see big tech companies like Google and Microsoft pushing a lot of open source, in a lot of cases it will connect people to their platform making it a solid strategy. **But what about all the other businesses that build software, are we also giving back instead of only consuming and should we?**

Yes we should, and some do! But how can we do that, not everyone at the office will be amused if we would fill our day with hobby projects and we certainly cannot just publish all our company's source code to the world. **There are however a few options to balance it out.** 

* The first one is obvious, when finding an issue or missing a feature in something you are using and you are able to fix it, 
do it and make the pull request, don’t build your internal version and keep it to yourself. In this case there is a fair chance that your fix or addition is properly reviewed (for free). In the case the fix is not pulled in (fast enough) you can fork the project and pull it in there.

* Another option, and maybe the one that is the most comfortable one starting with is documentation. A lot of projects also accept changes in their docs, from simple things like correcting spelling or grammar to adding some examples. This is a valuable way of contributing to the community and you also get to benefit yourself if you need the documentation later. After all, we cannot always remember every little detail. If your team lacks experience contributing to open source it's easy to start here.

* You can also create a blogpost where you write down how you used some tech in your scenario or describe the process you went through to make it work at all. This might help some other poor soul out there and it can also be useful for new team members or even for your future self.

* One of the more challenging options is creating and maintaining projects that you use specifically in your software. 
This will raise some questions, after all someone is paying the bill for this coding time, "is this not our intellectual property?". 
If it is something you need to create anyway, it is generic enough that other people might be interested in to use and contribute back into, it could be really interesting. Keep in mind that in this scenario you really need awareness and full support from your company.

* You can also pay back in the simple form of a donation. If your organisation heavily relies on a particular project and you really want it to stay healthy it might be wise to donate to the project. If this sounds interesting you should check out [Github sponsors](https://github.blog/2019-05-23-announcing-github-sponsors-a-new-way-to-contribute-to-open-source/).

I wrote this because I believe that not everyone in (tech)companies is aware that their development teams and their products are powered by the vast amount of communities and heroes working on pet projects. **If we increase contributing and sharing in our cultures I believe we can achieve even more.**

<sub>* Note: There is a thing to be very aware of, don't let your project become a security issue. If you share or contribute to an open source project you are also letting the world know that you are using it. Make sure nothing gets into the libraries you use that you do not want in there! But you are already checking that with everything you are using, right?</sub>



[oscarvantol.nl](https://oscarvantol.nl) 

