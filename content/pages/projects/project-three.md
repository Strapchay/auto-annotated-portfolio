---
type: ProjectLayout
title: Todo App JS
colors: colors-a
date: '2023-10-10'
client: Side Project
description: >-
  It was great working on this project, its design and implementations were
  original but did draw some concepts from google's notes app. It was fun
  working on its responsivity and its offline capabilities even though its not a
  PWA.
featuredImage:
  type: ImageBlock
  url: /images/todo-app.png
  altText: Todo app thumbnail image
media:
  type: ImageBlock
  url: /images/todo-app.png
  altText: Project image
---
This app is meant to replicate a todo app which is responsive and has offline capability. The project focused on code architecture, integration with api, implementing code sync with remote data and offline usability and building on my vanilla js skills.

The project involved implementing garbage collection on each component classes, used the MVC pattern for code architecture and logic separation. Implemented a drag functionality to switch todo list ordering.

For the backend/api, i made use of Django & Django REST Framework. Implemented a Token based login which had a 3 days expiration period and the all range of authentication flow needed for a project. I had tests for each functionalities.  The api is built on Docker and allowed learning the concept of microservices within the concept of the project and deploying with github actions.
Github repo FE: [todo\_app](https://github.com/Strapchay/todo_app)
Github repo BE: [todo\_api](https://github.com/Strapchay/todo_api) 
BE Doc: [todo\_api\_doc](https://appistodo.ddns.net/api/docs)
