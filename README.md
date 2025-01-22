# Introduction to MongoDB Aggregation Pipelines

MongoDB aggregation pipelines provide a robust and versatile way to work with your data. They allow to transform, filter, and group information stored in  collections seamlessly. By chaining together a series of stages, each performing a specific operation, we can manipulate the data and uncover valuable insights efficiently.

## Schema Overview

### Users Collection

```
{
  "_id": { "$oid": "678e6ac8ba2404dd3d61fe7e" },
  "index": 0,
  "name": "Aurelia Gonzales",
  "isActive": false,
  "registered": { "$date": "2015-02-11T04:22:39.000Z" },
  "age": 20,
  "gender": "female",
  "eyeColor": "green",
  "favoriteFruit": "banana",
  "company": {
    "title": "YURTURE",
    "email": "aureliagonzales@yurture.com",
    "phone": "+1 (940) 501-3963",
    "location": {
      "country": "USA",
      "address": "694 Hewes Street"
    }
  },
  "tags": ["enim", "id", "velit", "ad", "consequat"]
}
```

### Authors Collection

```
{
  "_id": 100,
  "name": "F. Scott Fitzgerald",
  "birth_year": 1896
}
```

### Books Collection

```
{
  "_id": 1,
  "title": "The Great Gatsby",
  "author_id": 100,
  "genre": "Classic"
}
```

## Aggregation Examples

### 1. How Many Active Users Are There?

```
[
  { "$match": { "isActive": true } },
  { "$count": "activeUsers" }
]
```

**Explanation:**

- `$match`: Filters documents to include only those where `isActive` is `true`.
    
- `$count`: Counts the resulting documents and outputs the count as `activeUsers`.
    

---

### 2. Average Age of Users

```
[
  { "$group": { "_id": null, "avgAge": { "$avg": "$age" } } }
]
```

**Explanation:**

- `$group`: Groups all documents together (`_id: null`) and calculates the average age using `$avg`.
    

---

### 3. Average Age of Males and Females

```
[
  { "$group": { "_id": "$gender", "avgAge": { "$avg": "$age" } } }
]
```

**Explanation:**

- `$group`: Groups documents by `gender` and calculates the average age for each group.
    

---

### 4. Top 5 Most Common Fruits

```
[
  { "$group": { "_id": "$favoriteFruit", "users": { "$sum": 1 } } },
  { "$sort": { "users": -1 } },
  { "$limit": 5 }
]
```

**Explanation:**

- `$group`: Groups documents by `favoriteFruit` and counts users for each fruit.
    
- `$sort`: Sorts the fruits by user count in descending order.
    
- `$limit`: Limits the result to the top 5 fruits.
    

---

### 5. Total Number of Males and Females

```
[
  { "$group": { "_id": "$gender", "users": { "$sum": 1 } } }
]
```

**Explanation:**

- `$group`: Groups documents by `gender` and counts the number of users in each group.
    

---

### 6. Country with the Most Registered Users

```
[
  { "$group": { "_id": "$company.location.country", "users": { "$sum": 1 } } },
  { "$sort": { "users": -1 } },
  { "$limit": 1 }
]
```

**Explanation:**

- `$group`: Groups documents by country and counts the number of users.
    
- `$sort`: Sorts by user count in descending order.
    
- `$limit`: Outputs the country with the highest user count.
    

---

### 7. List All Unique Eye Colors and Their Count

```
[
  { "$group": { "_id": "$eyeColor" } },
  { "$count": "uniqueEyeColors" }
]
```

**Explanation:**

- `$group`: Groups documents by unique `eyeColor` values.
    
- `$count`: Counts the unique eye colors.
    

---

### 8. Average Number of Tags per User

#### Using `$unwind`:

```
[
  { "$unwind": "$tags" },
  { "$group": { "_id": "$_id", "numberOfTags": { "$sum": 1 } } },
  { "$group": { "_id": null, "avgTags": { "$avg": "$numberOfTags" } } }
]
```

**Explanation:**

- `$unwind`: Deconstructs the `tags` array, creating a document for each tag.
    
- `$group`: First groups by user (`_id`) to count tags per user, then calculates the average tag count across all users.
    

#### Using `$addFields`:

```
[
  { "$addFields": { "numberOfTags": { "$size": "$tags" } } },
  { "$group": { "_id": null, "avgTags": { "$avg": "$numberOfTags" } } }
]
```

**Explanation:**

- `$addFields`: Adds a field `numberOfTags` containing the size of the `tags` array for each user.
    
- `$group`: Calculates the average of `numberOfTags` across all users.
    

---

### 9. Users with a Specific Tag

#### Users with the Tag `enim`:

```
[
  { "$match": { "tags": "enim" } },
  { "$count": "NumberOfUsers" }
]
```

**Explanation:**

- `$match`: Filters users whose `tags` array contains `enim`.
    
- `$count`: Counts these users.
    

#### Users Who Are Inactive and Have the Tag `velit`:

```
[
  { "$match": { "isActive": false, "tags": "velit" } },
  { "$project": { "name": 1, "age": 1 } }
]
```

**Explanation:**

- `$match`: Filters users who are inactive and have `velit` in their `tags` array.
    
- `$project`: Selects and outputs only the `name` and `age` fields.
    

---

### 10. Users with Phone Numbers Starting with "+1 (940)"

```
[
  { "$match": { "company.phone": { "$regex": "^\\+1 \\\(940\\)" } } },
  { "$count": "numberOfUsers" }
]
```

**Explanation:**

- `$match`: Filters users whose phone numbers start with `+1 (940)` using a regular expression.
    
- `$count`: Counts the resulting users.
    

---

### 11. User Registered Most Recently

```
[
  { "$sort": { "registered": -1 } },
  { "$limit": 1 }
]
```

**Explanation:**

- `$sort`: Sorts users by the `registered` field in descending order.
    
- `$limit`: Outputs the most recently registered user.
    

---

### 12. Categorize Users by Favorite Fruit

```
[
  { "$group": { "_id": "$favoriteFruit", "users": { "$push": "$name" } } },
  { "$addFields": { "numberOfUsers": { "$size": "$users" } } },
  { "$sort": { "numberOfUsers": -1 } }
]
```

**Explanation:**

- `$group`: Groups users by their favorite fruit and collects their names.
    
- `$addFields`: Adds a field `numberOfUsers` containing the size of the `users` array for each fruit.
    
- `$sort`: Sorts the fruits by the number of users in descending order.
    

---

### 13. Users with "ad" as Their Second Tag

```
[
  { "$match": { "tags.1": "ad" } },
  { "$count": "numberOfUsers" }
]
```

**Explanation:**

- `$match`: Filters users whose second tag (`tags[1]`) is `ad`.
    
- `$count`: Counts these users.
    

---

### 14. Users with Both "enim" and "id" Tags

```
[
  { "$match": { "tags": { "$all": ["enim", "id"] } } }
]
```

**Explanation:**

- `$match`: Filters users whose `tags` array contains both `enim` and `id`.
    

---

### 15. Companies Located in the USA and Their Counts

```
[
  { "$match": { "company.location.country": "USA" } },
  { "$group": { "_id": "$company.title", "NumberOfCompanies": { "$sum": 1 } } }
]
```

**Explanation:**

- `$match`: Filters companies located in the USA.
    
- `$group`: Groups companies by their title and counts occurrences.
    

---

### 16. Authors Who Have Written Books in a Specific Genre

#### Scenario 1: One Author per Book

```
[
  { "$match": { "genre": "Dystopian" } },
  { "$lookup": { "from": "authors", "localField": "author_id", "foreignField": "_id", "as": "author" } },
  { "$addFields": { "author": { "$arrayElemAt": ["$author", 0] } } },
  { "$project": { "author.name": 1, "author.birth_year": 1, "_id": 0 } }
]
```

**Explanation:**

- `$match`: Filters books with the genre `Dystopian`.
    
- `$lookup`: Joins the `authors` collection based on `author_id`.
    
- `$addFields`: Extracts the first author from the `author` array.
    
- `$project`: Outputs only the author’s name and birth year.
#### Scenario 2: Multiple Authors per Book

```
[
  {
    "$match": {
      "genre": "Classic"
    }
  },
  {
    "$lookup": {
      "from": "authors",
      "localField": "author_id",
      "foreignField": "_id",
      "as": "authors",
      "pipeline": [
        {
          "$project": {
            "name": 1,
            "birth_year": 1,
            "_id": 0
          }
        }
      ]
    }
  },
  {
    "$project": {
      "title": 1,
      "authors": 1,
      "_id": 0
    }
  }
]
```

**Explanation:**

- `$match`: Filters books with the genre `Classic`.
    
- `$lookup`: Joins the `authors` collection, linking by `author_id` and adding a list of authors under the `authors` field.
    
- `pipeline`: Limits fields returned from the joined `authors` documents to just `name` and `birth_year`.
    
- `$project`: Outputs the book title and the list of authors.
    

---

### 17. Books Grouped by Genre with Total Count per Genre

```
[
  {
    "$group": {
      "_id": "$genre",
      "totalBooks": {
        "$sum": 1
      }
    }
  },
  {
    "$sort": {
      "totalBooks": -1
    }
  }
]
```

**Explanation:**

- `$group`: Groups books by genre and calculates the total number of books per genre.
    
- `$sort`: Sorts the genres in descending order of book count.
    

---

### 18. Books Written by Authors Born Before 1900

```
[
  {
    "$lookup": {
      "from": "authors",
      "localField": "author_id",
      "foreignField": "_id",
      "as": "author"
    }
  },
  {
    "$unwind": "$author"
  },
  {
    "$match": {
      "author.birth_year": {
        "$lt": 1900
      }
    }
  },
  {
    "$project": {
      "title": 1,
      "author.name": 1,
      "_id": 0
    }
  }
]
```

**Explanation:**

- `$lookup`: Joins the `authors` collection.
    
- `$unwind`: Deconstructs the `author` array, creating a document for each author-book pair.
    
- `$match`: Filters authors born before 1900.
    
- `$project`: Outputs the book title and the author’s name.
    

---

### 19. Total Books Written by Each Author

```
[
  {
    "$group": {
      "_id": "$author_id",
      "totalBooks": {
        "$sum": 1
      }
    }
  },
  {
    "$lookup": {
      "from": "authors",
      "localField": "_id",
      "foreignField": "_id",
      "as": "author"
    }
  },
  {
    "$unwind": "$author"
  },
  {
    "$project": {
      "author.name": 1,
      "totalBooks": 1,
      "_id": 0
    }
  }
]
```

**Explanation:**

- `$group`: Groups books by `author_id` and calculates the total number of books per author.
    
- `$lookup`: Joins the `authors` collection to add author details.
    
- `$unwind`: Deconstructs the `author` array for each document.
    
- `$project`: Outputs the author’s name and their total book count.
