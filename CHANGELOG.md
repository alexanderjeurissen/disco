## 0.2.0 (2020-07-31)

- Changed score to always be between -1 and 1 for `item_recs` and `similar_users` (cosine similarity - this makes it easier to understand and consistent with `optimize_item_recs` and `optimize_similar_users`)

## 0.1.3 (2020-06-28)

- Added support for Rover
- Raise error when missing user or item ids
- Fixed string keys for Daru data frames
- `optimize_item_recs` and `optimize_similar_users` methods are no longer experimental

## 0.1.2 (2020-03-26)

- Added experimental `optimize_item_recs` and `optimize_similar_users` methods

## 0.1.1 (2019-11-14)

- Fixed Rails integration

## 0.1.0 (2019-11-14)

- First release
