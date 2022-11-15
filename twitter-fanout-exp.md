# Twitter Fan-out Example

## Main feature load parameters:

- Post tweet: 4.6k rps average, 12k rps at peak
- Home timeline: 300k rps

## Approach 1: Pull

1. Posting a tweet writes it directly into database
2. When a user requests its timeline, query and filter from all the followings and merge into a timeline

## Approach 2: Push

1. Posting a tweet not only writes it into database, but inserts it into all followers' timelines as well
2. Timeline of a user is always ready to be requested

## Middle ground

For celebrities with a huge amount of followers, user pull to reduce the amount of write operations required; otherwise, use push