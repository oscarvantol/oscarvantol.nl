---
date: 2020-02-20
title: Writing an Azure DevOps Pipeline extension
aliases:
    - /blog-azure-piplines
---
 *How I ended up having a Saturday night date with the Azure DevOps marketplace.*

**'At some moment we should really fix this!'** Is this something you and your team sometimes say? We had one of those, we are building really cool software at Virtual Vaults using Dotnet Core, Angular and all the Azure toys. We use Azure DevOps and have a lot of common NuGet packages that we share internally using the Artefacts Feed. 
Everyone who updates a package sometimes will at one point forget to bump the version before pushing and we will have a discussion about how we should branch and version the packages. But in the end we will complete the task at hand and move on. Only until the next time.

## Enough, let's do this

At one point I was sick of this. "let's settle this once and for all". After some quick conversations with strong opinioned team members we decided what would be a good strategy. We wanted to have the automated builds pushing out packages without bumping the version by hand. Also if the CI build would be triggered from any other branch than master, the version should be marked as pre-release. This sounded pretty straight forward but I could not make it work with any of the standard options I had in the pipeline. I found a lot of extensions in the [marketplace](https://marketplace.visualstudio.com) that helped me a bit but the yaml pipeline file quickly became a large messy script. Being a developer I decided to write my own extension for a custom build task.

## Building an extension

Because I created a simple build task before I knew it would be pretty straight forward. This was both true and untrue, the [documentation](https://docs.microsoft.com/en-us/azure/devops/extend/get-started/node?view=azure-devops) is really good and I had my basic logic running very quickly using some typescript (which is awesome). But to get the thing properly packaged including what was needed from the node_modules folder had me googling for a while. Lots of people writing extensions have their code on GitHub and this helped me figure out what I was doing wrong.

## Using the extension

The next step was uploading the extension to the marketplace and share it privately with my organisation. I was ready to start testing it in the context of a build. I struggled a bit with setting up the permissions correctly because I needed to use the DevOps api inside the task but it worked like designed and I had an example pipeline running. 

Finally I added the custom task to the pipeline of the package I was actually working on during 'normal hours' and told everyone to have fun with it.

## Done, or maybe not?

Noooooo! Not done, I needed to write some documentation on our wiki or at least some comments in the yaml file because otherwise this will definitely get lost as some point. And were to store the code for the extension. I thought to myself **I am doing this anyway, let's do it right!** So I pushed to [sourcecode](https://github.com/oscarvantol/azure-pipelines-version-increment) to GitHub and wrote some minimal documentation. Then I wanted to make the [extension](https://marketplace.visualstudio.com/items?itemName=ovantol.version-increment) publicly available on the marketplace. This meant some more reading and adding information and metadata to my profile and the package. Making this public also made me add some extra documentation, a yaml code example showing how to use the extension and last but not least, an icon. 


**What was the benefit of adding these extra steps?**
* I forced myself to write some documentation and clean up.
* This might be of help for someone else writing an extension.
* I might help my future self in need of something similar.
* Someone else might actually use my extension😱

 ![Marketplace](assets/blog-ape/extension.png "Azure DevOps Marketplace")

[oscarvantol.nl](https://oscarvantol.nl)
