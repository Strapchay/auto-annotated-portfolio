---
type: ProjectLayout
title: Journal App js
colors: colors-a
date: '2023-11-11'
client: Side project
description: >-
  It was a great experience working on this project, felt overwhelming when
  working on it but it was amazing and specifically what i envisioned for the
  project whilst starting out with it.
featuredImage:
  type: ImageBlock
  url: /images/journal-app.png
  altText: Journal app thumbnail image
media:
  type: ImageBlock
  url: /images/journal-app.png
  altText: Project image
---
<div style="text-align: left">This project was meant to replicate notion's journal table view, which i think was aptly captured with the end result of the project. The major focus of the project is mainly about code architecture, integration with api, implementing design patterns where necessary  and building on my vanilla js skills.</div>

A lot of hurdles were faced when writing the html and styling of the project, in order to make the project as responsive as the app being replicated. And due to the dynamicity of the components in the app, it pushed my markup and styling skills to be better and more refined.  Working on a huge solo project as this also allowed me learn different concepts, some examples of which was garbage collection. I had to create adaptable classes for each component view which had a method to garbage collect when the view is disposed off. Another concept was implementing the Pub/Sub pattern due to the need for specific classes/ component views to communicate with other component views and also trigger an event based on an action on a different class, the logic for this is implemented in the `signals.js` module and this was necessary due to the lack of reactivity in vanilla JS .

Also,  i implemented the MVC pattern to separate the logic for the app, this proved to make the code more verbose to follow on the pattern but it also allowed more reasonability on the code and ease to change specific logic where necessary, since its easy to track the specific point or know the specific place a bug might have occurred.

For the backend/api, i made use of Django & Django REST Framework. Implemented a Token based login which had a 3 days expiration period and the all range of authentication flow needed for a project. I had tests for each functionalities.  The api is built on Docker and allowed learning the concept of microservices within the concept of the project and deploying with github actions.
