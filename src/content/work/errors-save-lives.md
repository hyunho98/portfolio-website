---
title: "Rails and ActiveRecord: Errors Save Lives"
publishDate: 2023-11-11 00:00:00
img: /assets/rails-logo.png
img_alt: Ruby on Rails logo
description: |
  Use errors to your advantage.
tags:
  - Blog
  - Rails
  - ActiveRecord
---

## I thought errors were the enemy?

Errors are often unwelcome and sometimes confusing elements that can throw a wrench into a seemingly logically sound program. Other times, error messages are succinct and easy to address. Though regardless of the type of error, we as programmers see them as a problem that is out of our control. What if that wasn't the case? What if we made our own errors?

### Why custom errors?

Custom errors in rails can be used by programmers to communicate to other programmers **and** users that something has gone awry. A custom error can also be more specific and concise, making diagnosis and repair much easier than it would be otherwise. 


## Errors in ActiveRecord

More often than not an error in ActiveRecord is destructive to the user experience and could point to a problem without showing the actual cause of the problem. For example, ActiveRecord may throw a RecordInvalid error not because something is wrong with the code, but because the user tried to submit a form with invalid values (think about password fields requiring characters or length.)

![ActiveRecord error page](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/tsxil6wu9chve1wrptst.png)

In the case of an ActiveRecord error, the output would be an error page that is confusing to a user without coding knowledge.

### rescue_from

What we can do to avoid this situation is to use rescue_from in a controller to quite literally rescue the program from an error that may be thrown by the ActiveRecord model. A rescue_from call may look something like this.

`rescue_from ActiveRecord::RecordInvalid, with: :record_invalid`

Every method call that may result in a RecordInvalid error will now be caught and then deferred to a method (:record_invalid) that we have defined elsewhere in the controller. Since this method won't be accessed outside of the class, we can make it private.

```
private

def record_invalid(invalid)
   render json: { errors: invalid.record.errors.full_messages }, status: :unprocessable_entity
end
```

With this bit of code, instead of a confusing page view, we now get a json response containing an array of error messages and a 422 status code (unprocessable entity) from the server. Since this specific error is caused by validations, through the magic of ActiveRecord, we get an array of messages that tell us which validations failed. We could also return our own messages by replacing the code within the brackets.

```
...
def not_found
   render json: { errors: ["This thing was not found in that place"] }, status: :not_found
end
```

We've changed a couple of things between the last example and this one but the idea stays the same; instead of an error page, we get an error status code and an array of messages. However, this time, the error message is something that we can set ourselves. For example, if we have a program that works off of a database of apples in an apple tree and someone tries to search for an avocado, we can return this error message ["This [AVOCADO] was not found in the [APPLE TREE]."] and/or any other relevant error messages. 

### How does this help the user?

After the view receives the json response, it can then be used to communicate to the user that something has gone wrong. Instead of pulling the user out of the app to show them an error screen, since we have an array of error messages, we can just streamline these messages and display them in a way that the user can understand.

![Displaying errors to the user](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/wgk9rxianpjm78lg04qw.png)

Much easier to understand than the error page that would display otherwise.

### Using error messages as a programmer

In a development environment, code will often be run and rerun many times by multiple different programmers. So being able to communicate when and where something goes wrong is important. So just like the user, a programmer can use the error messages to either 
  1. Test and isolate certain errors to make sure the user cannot do anything that should not be allowed.
  2. Test the application and receive accurate feedback on what needs to be fixed.

## Usage outside of ActiveRecord

We've been talking about rails this entire time but how can we practice using custom errors outside of Ruby and ActiveRecord? Most coding languages also have custom error implementation. It's very common for languages to have a way to 'throw' and 'catch' errors because that's how errors are usually dealt with internally. That's why an understanding bugs and debugging is a supremely important skill for developers. Using this skill to create meaningful custom errors only further adds to an app.

### Building habits

So how can you or I start coding with custom errors in mind? Without inflating our code with multiple error throws that don't make much sense, how can we apply this mode of thinking? One good place to start is to visualize how the code will run as you're building it. Is there a component of code that has many moving parts? We should throw an error at important junctions of that component so the problem is easier to diagnose. What if we threw error messages in sections of the program that will frequently interface with other bits of code? Although understanding good practice may be difficult at first, constantly asking questions and testing ideas may help build key habits that could be used throughout an entire career.
 
