# Service Objects
## Where to put them?
So our app folder looks something like this:
```
.
├── app
│   ├── channels
│   └── controllers
│   └── jobs
│   └── mailers
│   └── models
│   └── views
```
But none of these seem like a good fit...
We also have the `lib/` folder, but this also does not feel right.
Everything in the lib folder should be code that can be used in a different project and could technically become a gem.

So we have to create our own folder like `app/services/`.
Some people seem to forget that it is perfectly fine to add folders to your project!

## The Base Class
Now that we have our `app/services/` folder we can create our base class. Every service
will inherit from this class, just like with `ApplicationRecord` and `ApplicationController`.

So we create our `app/services/application_service.rb`
```ruby
class ApplicationService
  def self.execute(*args, &block)
    new(*args, &block).execute
  end
end
```

## The Structure
Let's say we have a model `Article` and a controller `ArticlesController`.
Let's create our create service.

First we will create a subfolder `app/services/article/`.
Next we create our `Article::CreateService`
```ruby
class Article::CreateService < ApplicationService
  attr_accessor :article

  def initialize(params)
    @article = Article.new(params)
  end

  def execute
    ActiveRecord::Base.transaction do
      article.save!
      create_activiy!
      notify_subscripers!
    end

    article
  end

  private

  def create_activiy!
    # some code to create an activiy
  end

  def notify_subscripers!
    # some code to send some mails
  end
end
```

So now in our `ArticlesController` we can do:
```ruby
def create
  @article = Article::CreateService.execute(article_params)
  if @article.persisted?
    redirect_to @article
  else
    render :new
  end
end
```
Because of our `ApplicationService` we can call `.execute` on the service class and this will initialize and execute our service!

Our service folder should look like:
```
.
├── services
│   │
│   ├── article
│   │    └── create_service.rb
│   │    └── destroy_service.rb
│   │    └── update_service.rb
│   │
│   ├── user
│   │    └── create_service.rb
│   │    └── subscribe_service.rb
│   │    └── follow_service.rb
│   │
│   └── application_service.rb
```
Also the idea is that a service inside the `articles` folder always returns an article.

## The other controller actions:
### Destroy action
```ruby
def destroy
  article = Article.find(params[:id])
  Article::DestroyService.execute(article)
end
```
or
```ruby
def destroy
  @article = Article.find(params[:id])
  @article = Article::DestroyService.execute(@article)
  if @article.destroyed?
    # yay!
  else
    # oh no!
  end
end
```

### update action
```ruby
def update
  @article = Article.find(params[:id])
  @article = Article::UpdateService.execute(@article, article_params)
  if @article.errors.empty?
    redirect_to @article
  else
    render :edit
  end
end
```
