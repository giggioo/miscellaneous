See `git-github-reference.pdf`. It contains all the basic/intermediate informations that you need.

Feel free to add here any useful advanced command you can think of git-related.

It's possible to create new branches and moving onto them by using `git checkout -b branch-name`.

To just create a new branch simply type `git branch branch-name`. This branch is going to be inserted in the current commit we're working on. We can switch on this new branch with `git checkout branch-name`.

> When merging, the results will go on to the branch that we're currently working on!

## Branching Strategies (from easiest to hardest)

##### Trunk Based

We have a single branch (probably is gonna be master) and everyone is cloning that only branch, making changes and pushing them into the master as soon as they are done. 

No other branches, no pull request needed, no merge.

That's the **easiest approach** but the **hardest to maintain**! 

The team must be experienced and really needs to follow TDD (Test-Driven-Development) writing tests to run in advance before deploying code to the mainline.

In this case of development is indispensable to use feature toggles. 
If we are pushing frequently on the mainline (very likely if we are using this development strategy) we need to make sure not only that those changes tested, valid and working but also that half-done features are not visible to users.

Feature toggles are quite handy: We can disable features even though they are deployed to production.

##### Feature Branching

Whenever we wanna work on something we create a new branch.

Features are typically split into very small chunks (so that it doesn't take too much time to write code) and once we're finished we create a pull request, review it, possibly run some automation on it and once everything is fine we merge that branch back onto the mainline.

It's very important to work on small chunks so we have short delivery cycles. Every feature should be accomplished within a day or two or even a few hours.

That comports that we do continuous delivery. 

Features toggles are not essentials as in trunk based development but still very useful.

What is instead mandatory are pull requests. 

> We create pull requests not only when we are finished working on a feature but whenever we need feedback, even in the middle of development.

##### Forking Strategy

It's similar to feature branching but instead of creating branches we fork the whole repository, work on some part of it and then create a pull request to the mainline. When we fork we are free to branch whenever we want and then create some pull request for the fork-branch and the fork-master to be merged into the mainline. That's the most common strategy when contributing to open source. 

The major advantage of forking repos over branching is that we don't need to deal with permissions. 

##### Release Branching

Unlike feature branches, release branches tend to be longer lasting (weeks, months). 

Different teams might be working on different releases so we have a branch for every release and a team working onto each release.

On top of that we could have some other branches such as hotfixes and whenever we merge an hotfix we also need to make sure that all the currently open branches are also updated pulling the changes. 

Also, each team is independent and we don't see the changes until one of the team is finished. 

This comports <u>low frequency deployments</u> and <u>no continuous integration</u>. It takes a lot of time until changes are merged back to the mainline and it takes even more time until different teams get to integrate their code with other people code.

So why would we use this strategy?

We might found it useful if we were software vendor that needs to support multiple versions. 

This is safer because everyone is working on his release branch but its more complicated to the point that you might need a dedicated figure/team just to merge code and resolve conflicts.

##### Git Flow and Environment Branches

i won't even discuss about those. They're way too complicated and honestly they don't even make sense. 

Okay maybe i'll try to explain git flow just because you could find it into big old-fashioned companies. 

- Master branch
  
  - This branch is used for production releases.
  
  - We can push hotfixes directly into the master.

- Develop branch
  
  - This branch contains **stable features** for the next release.
  
  - From here we can create as many feature branch as we like.

- Feature branch 
  
  - We can **integrate** features **into the develop branch** when the feature is **stable** and **tested**.
  
  - When we make changes into the develop branch by integrating a feature-1, those changes must be merged back into other feature branches (feature 2,3,..) 

- Release branch
  
  - Usually we also find this kind of branch that are used to isolate and stabilize the release. 
  
  - From develop branch we create a release.
    
    - We'll most likely find some bugs and fix them in the release branch, those fix follow the same procedure as the develop branch: When we make changes to the release branch by fixing a bug other features must merge it.
  
  - From the release branch when we corrected every bug we can merge into master.

### Real programmers commit to master

While adding complexity to the git structure might seem a good idea to maintain things organized it actually does quite the opposite. The more the project become the more messy it gets. That's why we should follow the trunk based strategy.

We have the master branch and we commit to it. Committing to master doesn't mean that we cannot create branches!

We create branches for pull request but the idea is that those branches are short-lived. 

Eventually we are going to create a release branch, we are ready for release 1.0 while the rest of the team keeps on going in the master branch.

Let's say that while testing around the release we found an error with the release going out. We need to fix this bug but where do we do it? In the release branch?...

...Wrong!  The bug must be fixed in master, and then we cherrypick that commit to the release.  

Why? Fixing bugs in a separate branch is the risk for regressions. Regressions bugs are embarrassing. 

If we fix the bug into the release branch and then we put it into production, in the next release we will get the same bug because it wasn't merged back into master.

If we fix bug in master this will never be an issue.

If we found a production bug instead it's the same thing. We create a branch from the commit that is currently in production, again we fix the bug in trunk, cherrypicking it into the hotfix branch and we deploy it into production. 

<img title="" src="file:///Users/giggio/Documents/UNI/img/miscellaneous/Immagine%2010-04-24%20-%2000.00.jpg" alt="" width="763" data-align="center">

#### Benefits

- Minimize merge work/conflicts
  
  - Everybody is committing into master! 

- Encourages making small batches/chunks of work
  
  - If there's a team of 10 developers working continuously locally for for a long time with your commits the changes are gonna starts to pile up locally!

- Encourages refactoring

- Always release ready

- Better delivery  throughput into production

### Nice. But how do we do it?

If we just committing code into master won't that break a lot of stuff?

- As we mentioned, we need small incremental batches of work. Every commit should be releasable.

- It will probably take more than a day to finish a feature but this commits will go into production anyways so we need ways to "hide" this work until is 100% done so we use **feature toggles**.

- We need to enforcing quality in our changes and as soon as possible, as much as possible; before that the change actually ends up in master.
  
  - We can do that by running **verification builds**, running **unit tests**, doing **code review** etc.

- Sooner or later, bad code will be deployed into production. When we deploy into production we wanna make sure that if you're deploying bad code it will impact as few use as possible.
  
  - So, instead of deploying a new feature to everyone across all your environment, try to minimize it: Do it incrementally. We're gonna talk about some deployment techniques.

#### Feature Toggles

The concept is very easy. Let's say we have a webpage and we have a really boring grid to display some content. We are working towards changing that to a graph but that is going to take a day or perhaps longer. So we create a feature toggle! Sounds intimidating? Consider it as an if-else statement to keep it simple.

If the feature toggle is turned off the graph is there but is not visible to the end user.

```cpp
if(toggle.FeatureEnabled){
    // Feature enabled
}
else{
   // Feature disabled 
}
```

For more informations on feature toggles check this article from Martin Fowler: [Feature Toggles aka Feature Flags](https://martinfowler.com/articles/feature-toggles.html)

##### Where should I put feature toggles?

Let's pretend we have an UI $\to$ that calls an $\to$ API $\to$ then we have some $\to$ Logic $\to$ and then we have some $\to$ Storage

I'm working on the new graph i just talked about. Then obviously i would add it in my UI because that's where i'm changing stuff. Is that enough or should i put it even somewhere else?

Maybe the UI calls a new endpoint on my API.

> An endpoint is an access point into my API that can be reached through HTTP requests in order to perform specific operations such as sending and retrieving data. Endpoints can include operations like "retrieve a list of users" or "delete/update this specific user"

Okay but if the UI isn't  visible it won't never call that endpoint right? Well, no. That endpoint will actually exist there. It will be exposed. Even if i'm not calling those endpoints, someone else might. We need to think about security as well! We want to put this endpoint as high as possible but don't forget about those publicly exposed layers.

We're considering just a single change (table $\to$ graph), but usually we move and remove and add more stuff from a single webpage. Do i need 15 if else statements in my web page? Probably that's not a good idea.

The best solution here is to just keep the old page and keep working on this new page in parallel with the existing one. 

We use routing so that the original page route will just keep working and all user will be directed to the existing page and you as a team can start working on the new page. 

Of course there will be some duplicated code there because you are copying the old page and start working on it but that's fine because its for a short period of time. 

But now we can even add feature toggles to the mix! We can enable the new page for some pilot customer to get feedback.

#### Implementing feature toggles

A lot of companies build them themselves and there are several different frameworks around [dotnet](https://dotnet.microsoft.com/en-us/learn/dotnet/what-is-dotnet)):

- NFeature

- FeatureToggle

- FeatureSwitcher

- ...

Other solutions are SaaS services:

- LaunchDarkly

- Rollout.io

- Split.io

Instead of calling our own code to ask if the feature is available for the specific user we'll call the service instead.
They also implement some useful functionalities instead of just answering yes or no!




