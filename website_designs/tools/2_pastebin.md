# Pastebin
- Platforms: pastebin.com, pasted.co, chopapp.com
- Users can store plain text or images
- Users get a randomly generated URL to access it
- Used to share data quickly (source code, configs, logs)

## Requirements
- Functional Requirements
  - Users can upload or paste their data and get a unique URL to access it
  - Users can only upload text
  - Data and links will expire automatically after a default timespan
  - Users can optionally pick a custom alias and specify custom expiration time
- Non-Functional Requirements
  - System should be highly reliable, any uploaded data shouldn't be lost
  - System should be highly available, users should be able to access their data at any time
  - Users should be able to access their data in real time with minimum latency
  - Paste links shouldn't be predictable
- Extended Requirements
  - Analytics like how many times a paste was accessed
  - Service should be accessible through rest api

## Considerations
- What should be the limit of the text that can be pasted
  - Since it's not a full fledged storage cloud, it should be small
  - We can keep the limit at 10 MB to stop the abuse of the service
- Should we impose size limit on custom URLs
  - Impose a size limit on a custom alias to ensure we've consistent URL database
  - Let's assume users can specify a max of 16 chars per customer key

## Estimation
- System would be read heavy, assume the read-write ratio to be 5 : 1
- Traffic
  - Write (New pastes): 1M/day = 12/s
  - Read (Paste accesses): 5M/day = 58/s
- Storage
  - Average size of each paste: 10KB
    - Users store text like source code, configs, logs
    - Such texts are not huge
  - Daily data: 1M paste/day * 10KB/paste = 10GB/day
  - Storage time for each paste: 10 years
  - Storage required: (10GB * 365 days) * 10 years = 36TB
  - Total storage with margin of 30%: 36TB * 1.3 = 51.4TB
- Storage for Keys
  - 6 letter keys with base64 encoding ([A-Z, a-z, 0-9, ., -])
    - 64^6 ~= 68.7 billion unique strings
  - Keys required for 10 years: 1M paste/day * 365 days * 10 years = 3.6B keys
  - If it takes one byte to store one char: 3.6B * 6 = 22GB
  - 22GB is negligible compared to 36TB required for data
- Bandwidth
  - Write: 12 paste/s * 10KB/paste = 120 KB/s
  - Read: 58 paste/s * 10KB/paste = 0.6 MB/s
- Memory
  - We can cache 20% of the pastes that are frequently accessed (80-20 principle)
  - Cache storage: 5M paste/day * 10KB/paste * 20% = 10 GB/day

## API
- addPaste (returns the paste url)
  - Required: api_key, data
  - Optional: custom_url, user_id, paste_name, expiration_date
- getPasteInfo
  - Required: api_key, paste_url
- editPaste
  - Required: api_key, paste_url
  - Payload: data, custom_url, paste_name, expiration_date
- deletePaste
  - Required: apiKey, paste_url

## Database
- Constraints
  - We need to store billions of records
  - Read heavy (58 requests/s)
  - Each metadata object would be small (< 100 bytes)
  - Each paste object can have max size of 10 MB
  - There are no relationship between records
- Schema
  - Paste (id, key, paste_url, content_url, created_at, expiration_at)
  - User (id, email, name, api_key, created_at, last_login_at)
- Type
  - We anticipate storing billions of rows and don't need relationships
  - NoSQL key-value store (like Dynamo or Cassandra) is a good choice
  - Should we store the content in this db itself or in external file storage like S3?
  - It would be easier to scale as well

# Component Design
## Generating Key for Pastes
- Refer to URL shortening for detailed description, the main points are listed below
- Generate a 6 letter random string which would server as the key of the paste
  - If user has provided a custom key, then consider that
  - If the key is a duplicate, keep generating keys until a unique key is found
- Another solution is to run a standalone Key Generation Service (KGS)
  - It will generate these keys beforehand and store them in a separate database
  - It can have two tables: used keys, unused keys
  - It can keep some keys in memory to provide keys quickly

# Scalability
- Similar to URL shortening
