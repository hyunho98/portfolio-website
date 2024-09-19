---
title: Ad Space
publishDate: 2023-12-20 00:00:00
img: /assets/ad-space-preview.jpeg
img_alt: AI Generated billboard.
description: |
  An open market for advertisements.
tags:
  - Rails/React
  - CSS/HTML
  - REST API
git: https://github.com/hyunho98/ad-space
blog: /work/delegated-types
---
# ad-space

Welcome to Ad Space! Ad Space is an open market for product advertisements. This app offers an easy way for companies to list product ads that can then be claimed and run by advertising agencies. 

## Features
- All Users can:
  1. Create an account as either a company or agency
  2. View a list of all ads that have been posted
  3. View a product ad and its claim status
- Companies can:
  1. Create and list a product advertisement
  2. Edit or Delete any created ads
  3. View a personal profile page with a list of created ads
  4. Edit the personal profile as well as any Company specific attributes
- Agencies can:
  1. Claim any ads that do not yet belong to another Agency
  2. Unclaim any ads that can then be claimed by another Agency
  3. View a personal profile page with a list of claimed ads
  4. Edit the personal profile as well as any Agency specific attributes

## Installation

Fork and clone this repository, navigate to the directory that contains this program in your terminal then install the dependencies and migrate the database using

```bash
npm install --prefix client
```

```bash
rails db:migrate
```

#### Optional

Seed the database to simulate the app with an already-populated database using

```bash
rails db:seed
```

## Usage

Open the directory containing this program in two different terminals

One of the terminals will be used to run the rails server in the back end

```bash
rails s
```

While the other will be used to run the front end

```bash
npm start --prefix client
```

In a web browser, navigate to the local host using the http address shown in the front end terminal
(http://localhost:4000 by default)

## Extras

This app was built during my time with Flatiron School to demonstrate my creative problem solving ability through the creation of a full stack application that uses techniques not taught in the Flatiron Full Stack Engineering curriculum. My research lead me to the use of Delegated Types to manage different types of Users in the backend, a blog post explaining this can be found [here](/work/)