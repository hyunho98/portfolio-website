---
title: "Utilizing Delegated Types to Manage Multiple Types of Users"
publishDate: 2023-12-19 00:00:00
img: /assets/rails-logo.png
img_alt: Ruby on Rails logo
description: |
  My personal experience with using DelegatedTypes.
tags:
  - Blog
  - Rails
  - ActiveRecord
---

In ActiveRecord, the Delegated Types approach is used to simultaneously interface with two tables with separate attributes. The Delegated Types [documentation page](https://api.rubyonrails.org/classes/ActiveRecord/DelegatedType.html) poses that "pagination", or the grouping and division of code blocks, of two different tables was not possible before the inclusion of delegated types because the attributes of an instance of a superclass and its subclasses can only persist within their respective tables. Because of this, there's no way for us to easily retrieve and organize instances of subclasses without multiple queries. The addition of another subclass would only complicate this process further. Delegated types streamlines this process by allowing the superclass to share its attributes with any of its subclasses.

## Practical Usage

Using my project [AdSpace](https://github.com/hyunho98/ad-space) as an example, one way we can use delegated types is to implement different kinds of Users. In AdSpace, there are two kinds of Users. One of these is a Company that can create advertisements for a product that the other user type, the Advertiser, can then claim and run in their specific advertising market.

### Initial Setup

Since the superclass in this case is the User, we implement delegated types in the User class file.
```
# app/models/user.rb

class User < ApplicationRecord
    delegated_type :userable, types: %w[ Agency Company ], dependent: :destroy
    ...
end
```
In this one line, we declare the delegated type as userable, which then accepts the different kinds of Users as an array of strings. Dependent destroy can also be added to also destroy any Agency or Company instances that are attached to a User instance that is destroyed.
```
# app/models/userable.rb

module Userable
    extend ActiveSupport::Concern

    included do
        has_one :user, as: :userable, touch: true
        accepts_nested_attributes_for :user
    end
end
```
We then create the Userable concern that, when included in our
company.rb and agency.rb files, establishes a has_one relationship with our User class.
```
# db/schema.rb

create_table "users", force: :cascade do |t|
  t.string "username"
  t.string "password_digest"
  t.string "userable_type"
  t.integer "userable_id"
  ...
end
```
The last thing we need to do is include userable type and id columns to our users table. These are used by delegated types to pull our user type by its type and id (eg. "Company", Company.id of 1)

NOTE: In AdSpace, the attribute "username" is given to the superclass so it can be validated as present and unique across all users while an attribute like "name" is given to the user types so they can be validated and compared amongst other instances of the subclass (An Agency and Company can both be  named "brown_cow" but two Companies can't both be named "how_now".) 

### Controller Actions

Since we've established Companies and Agencies as different types of Users, we can handle any generic or shared controller actions in the Users Controller.
```
# app/controllers/users_controller.rb

def create
    company_agency = params.has_key?(:industry) ? (
        Company.new(company_params)
    ) : (
        Agency.new(agency_params)
    )

    user = User.create!({**user_params, userable: company_agency})
    session[:user_id] = user.id
    session[:user_type] = user.userable_type
    render json: user.userable, status: :created
end
```
In our create action, we create an instance of the userable depending on the received parameters (NOTE: If we add more user types, we could receive a "type" parameter and create a new type using a switch statement instead of a ternary operator.)

`Entry.create! entryable: Comment.new(content: "Hello!")`

The documentation uses this example to create a new record with a delegator and delegatee at the same time but I had difficulty implementing it in this way in the controller.

`user = User.create!({**user_params, userable: company_agency})`

I found that with strong params, using the splat operator (**) along with the userable attribute allows me to create, save, and validate the User instance in one line. This comes with the added bonus of also saving and validating company_agency.

We then save the user's id and the userable's type in the sessions hash to be used in other backend actions. Since we want to keep attributes like the password digest from being passed back into the front end, we only need to render the userable. NOTE: In AdSpace, I've opted to send back some User attributes like the username, user id, and userable_type to the frontend to filter or change views depending on the type of user (An Agency cannot reach the Ad creation page.)
```
# app/controllers/users_controller.rb

def show
    user = User.find(session[:user_id]).userable
    render json: user
end
```
In our backend, we can now query for and find a user based on the user_id stored in our sessions hash. If we wanted to search for a User amongst all Companies or Agencies without delegated types we would have to:
* Add additional validations to make sure a Company and Agency don't share the same login credentials
* Find a way for the front end to communicate to the backend what kind of User to look for (Open to user error)
* Query through all Companies or Agencies for a match
```
# app/controllers/application_controller.rb

class ApplicationController < ActionController::API
    ...
    private
    ...
    def company_only
        render json: { errors: ["You must be a company to do this"] }, status: :unauthorized unless session[:user_type] == "Company"
    end
end
```
Since we can get the user's type from the userable_type column in the User's table, it is also easier to limit controller actions to the company.
```
# app/controllers/ads_controller.rb

class AdsController < ApplicationController
    before_action :company_only, only: [:create, :destroy]
    ...
end
```
We can add a before_action line to limit the creation and destruction of ads to a Users with a userable_type of "Company." This way of using delegated_types to distinguish User type in the backend is much more elegant than the alternative of storing an attribute of the Company in the sessions hash and then running it through an if or case statement. And this is only one of the many ways we save space in our code.

### The Rest of the Code

The inclusion of Delegated Types assists us in cleaning up our code and establishing a separation of concerns in the backend and frontend. It saves us from having to write out CRUD actions for each type of User and keeps our logic in one place by limiting the required amount and complexity of fetch requests. Thanks to this, we can more easily build on top of our code using the existing framework.