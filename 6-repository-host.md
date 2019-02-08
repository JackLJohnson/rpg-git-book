# Repository hosting

This section will describe the use between hosting git repositories on-premise or in the cloud. This section is needed as it should help you determine where to host your future repositories.

## On-premise hosting

In the IBM i world, it's totally normal for people to host their code on-premise - maybe even across multiple servers. Storing multiple copies of source members across multiple libraries and systems can be dangerous as you could lose track of the latest versions. Automatically storing source in git is better because you can track different versions (and even for different systems using branching).

When using git, hosting on-premise works and some businesses do it. But of course, there should only be on master copy (the main bare repository). If the system that is hosting that master repository goes down, where are your developers going to push that code? You also take the risk that one of your developers might delete the master repository (although you hopefully have your system authorties setup correctly, for this not to happen!).

## Cloud hosted

Repositories hosted in the cloud are not only more secure, but they are also more user friendly. There are tones of benefits to hosting in the cloud:

* Easy to setup SSH keys
* Friendly UI to browse code, commit history, blame
* Easy to create and maintain branches (and pull requests)
* Easier to implement a CI/CD workflow
* Easier to clone onto different systems

I will highlight '**easier to clone onto different systems**'. If you use on-premise hosting and have multiple systems outside of your network, it's hard to clone repositories across networks. If it's hosted in the cloud, it's easy to replicate across multiple systems securely.

The real issue with cloud hosted repositories is selecting which cloud hosting to use. There are more than a few options, but the more popular hosting solutions are:

* GitHub
* BitBucket
* GitLab

Choosing one of those solutions generally comes down to a number of things:

* Price per user
* How easy their UI to use
* Which one has extra features (like issues) which might be useful to your business

This book will mostly focus on GitHub as the solution of choice. This is because:

1. A very simplistic UI for the developers (for things like searching for code)
2. Easy to create code reviews on branches
3. Easy to assign tasks and reviews to users
4. Integrates with almost every CI/CD system
5. Easy to manage permissions for users on a per team or per project basis.
