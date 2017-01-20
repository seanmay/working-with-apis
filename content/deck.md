---
title: API Workshop
theme: css/theme/league.css
---

# Working with APIs

Note:

Sections to cover:  
- Understanding APIs
- Understanding Strategies when Using APIs
- Applying Strategies to the real world

---

## Understanding APIs

<div class="fragment">Application Programming Interface</div>
<div class="fragment">Interfaces given to consumers to use for their own purposes.</div>

Note:

In our case, Network interfaces, over HTTP.
Not in-language APIs.
Many of the concepts still apply in the abstract, though.

---

## REST

Representational State Transfer

---

**what ReST isn't**

```java
// POST /getItemsByDateFromItemByDateService
List<Item> itemsByDate = ItemByDateService.getItemsByDate();

// POST /validateSessionWithCredentialManager?user=Bob&password=123b0B!
// ...
```

<ul class="fragment">
  <li>SOAP
  <li>WSDL
</ul>

<blockquote class="fragment">
  “Now you can have Java build the XML,  
  to build the Java for your XML API!”
</blockquote>

Note: 
This is what a lot of code looks like, even today.  
We have frontends which have to know the backend, down to what code to run.

---

**what ReST is**

- Coined by Roy Fielding ~ 2000
- Uses HTTP Methods to describe Verbs
- Uses URLs to describe Nouns

<div class="fragment">
also: 

- Useful in many cases, but not a panacea
- A tool, not a dogma
</div>

Note: Recurring theme:

use the tools that are useful, where they're useful

---

## RESTful API Verbs

<ul>
  <li class="fragment">
    `GET` - "get", "fetch", "read", "load"  
    <small>_Give me X._</small>
  </li>
  <li class="fragment">
    `PUT` - "put", "replace", "update", "switch"  
    <small>_Your version of X is no good; use mine._</small>
  </li>
  <li class="fragment">
    `POST` - "create", "make", "insert"  
    <small>_Make me a new X; use this information._</small>
  </li>
  <li class="fragment">
    `DELETE` - "delete", "remove", "drop"  
    <small>_Get rid of X._</small>
  </li>
</ul>

---

## RESTful API Nouns

```http
/workshops/
  # all workshops

/workshops/rangle/
  # the subset of all workshops by Rangle

/workshops/rangle/working-with-backend-apis/
  # the subset of workshops by Rangle, called "Working..."
```

---

If you've ever been a programmer, you're probably familiar with this verb/noun combination:

```javascript
// GET /workshops/;
load(workshops);

// POST /examples/
//      Body: { "id": 123, "title"... }
examples[123] = new Example({ id: 123, /*...*/ });

// DELETE /theatre-goers/person-kicking-the-back-of-my-seat/
remove(theatreGoers.personKickingTheBackOfMySeat);
```

---

**What's the difference?**

```http
POST /getLatestUpdatesFromTwitterService
     Body: user=raffi_rc

GET /tweets/@raffi_rc
```

<ul class="fragment">
  <li>We're using the URL to describe the data we want, not the service we want to talk to.</li>
  <li>The URL is uniquely identifying the data we want to see.</li>
  <li>We're using the HTTP Method to define what we want to do with that unique resource.</li>
</ul>

Note: 
Exercise: Build an API

---

---

### Documenting an API

If we can't share what we made, what was the point of making an API?

---

### Formalization is Hard; NP Hard

This is where all of the difficult decisions get made.

---

<ul>
  <li>Excluded/`null` Members versus Empty Members</li>
  <li class="fragment">Resources mapped by ID, versus resources in a set</li>
  <li class="fragment">Error handling; response codes versus data properties</li>
</ul>

<br>
<div class="fragment">There are no right answers, but each will be less wrong, depending on circumstance.</div>

---

### What Would the Consumer Want?

We're building the API for consumers.  
<div class="fragment">Even if the consumer is Future-You.</div>
<div class="fragment">_Especially if the consumer is Future-You._</div>

---

### From the Consumer's Perspective

Request:  `DELETE /blogs/123/`  
Problem: It doesn't exist

Which is a better consumer experience?

- OK - The thing you wanted gone is gone
- ERROR! - Can't find the thing you want to be gone

---

### From the Consumer's Perspective 

Request:  `GET /blogs/123/comments/`  
Problem: No comments exist on the post yet.

Which is a better consumer experience?

- OK - Here is the list of 0 comments
- ERROR! - Can't find any comments for this post

---

### From the Consumer's Perspective

Request:  `GET /blogs/123/`  
Problem: User wants to use the `similarArticles` list on the object; there are none.

Which is a better consumer experience?

```json
// OK
{
  "id": "123",
  "similarArticles": null
}
```

```json
// OK
{
  "id": "123",
  "similarArticles": []
}
```

---

### From the Consumer's Perspective

Request:  `GET /blogs/123/comments/`  
Problem: User wants to use the `comments` for a given `Blog`.

Which is a better consumer experience?

```json
[
  { "id": "abc", ... },
  { "id": "def", ... }
]
```

```json
{
  "abc": { "id": "abc", ... },
  "def": { "id": "def", ... }
}
```

---

> “But what if my API needs to return extra metadata to work?”

That's fine; just pick a standard for root level meta versus data, and stick to it.  

```json
// GET /blogs/123/comments/
{
  "meta": {
    "type": "blog.comments",
    "parentId": "123",
    "resourceId": "/blogs/123/comments/",
    "contentType": "application/vnd.mycompany.blog-comments.v3+json"
  },
  "data": [
    { "id": "abc", ... }
  ]
}
```

---

> “How am I supposed to get all of this perfect, the first time.” 

<div class="fragment">Don’t worry. You can’t.</div>
<div class="fragment">Even if you got it perfect on your first try, your consumers or your boss would find new uses and requirements for your system, anyway.</div>
<div class="fragment">Expect your API to change.</div>

---

> “How am I supposed to think of the consumer, and provide what they want, while expecting what they want to change, all without breaking their app?”

---

### Versioning

By controlling the version a user consumes, you allow them to opt into your new features and new breaking-changes.

---

**You don‘t need to go full-SemVer**

Users are either getting

- more-specific data / new data properties (_feature_)
- less-specific data / changed data properties (_break_)

`v2.4` and `v24` might not be the same, but are both valid approaches.

Note: Consumers don't care what optimizations or server language changes they are opting into.  
Underlying performance improvements should happen without their effort.

---

**User Controlled Versions**

Now that you’ve decided on your versioning scheme, how are you going to let users opt into the version they want to use?

---

**Three Common Methods**

URL Versioning 

```http
GET /v2/blogs/123/
```

Custom-Header Versioning

```http
GET /blogs/123/
MyApp-Version: v24
```

Accepts-Header Versioning

```http
GET /blogs/123/
Accepts: application/vnd.mycompany.myapi.v24+json
```

---

**So many choices!**
**Which is the right one?**

---

They're all bad!
Pick the one that works for your project, socialize it and stick with it.

---

URL Versioning 

Pros:

- Good for quick construction / implementation
- Don’t have to implement fancy cache-invalidation
- Easy to test and work with

Cons:

- Leaking details into the resource description
- Maintenance pain on the consumer side, without abstractions

---

Custom Header Versioning

Pros:

- Keeps version out of URL
- Custom schemas

Cons:

- Worst of both worlds
- Custom schemas

---

Accepts Header Versioning

Pros:

- Keep version out of URL
- Standard spot for encoding type, version, and format

Cons:

- Hard to get started with
- Harder to manually test

---

Prefer URL or Accepts Header Versioning; choose the one that will balance your consumer’s ease of implementation and your ease of maintenance.

---

**Documenting APIs**

- Swagger
- RAML
- Apiary

Swagger is a common “Enterprise” standard.  
Be familiar with its doc-generating;  
not its code-gen.

---


---

### Consuming APIs 

---

**Testing APIs**

- Postman
- In-browser URL Testing
- Integration Testing (Jasmine / Mocha / et cetera)
- Load Testing (Artillary)


---

**Solving Common Problems**

---

**Problem:**  
Data shape is deep/complex, and difficult to reason/navigate.

**Solution:**  
Frontend DSL

---

### DSL 

**Domain Specific Language**

There are textbooks dedicated to this topic.  
In short, name things that provide value, in ways everyone understands.

---

- **Business Objects**  
  The specific objects that provide the business value, in the format you want to use them in the client app.

- **Business Intents**  
  The named features / functionality which add value to the customer or the business.

---



- Define the shapes and properties that make sense to the business, not what the server has to offer
- Don't go overboard; define meaningful objects, not glue
- Do formalize glue, if it becomes important *when* it does


---

```http
GET /bands/sonny+and+cher/band-members/ 
```

```json
{
  "data": [
    { "id": "123", "name": { "first": "Sonny", "last": "Bono" } },
    { "id": "234", "name": "Cher" }
  ]
}
```

---

```typescript
interface Person {
  id: string;
  type: "Person";
  firstName: string;
  lastName: string;
}

interface Personality {
  id: string;
  type: "Personality";
  name: "Madonna" | "Cher" | "Prince";
}

type Artist = Person | Personality;
```

---

```typescript
interface MembershipDetails {
  yearsActive: string;
  isFounder: boolean;
  title: string;
}

type BandMember = MembershipDetails & Artist;

interface Band {
  id: string;
  name: string;
  members: BandMember[];
}
```

---

- What problems did we just solve for ourselves?
- What problems did we just solve for future teammates?
- How can we apply this to designing / updating backend services?

---

---

**Problem:**  
Data is returned with unpredictable shape.

- Occasionally missing fields
- Non-optimal shapes
- More complex than needed, or too normalized

**Solution:**  
Transform Layer

---


What if the payload comes back like this?

```json
{
  "band_name": "...",
  "band_member_1_first_name": "...",
  "band_member_4_first_name": "Cher",
  "band_member_2_first_name": "...",
  "band_member_1_last_name": "...",
  "band_member_4_last_name": null
}
```

Do you want to be referencing those properties through your application?

> Hint: Nope

---

```typescript
class BandDataHelper {
  castFromAwfulInput (input: AwfulBandData): BandData {
    // ...
  }
}

class BandService {
  constructor (
      private lib: BandDataHelper,
      private dataService: SomeDataService
  ) { }

  loadData (id: string): BandData {
    return this.dataService.getData(id)
      .then(this.lib.castFromAwfulInput);
  }
}
```

---

- What problems did we just solve?
- What benefits did we just gain?

---

**Testability**

How much work would it be to test each component of this service, now?

```typescript
const dataService = new FakeDataService();

const bandDataHelper = new BandDataHelper();
// test that the bad data goes in
// and the right shape comes out

const bandService = new BandService(
  bandDataHelper,
  dataService
);
// test that the service gets called,
// and the data gets transformed
```

---

---

**Problem:**
Initial API is not ready, or there is no "dev" version.

**Solution:**
Inject mock data, near the Transform Layer

---

We've actually seen the solution to this problem, already.

```typescript
const dataService = new FakeDataService();
const bandDataHelper = new BandDataHelper();

const bandService = new BandService(
  bandDataHelper,
  dataService
);
```

---

- What problems have we solved?
- What SOLID principles have we used?
- What opportunities have we created?
- How could we use this to easily migrate between API versions or even between APIs?

---

---

### Dealing with Humans

---

**Problem:**  
You need to make headway on the project *now*, but the API doesn't exist yet.


Note:
- Make a DSL  
- Make a mock data-service

---

**Problem:**  
The API keeps changing from under your nose; the backend team swears their releases are just "improvements" that have no impact on the data.

---

**Problem:**  
The backend team is in the middle of migrating a huge system, between XML and JSON, and between a typical WebService and REST.

They would like our input.

---

*fin*
