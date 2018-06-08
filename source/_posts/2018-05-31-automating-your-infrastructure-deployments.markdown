---
layout: post
title: "Automating your infrastructure deployments"
category: infrastructure
comments: true
author:
  name: Federico Kauffman
  email: federico@wyeworks.com
  twitter_handle: fedekauffman
  github_handle: fedekau
  image: /images/team/federico-kauffman.jpg
  description: Software Engineer at WyeWorks. Currently working with Javascript and Ruby. Learnaholic.
---

We all know that modern applications can get very complex, very quickly. Because of that, I've been trying to add more automation to my daily life in general, as I know I’m human and will eventually mess up due to being distracted, tired, or something else.

One of the first things I did while on my automation journey was to figure out a way to automate the creation of cloud resources. In order to accomplish that, I became proficient at using Infrastructure as Code (IaC) tools such as Terraform and AWS CloudFormation, among others.

After that, the next logical step was to integrate that knowledge in infrastructure provisioning into the CI/CD pipelines, since I have (or try to have) CI/CD pipelines setup in every project I’m involved in. In this post, I will present the benefits of doing just that and will demonstrate it using a specific Terraform + CircleCI example.

<!--more-->

## Benefits of automating infrastructure

If you’re not used to the concept of **automatically deploying infrastructure**, it might sound crazy, but in my opinion the benefits far outweigh the risks. The main benefits are as follows:

- The deployment will be automatic.
- You’ll tend to be more granular when making changes.
- You’ll exercise more caution when making changes to your infrastructure and this will almost always lead to less downtime.
- You will never forget to update your branch before deploying.
- Your development and feedback loops will be shorter; you merge your changes and everything gets applied for you. No need to wait for an "Ops" team to help you out.
- If you need to perform multiple steps for a deployment, you simply merge multiple PRs, one after another.
- You can automate the testing of your infrastructure by adding some automated smoke testing following deployment (or beforehand if you use [blue/green deployment](https://martinfowler.com/bliki/BlueGreenDeployment.html)).
- You can invest more time in developing features.
- All deployments can be tracked in your pipeline.

## Risks of having automated infrastructure

As with everything in life, there are also cons to this approach. They are, however, easy to overcome with some team training or by simply adding some security mechanisms.

Let's review some of the risks:

- Your team must be aware of the automated process. If you merge a PR, it’s because you wanted to merge it. If you accidentally merge code, it will get applied. To solve this potential problem, you can add a manual confirmation of the deployment for the `production` environment.
- You might forget how to manually deploy. This is easy to solve - just manually perform the deployment process every now and then.

Now that we have seen the pros and cons of automating your infrastructure, let's go through a concrete example of how to do it.

# CI/CD with Terraform and CircleCI

I created an [example repository](https://github.com/fedekau/terraform-with-circleci-example) that includes some ideas which you can use to kickstart your project or make your current project better. The repository includes a detailed readme explaining the reasons why I made each decision.

The main purpose is to demonstrate only one of the many ways you could manage your infrastructure using an IaC tool like [Terraform](https://www.terraform.io/) and a continuous integration tool such as [CircleCI](https://circleci.com).

I have followed many of the best practices described in the book [Terraform: Up & Running](https://www.terraformupandrunning.com/) and the [CircleCI 2.0 Documentation](https://circleci.com/docs/2.0/). I will also assume you have some knowledge of these tools and [Amazon Web Services](https://aws.amazon.com) in general (though no need to be an expert).

## Suggested workflow

Assuming you have two environments, `production` and `staging`: when a new feature is requested, you branch from `staging`, commit the code and open a PR to the `staging` branch. After that, CircleCI will run two jobs, one for linting and one that will plan the changes to your `staging` infrastructure so that you can review them (see image below).

![Image of PR creation jobs](https://raw.githubusercontent.com/fedekau/terraform-with-circleci-example/staging/.images/pr.png)

Once you merge the PR, if everything goes as planned, CircleCI will run your jobs and automatically deploy your infrastructure!

![Image of jobs after staging merge](https://raw.githubusercontent.com/fedekau/terraform-with-circleci-example/staging/.images/staging-merge.png)

After you have tested your infrastructure in `staging`, you just need to open a PR from `staging` to `master` to "promote" your infrastructure to `production`.

In this case, we want someone to manually approve the release to master, so after you merge you need to tell CircleCI that it can proceed and it will deploy the infrastructure after receiving confirmation.

![Image of jobs after master merge](https://raw.githubusercontent.com/fedekau/terraform-with-circleci-example/staging/.images/master-merge.png)

## Improvements

This is a very basic workflow and there are many things that must (or can) be improved upon. Here are a couple of them:

- The most important improvement that needs to be made, in my opinion, is to add a way to test the infrastructure for each PR before merging them with `staging`. Perhaps something like the [Heroku Review Apps](https://devcenter.heroku.com/articles/github-integration-review-apps) would be best. If you use Terraform modules and you write them in a generic way, you could very easily implement infrastructure testing.
- Add tests! This is of course a must in every project. If you want quality, you need to test. You can use tools like [KitchenCI](https://kitchen.ci/) or [InSpec](https://www.inspec.io/) to accomplish this.

# Conclusions

I always like to present some final, very general takeaways in my articles, so here are a few:

- Try to automate more. This will lead your team towards efficiency and make it less error prone.
- Use IaC tools. Doing so will help with maintainability and understanding of all the pieces.
- Find ways around problems; don't let problems mold you.
