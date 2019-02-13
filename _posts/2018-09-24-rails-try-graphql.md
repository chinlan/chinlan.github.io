---
layout: post
title: "Rails: try graphql"
description: ""
categories: [rails]
tags: [graphql]
redirect_from:
  - /2018/09/24/
---
[getting started](http://graphql-ruby.org/getting_started)

[ref](http://graphql-ruby.org/queries/executing_queries.html)

[ref2](https://revunit.com/building-graphql-api-rails/)

[ref3](https://graphql.org/learn/queries/#mutations)

app/graphql/types/query_type.rb

~~~ruby

module Types
  class QueryType < Types::BaseObject
  # Add root-level fields here.
  # They will be entry points for queries on your schema.

    field :post, PostType, null: true do
      description "Find a post by ID"
      argument :id, ID, required: true
    end

    def post(id:)
      Post.find(id)
    end

    field :posts, [PostType], null: true do
      description "Find all posts"
    end

    def posts
      Post.all
    end
  end
end
~~~

app/graphql/types/post_type.rb

~~~ruby
module Types
  class PostType < Types::BaseObject
    field :id, ID, null: false
    field :title, String, null: true
    field :rating, Integer, null: true
    field :comments, [Types::CommentType], null: true
  end
end
~~~

app/graphql/types/comment_type.rb

~~~ruby
module Types
  class CommentType < Types::BaseObject
    field :id, ID, null: false
    field :body, String, null: true
  end
end

~~~

query in GraphiQL IDE like:

~~~
{
  posts {
    id
    title
    rating
    comments {
      body
    }
  }
}
~~~

~~~
{
  post(id: 3) {
    id
    title
    rating
    comments {
      body
    }
  }
}
~~~

Mutation

[ref](https://github.com/rmosolgo/graphql-ruby/blob/master/guides/schema/generators.md)

[ref](https://qiita.com/vsanna/items/031aa5a17a2f284eb65d)

If running `rails g graphql:mutation createPost` will inherit `GraphQL::Schema::RelayClassicMutation`

Revise to inherit `GraphQL::Schema::Mutation`

app/graphql/mutations/create_post.rb

~~~ruby

module Mutations
  class CreatePost < GraphQL::Schema::Mutation
    null true
    # TODO: define return fields
    # field :post, Types::PostType, null: false
    field :post, Types::PostType, null: true
    field :errors, [String], null: false

    # TODO: define arguments
    # argument :name, String, required: true
    argument :title, String, required: true
    argument :rating, Int, required: true
    # TODO: define resolve method
    # def resolve(name:)
    # { post: ... }
    # end
    def resolve(title:, rating:)
      post = Post.new(title: title, rating: rating)
      if post.save
        {
          post: post,
          errors: [],
        }
      else
        {
          post: nil,
          errors: post.errors.full_messages
        }
      end
    end
  end
end
~~~

app/graphql/mutations/mutation_type.rb
~~~ruby
class Types::MutationType < Types::BaseObject
  field :createPost, mutation: Mutations::CreatePost
end
~~~

query in GraphiQL like:
~~~
mutation {
  createPost(title: "Fried chicken", rating: 5) {
    post {
      title
      rating
    }
    errors
  }
}
~~~

app/graphql/mutations/mutation_type.rb

~~~ruby
class Types::MutationType < Types::BaseObject
  field :createPost, mutation: Mutations::CreatePost
  field :updatePost, mutation: Mutations::UpdatePost
end
~~~

app/graphql/mutations/update_post.rb

~~~ruby
module Mutations
  class UpdatePost < GraphQL::Schema::Mutation
  null false

  argument :id, ID, required: true
  argument :title, String, required: false
  argument :rating, Int, required: false

  field :post, Types::PostType, null: false
  field :errors, [String], null: false

  def resolve(id:, title: nil, rating: nil)
    post = Post.find(id)
    post.title = title if title
    post.rating = rating if rating

    if post.save
      { post: post, errors: [] }
    else
      { post: post, errors: post.errors.full_messages }
    end
  end
end
~~~

query in GraphiQL IDE:

~~~
mutation{
  updatePost(id: 1, title: "Steak"){
    post{
      title
      rating
    }
    errors
  }
}
~~~

