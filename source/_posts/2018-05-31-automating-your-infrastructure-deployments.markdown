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

We all know that modern applications can get very complex very quickly. Because of that I've been trying to add more automation to my daily life in general, I know I am human and I know I will eventually fail because I was distracted, I was tired or something else.

One of the first things I did in my automation journey was figuring out a way to automate the creation of cloud resources, to do that I became proficient at using some infrastructure as code tools like Terraform, AWS CloudFormation, and others.

After that, since I have (or try to have) CI/CD pipelines setup in every project I am involved, my next logical step was to integrate that knowledge in infrastructure provisioning into the CI/CD pipelines. In this post I will be presenting benefits of doing that and will use an specific Terraform + CircleCI example.

<!--more-->

## Benefits of having automated infrastructure

If you are not used to the concept of **Automatically deploying infrastructure** it might sound crazy, but in my opinion the benefits outweight the risks. The main benefits include:

- Your deploy will happen automatically.
- You will tend to be more granular when doing changes.
- You will be more careful when changing your infrastructure and this will almost for sure lead to less downtime.
- You will never forget to update your branch before deploying.
- Your development and feedback loop will be shorter, you merge your changes and everything gets applied for you. No need to wait for a "Ops" team to help you out.
- If you need to do multiple steps for a deployment you just merge multiple PRs one after the other.
- You can automate the testing of you infrastructure by adding some automated smoke testing after deploying (or before if you use a [blue/green deployment](https://martinfowler.com/bliki/BlueGreenDeployment.html)).
- You can invest more time in developing features.
- All deploys can be tracked in your pipeline.

## Risks of having automated infrastructure

As everything in life, there are cons to this approach, but they are easy to overcome with some team training or simply by adding some security mechanisms.

Let's see some of the risks:

- Your team must be aware of this automated process. If you merge a PR it is because you wanted to merge it. If you accidentally merge code it will get applied. You can add a manual confirmation of the deployment for the `production` environment to solve this potential problem.
- You might forget how to manually do the deploy. This is easy to solve, just manually do the deploy process every now and then.

Now that we have seen the pros and cons of automating your infrastructure let's see a concrete example of a possible way of achieving that.

# CI/CD with Terraform and CircleCI

I created an [example repository](https://github.com/fedekau/terraform-with-circleci-example) that you can use to kickstart your project or to make your current project better with some of the ideas included there. The repository includes a detailed readme explaining the reasons why I took each decision.

The main purpose is to show one of the many possible ways you could manage your infrastructure using an Infrastructure as Code (IAC) tool like [Terraform](https://www.terraform.io/) and a continuous integration tool like [CircleCI](https://circleci.com).

I have followed many of the good practices described in the book [Terraform: Up & Running](https://www.terraformupandrunning.com/) and in the [CircleCI 2.0 Documentation](https://circleci.com/docs/2.0/). I will also assume you have some knowledge about those tools and [Amazon Web Services](https://aws.amazon.com) in general (no need to be an expert).

## Suggested workflow

Assuming you have two environments, `production` and `staging`, when a new feature is requested you branch from `staging`, commit the code and open a PR to the `staging` branch. After that, CircleCI will run two jobs, one for linting and one that will plan the changes to your `staging` infrastructure so you can review them (see image below).

![Image of PR creation jobs](https://raw.githubusercontent.com/fedekau/terraform-with-circleci-example/staging/.images/pr.png)

Once you merge the PR, if everything goes as planned, CircleCI will run your jobs and will automatically deploy your infrastructure!

![Image of jobs after staging merge](https://raw.githubusercontent.com/fedekau/terraform-with-circleci-example/staging/.images/staging-merge.png)

After you have tested your infrastructure in `staging` you just need to open a PR from `staging` into `master` to "promote" you infrastructure into `production`.

In this case we want someone to manually approve the release into master, so after you merge you need to tell CircleCI that it can proceed and it will deploy the infrastructure after the confirmation.

![Image of jobs after master merge](https://raw.githubusercontent.com/fedekau/terraform-with-circleci-example/staging/.images/master-merge.png)

## Improvements

This is a very basic workflow and there are many things that must (or can) be improved. Here are some of them:

- The most important improvement that needs to be done, in my opinion, is to add a way to test the infrastructure for each PR before merging them into `staging`. Something like the [Heroku Review Apps](https://devcenter.heroku.com/articles/github-integration-review-apps). If you use Terraform modules and you write them in a generic way you could very easily implement this.
- Add tests! This is of course a must in every project, if you want quality you need testing. For this purpose you can use tools like [KitchenCI](https://kitchen.ci/) or [InSpec](https://www.inspec.io/).

# Conclusions

I always like to include some conclusion in my articles, so here are some very general conclusions that can be extracted:

- Try to automate more. This will lead your team towards efficiency and make it less error prone.
- Use infrastructure as code tools. This helps with maintainability and understanding all the pieces.
- Find ways around problems, don't let problems mold you.



