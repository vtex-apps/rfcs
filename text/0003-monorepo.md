- Start Date: 2020-07-03
- Implementation PR: (leave this empty)
- Store Discussion Issue: (leave this empty)

# Summary

This RFC proposes a solution to create a single repository to all apps of the Store Framework. 

# Basic example

We want to do something similar to what the following frameworks do:

* [Angular](https://github.com/angular/angular)
* [React](https://github.com/facebook/react)
* [Material UI](https://github.com/mui-org/material-ui)

They keep all packages in the same repository.

# Motivation

When the Store Framework started it had only one team united team with all the people in the same place and it had not so many apps (around 12 apps) to this team to support, so it was easy to control who was doing what and if it was being done respecting the standards of the framework, but today we have around 73 repositories that 6-7 different teams of VTEX support and on top of it now we have an open-source community that uses those apps and even contribute to it. With all this code base it became complicated to update tooling, fix technical depts, and even update these apps to respect best practices of code or API. With a monorepo we can get closer to the idea of creating a product that has the same ideas across all blocks.

Pain points of keeping 73 repos up to date:

* Lint rules always get out of date in some forgotten apps
* Github templates have the same problem of lint rules
* API patterns and best practices to develop a new block
* Avoid duplication of logic
* Test setup
* Misleading open-source flow to create issues


# Detailed design

Since this is not a feature but an organizational change that might help everyone that works in Store Framework and the open-source community this we don't have much to add to this section, but we expect to fix all the problems listed above and that the monorepo help us to make it easier the following topics:

* Share code to make testing easier because we wouldn't need to mock all IO dependencies
* Create a calendar of support for majors, like [NodeJS](https://nodejs.org/en/about/releases/)
* Plan new majors for the whole framework
* Create migration guides to each change of these majors
* Predictable manual updates to our clients to get a better store

# Drawbacks

Why should we *not* do this? Please consider:

- To launch a new major we would need to launch a major of all apps or to put the legacy major in a separate folder and to add the current in the place of the current major (similar with what the [search-result](https://github.com/vtex-apps/seach-result) does)
- Older major versions that were being kept in different branches would have to make the same strategy to continue to exist
- We would have to update the bot that launches a new version of the apps
- The repository would be **incredibly** big

# Alternatives

If we have a tool that could make the best of two sides, creating a monorepo or using multiple repositories to each app. It would be something easy for the open-source community to adopt and easy for all of us that work on the Store Framework to communicate about API changes and to share code, but this would lead to a lot of work of VTEX too.

# Adoption strategy

We would have to adopt this in phases:

1. Prepare the bot to launch new versions in this repository
2. Move at least two Store Framework repositories, setup all the tooling considering multiple apps, make sure that everything is working and that this has no implication in production (without sharing code until phase 6)
3. Migrate all apps of the Store Framework team and stay this way for a while as a trial phase
4. Migrate one team at a time to the monorepo
  - The teams involved into this are: Checkout UI, Search, Post Front and Login
5. Think about how we could share code between apps
6. Start sharing code
7. Plan next majors how would this work

# How we teach this

We should archive all current repositories and add something in the readme to point to the monorepo.

# Unresolved questions

These unresolved questions should have a RFC for it else:

* How we should make majors and how to make it predictable to the clients when they need to upgrade there store and what would be the benefits of doing it.

* How we should share code
