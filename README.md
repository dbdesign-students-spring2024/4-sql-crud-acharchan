# SQL CRUD

An assignment to design relational database tables with particular applications in mind.

Complete the following exercises. For each:

- determine the structure of any tables
- determine the data type of any fields
- determine which field(s) is the primary key for each table
- determine which field(s) is the foreign keys for any tables that require them
- write SQL queries that perform the requested functionality.

## Part 1: Restaurant finder
### Table structure
Here is the SQL code to create the tables and insert the practice data:
```
CREATE TABLE restaurants (
    id INTEGER PRIMARY KEY,
    name TEXT NOT NULL,
    category TEXT NOT NULL,
    price_tier TEXT NOT NULL,
    neighborhood TEXT NOT NULL,
    opening_hours TEXT NOT NULL,
    average_rating REAL,
    good_for_kids BOOLEAN
);

CREATE TABLE reviews (
    id INTEGER PRIMARY KEY,
    restaurant_id INTEGER,
    user_name TEXT,
    review_text TEXT,
    rating REAL,
    FOREIGN KEY (restaurant_id) REFERENCES restaurants (id)
);
```

### Practice data
To import the data:
```
.mode csv
.import /Users/acharchan/Desktop/4-sql-crud-acharchan/data/restaurants.csv restaurants
```
Here is the link to the [restaurants CSV](/Users/agnac/4-sql-crud-acharchan/data/restaurants.csv)

### Queries

Write a single SQL query to perform each of the following tasks:

1. Find all cheap restaurants in a particular neighborhood (pick any neighborhood as an example)
```
SELECT * FROM restaurants WHERE price_tier = 'cheap' AND neighborhood = 'Chelsea';
```

1. Find all restaurants in a particular genre (pick any genre as an example) with 3 stars or more, ordered by the number of stars in descending order.
```
SELECT * FROM restaurants WHERE category = 'Italian' AND average_rating >= 3 ORDER BY average_rating DESC;
```

1. Find all restaurants that are open now (see hint below).
```
SELECT * FROM restaurants WHERE strftime('%H:%M', opening_hours) <= strftime('%H:%M', 'now') AND strftime('%H:%M', 'now') < strftime('%H:%M', opening_hours, '+1 day');
```

1. Leave a review for a restaurant (pick any restaurant as an example; note that leaving a review has no automatic effect on the average rating of the restaurant).
```
INSERT INTO reviews (restaurant_id, user_name, review_text, rating) VALUES (1, 'John Doe', 'Great place!', 4.5);
```

1. Delete all restaurants that are not good for kids.
```
DELETE FROM restaurants WHERE good_for_kids = 0;
```

1. Find the number of restaurants in each NYC neighborhood.
```
SELECT neighborhood, COUNT(*) FROM restaurants GROUP BY neighborhood;
```

## Part 2: Social media app

The social media app you are designing a database for allows Users to share two kinds of content: **Messages** and **Stories**.

### Tables structure
```
CREATE TABLE users (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  email TEXT NOT NULL UNIQUE,
  password TEXT NOT NULL,
  handle TEXT NOT NULL UNIQUE
);

CREATE TABLE posts (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  user_id INTEGER NOT NULL,
  content TEXT NOT NULL,
  is_visible BOOLEAN NOT NULL DEFAULT 1,
  post_type TEXT NOT NULL,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users (id)
);
```

### Practice data
```.mode csv
.import /Users/acharchan/Desktop/4-sql-crud-acharchan/data/users.csv users
```
Here is the link to the [users CSV](/Users/agnac/4-sql-crud-acharchan/data/users.csv)

```
.mode csv
.import /Users/acharchan/Desktop/4-sql-crud-acharchan/data/posts.csv posts
```
Here is the link to the [posts CSV](/Users/agnac/4-sql-crud-acharchan/data/posts.csv)

Here's an example of how to insert `data` into the users table:
```
INSERT INTO users (email, password, handle) VALUES ('user1@example.com', 'password1', 'user1');
INSERT INTO users (email, password, handle) VALUES ('user2@example.com', 'password2', 'user2');
```
and into the `posts` table:
```INSERT INTO posts (user_id, content, post_type) VALUES (1, 'Hello, User 2!', 'message');
INSERT INTO posts (user_id, content, post_type) VALUES (2, 'Hi, User 1!', 'message');
INSERT INTO posts (user_id, content, post_type) VALUES (1, 'Here is my story...', 'story');
```

### Queries

Write a single SQL query to perform each of the followin tasks:

1. Register a new User.
```
INSERT INTO users (email, password, handle) VALUES ('newuser@example.com', 'password', 'newuser');
```

1. Create a new Message sent by a particular User to a particular User (pick any two Users for example).
```
INSERT INTO posts (sender_id, receiver_id, post_type, post_text, visible_start_time, visible) VALUES (1, 9, 'message', 'Hello, how are you?', strftime('%Y-%m-%d %H:%M:%S', 'now', 'localtime'), 0);
```

1. Create a new Story by a particular User (pick any User for example).
```
INSERT INTO posts (sender_id, receiver_id, post_type, post_text, visible_start_time, visible) VALUES (4, 'NULL', 'story', 'Just having a great day!',strftime('%Y-%m-%d %H:%M:%S', 'now', 'localtime'), 1);
```
1. Show the 10 most recent visible Messages and Stories, in order of recency.

1. Show the 10 most recent visible Messages sent by a particular User to a particular User (pick any two Users for example), in order of recency.
```
SELECT * FROM posts WHERE is_visible = 1 AND user_id = 1 AND post_type = 'message' ORDER BY created_at DESC LIMIT 10;
```

1. Make all Stories that are more than 24 hours old invisible.
```
UPDATE posts SET is_visible = 0 WHERE post_type = 'story' AND created_at < DATE('now', '-24 hours');
```

1. Show all invisible Messages and Stories, in order of recency.
```
SELECT * FROM posts WHERE is_visible = 0 ORDER BY created_at DESC;
```

1. Show the number of posts by each User.
```
SELECT sender_id, COUNT(post_id) FROM posts GROUP BY sender_id;
```

1. Show the post text and email address of all posts and the User who made them within the last 24 hours.
```
SELECT users.email, posts.post_text FROM users INNER JOIN posts ON users.id = posts.sender_id WHERE ROUND((JULIANDAY('now') - JULIANDAY(posts.visible_start_time)) * 24) <= 24;
```

1. Show the email addresses of all Users who have not posted anything yet.
```
SELECT email FROM users LEFT JOIN posts ON users.id = posts.sender_id WHERE posts.sender_id IS NULL;
```



## Write a report

Update the `README.md` file to include the solutions to all of these instructions. For each part, be sure to include:

- the SQL code to create each of the required tables
- a link to each of the practice CSV data files in the `data` directory.
- the SQLite code to import the practice CSV data files into the tables.
- the SQL queries that solve each of the tasks you were asked to do. Make it clear which task each query is intended to solve - include the task number and text on the line above the SQL code solution.

Make the document well-formatted using Markdown code. Use clear headings and sub-headings and use Markdown's ability to embed code snippets into a document.

