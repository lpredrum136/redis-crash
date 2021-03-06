Redis
Install redid by googling

Play around:

- Redis-cli
- 127.0.0.1:6379> SET name Henry
- … > GET name

Regis kinda always returns string

- SET age 26
- GET age (return “26”)

Delete key:

- DEL age

Check if key exists:

- EXISTS name

Find keys that match pattern:

- KEYS \*

Delete everything:

- FLUSHALL

Clear screen

- CLEAR

Get time to live:

- TTL name (returns seconds left to live, -1 is forever, -2 is non-existent)

Set time to live (in seconds)

- EXPIRE name 10

Set expirable key:

- SETEX name 10

WORKING WITH LIST (ORDERED ARRAY)

Create an array called “friends” with one element “John”

- LPUSH friends john
- Note: GET friends will return an error, since “GET” only works for strings
- LRANGE friends 0 -1 instead

Continue to push more:

- LPUSH friends sally
- LRANGE friends 0 -1

Push from the end of the array

- RPUSH friends mike
- LRANGE friends 0 -1

Pop from array

- LPOP friends
- RPOP friends

WORKING WITH SETS (LIKE LIST, BUT EVERY ELEMENT IS UNIQUE, AND THERE IS NO ORDER)

Add to a set:

- SADD hobbies “weight lifting”

Get members of a set

- SMEMBERS hobbies

You cannot add the same member:

- SADD hobbies “weight lifting” will return 0
- SMEMBERS hobbies will still have only one member

Remove a member:

- SREM hobbies “weight lifting”

WORKING WITH HASHES (LIKE ONE-LEVEL JSON): { key: { field: value, field: value } }

Create a hash

- HSET person name henry

Get a hash field

- HGET person name

Get all hash fields

- HGETALL person (This will return field -> value -> field -> value…)

Remove a field (along with its value)

- HDEL person age

Check if a field exists

- HEXISTS person name

REAL APP

Install redis

- Yarn add redis

Start redis

- Redis-server or brew services start redid

Use redis in code

```


import Redis from ‘redis’

const DEFAULT_EXPIRATION = 3600

const client = Redis.createClient({ url?: <production url>})

const photos = await getOrSetCache(`photos?albumId=${someInputAlbumId}`, async () => {
  // if we don't already have 'photos' in redis, we have to get data for real,
  // and save it in redis this time
  const data = await axios.get('...')
  return data
})

return photos

// With the use of refactored getOrSetCache, this whole chunk client.get() can be commented out
client.get(`photos?albumId=${someInputAlbumId}`, (error, photos) => {
  if (error) console.log(error)

  // if we already have 'photos' in redis
  if (photos != null) {
    return JSON.parse(photos) // since photos must've been JSON stringified
  }

  // if we don't already have 'photos' in redis, we have to get data for real,
  // and save it in redis this time
  const data = await axios.get('...')

  // client.set('name', 'henry')
  client.setex(`photos?albumId=${someInputAlbumId}`, DEFAULT_EXPIRATION, JSON.stringify(data))

  return data
})

const getOrSetCache = (key, callback) =>
  new Promise((res, rej) => {
    client.get(key, async (error, data) => {
      if (error) return reject(error)
      if (data != null) return resolve(JSON.parse(data))

      const freshData = await callback()
      client.setex(key, DEFAULT_EXPIRATION, JSON.stringify(freshData))

      resolve(freshData)
    })
  })


```

Now after the request, you can go to redis-cli and type GET photos or GET name, and you’ll see the data JSON stringified
