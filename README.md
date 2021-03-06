# Disco

:fire: Recommendations for Ruby and Rails using collaborative filtering

- Supports user-based and item-based recommendations
- Works with explicit and implicit feedback
- Uses high-performance matrix factorization

[![Build Status](https://travis-ci.org/ankane/disco.svg?branch=master)](https://travis-ci.org/ankane/disco)

## Installation

Add this line to your application’s Gemfile:

```ruby
gem 'disco'
```

## Getting Started

Create a recommender

```ruby
recommender = Disco::Recommender.new
```

If users rate items directly, this is known as explicit feedback. Fit the recommender with:

```ruby
recommender.fit([
  {user_id: 1, item_id: 1, rating: 5},
  {user_id: 2, item_id: 1, rating: 3}
])
```

> IDs can be integers, strings, or any other data type

If users don’t rate items directly (for instance, they’re purchasing items or reading posts), this is known as implicit feedback. Leave out the rating, or use a value like number of purchases, number of page views, or time spent on page:

```ruby
recommender.fit([
  {user_id: 1, item_id: 1, value: 1},
  {user_id: 2, item_id: 1, value: 1}
])
```

> Use `value` instead of rating for implicit feedback

Get user-based (user-item) recommendations - “users like you also liked”

```ruby
recommender.user_recs(user_id)
```

Get item-based (item-item) recommendations - “users who liked this item also liked”

```ruby
recommender.item_recs(item_id)
```

Use the `count` option to specify the number of recommendations (default is 5)

```ruby
recommender.user_recs(user_id, count: 3)
```

Get predicted ratings for specific items

```ruby
recommender.user_recs(user_id, item_ids: [1, 2, 3])
```

Get similar users

```ruby
recommender.similar_users(user_id)
```

## Examples

### MovieLens

Load the data

```ruby
data = Disco.load_movielens
```

Create a recommender and get similar movies

```ruby
recommender = Disco::Recommender.new(factors: 20)
recommender.fit(data)
recommender.item_recs("Star Wars (1977)")
```

### Ahoy

[Ahoy](https://github.com/ankane/ahoy) is a great source for implicit feedback

```ruby
views = Ahoy::Event.
  where(name: "Viewed post").
  group(:user_id, "properties->>'post_id'"). # postgres syntax
  count

data =
  views.map do |(user_id, post_id), count|
    {
      user_id: user_id,
      item_id: post_id,
      value: count
    }
  end
```

Create a recommender and get recommended posts for a user

```ruby
recommender = Disco::Recommender.new
recommender.fit(data)
recommender.user_recs(current_user.id)
```

## Storing Recommendations

Disco makes it easy to store recommendations in Rails.

```sh
rails generate disco:recommendation
rails db:migrate
```

For user-based recommendations, use:

```ruby
class User < ApplicationRecord
  has_recommended :products
end
```

> Change `:products` to match the model you’re recommending

Save recommendations

```ruby
User.find_each do |user|
  recs = recommender.user_recs(user.id)
  user.update_recommended_products(recs)
end
```

Get recommendations

```ruby
user.recommended_products
```

For item-based recommendations, use:

```ruby
class Product < ApplicationRecord
  has_recommended :products
end
```

Specify multiple types of recommendations for a model with:

```ruby
class User < ApplicationRecord
  has_recommended :products
  has_recommended :products_v2, class_name: "Product"
end
```

And use the appropriate methods:

```ruby
user.update_recommended_products_v2(recs)
user.recommended_products_v2
```

For Rails < 6, speed up inserts by adding [activerecord-import](https://github.com/zdennis/activerecord-import) to your app.

## Storing Recommenders

If you’d prefer to perform recommendations on-the-fly, store the recommender

```ruby
bin = Marshal.dump(recommender)
File.binwrite("recommender.bin", bin)
```

> You can save it to a file, database, or any other storage system

Load a recommender

```ruby
bin = File.binread("recommender.bin")
recommender = Marshal.load(bin)
```

## Algorithms

Disco uses high-performance matrix factorization.

- For explicit feedback, it uses [stochastic gradient descent](https://www.csie.ntu.edu.tw/~cjlin/papers/libmf/libmf_journal.pdf)
- For implicit feedback, it uses [coordinate descent](https://www.csie.ntu.edu.tw/~cjlin/papers/one-class-mf/biased-mf-sdm-with-supp.pdf)

Specify the number of factors and epochs

```ruby
Disco::Recommender.new(factors: 8, epochs: 20)
```

If recommendations look off, trying changing `factors`. The default is 8, but 3 could be good for some applications and 300 good for others.

## Validation

Pass a validation set with:

```ruby
recommender.fit(data, validation_set: validation_set)
```

## Cold Start

Collaborative filtering suffers from the [cold start problem](https://www.yuspify.com/blog/cold-start-problem-recommender-systems/). It’s unable to make good recommendations without data on a user or item, which is problematic for new users and items.

```ruby
recommender.user_recs(new_user_id) # returns empty array
```

There are a number of ways to deal with this, but here are some common ones:

- For user-based recommendations, show new users the most popular items.
- For item-based recommendations, make content-based recommendations with a gem like [tf-idf-similarity](https://github.com/jpmckinney/tf-idf-similarity).

## Data

Data can be an array of hashes

```ruby
[{user_id: 1, item_id: 1, rating: 5}, {user_id: 2, item_id: 1, rating: 3}]
```

Or a Rover data frame

```ruby
Rover.read_csv("ratings.csv")
```

Or a Daru data frame

```ruby
Daru::DataFrame.from_csv("ratings.csv")
```

## Faster Similarity

If you have a large number of users/items, you can use an approximate nearest neighbors library like [NGT](https://github.com/ankane/ngt) to speed up item-based recommendations and similar users.

Add this line to your application’s Gemfile:

```ruby
gem 'ngt', '>= 0.3.0'
```

Speed up item-based recommendations with:

```ruby
model.optimize_item_recs
```

Speed up similar users with:

```ruby
model.optimize_similar_users
```

This should be called after fitting or loading the model.

## Reference

Get the global mean

```ruby
recommender.global_mean
```

Get the factors

```ruby
recommender.user_factors
recommender.item_factors
```

## Credits

Thanks to:

- [LIBMF](https://github.com/cjlin1/libmf) for providing high performance matrix factorization
- [Implicit](https://github.com/benfred/implicit/) for serving as an initial reference for user and item similarity
- [@dasch](https://github.com/dasch) for the gem name

## History

View the [changelog](https://github.com/ankane/disco/blob/master/CHANGELOG.md)

## Contributing

Everyone is encouraged to help improve this project. Here are a few ways you can help:

- [Report bugs](https://github.com/ankane/disco/issues)
- Fix bugs and [submit pull requests](https://github.com/ankane/disco/pulls)
- Write, clarify, or fix documentation
- Suggest or add new features
