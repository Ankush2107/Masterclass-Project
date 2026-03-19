# Masterclass 3 — The Memory
## NoSQL Modeling & Schema Design with MongoDB

---

## Slide 1 — Welcome Back

**What we've built so far:**
- 🎨 **Masterclass 1 (Face)** — Complete React frontend with all pages, components, context providers, and API service layer
- 🧠 **Masterclass 2 (Brain)** — Express REST API with JWT auth, Groq AI integration, voice transcription, and request validation

**The problem right now:**
If you restart the server, every user account and every chat conversation disappears. That's because the server is holding data only in memory (JavaScript variables). Memory is wiped when the process stops. We need a database.

**Today's focus → THE MEMORY**
We're going to connect our app to **MongoDB**, a document database that stores data permanently on disk. Even if the server crashes, restarts, or is deployed to a new machine — the data survives. We'll use **Mongoose**, a library that acts as a bridge between our JavaScript code and the MongoDB database.

---

## Slide 2 — The 4 Masterclass Journey

| # | Name | What We Build |
|---|------|---------------|
| 1 | The Face | React UI — pages, components, routing |
| 2 | The Brain | Express server, REST APIs, Groq AI |
| **3** | **The Memory** ← *You are here* | MongoDB, Mongoose schemas, persistent data |
| 4 | The Launch | Cloud deployment to Vercel + Render |

---

## Slide 3 — What is MongoDB?

**MongoDB** is a NoSQL document database. It stores data in a format called BSON (Binary JSON) — which looks and behaves almost exactly like JavaScript objects. This makes it a natural fit for JavaScript applications.

Before learning MongoDB, let's map familiar concepts from the SQL/relational world to the MongoDB world:

```
SQL World            →    MongoDB World
──────────────────────────────────────────
Database             →    Database
Table                →    Collection
Row (one record)     →    Document (one object)
Column (field type)  →    Field
Primary Key (id)     →    _id (auto-generated ObjectId)
Foreign Key (ref)    →    Reference (ObjectId) or Embedded document
JOIN (combining)     →    $lookup aggregation OR embed the data
ALTER TABLE          →    Just add the field to new documents
```

**Example MongoDB document (what a User looks like stored):**
```json
{
  "_id": "64abc1230000000000000001",
  "name": "Ankush Kumar",
  "email": "ankush@example.com",
  "password": "$2b$12$crypted_hash_here",
  "createdAt": "2024-01-15T10:30:00.000Z",
  "updatedAt": "2024-01-15T10:30:00.000Z",
  "__v": 0
}
```

**Key MongoDB advantages:**
- **Flexible schema** — You can add new fields to new documents without migrating existing ones
- **JSON-native** — Data looks exactly like the JavaScript objects you're already working with
- **Embedded documents** — Related data (like messages inside a chat) can live in the same document — no JOINs needed
- **Horizontal scaling** — MongoDB can distribute data across many servers as your app grows

---

## Slide 4 — What is Mongoose?

**Mongoose** is an ODM (Object Document Mapper). Think of it as the translator between your JavaScript application and MongoDB. Without Mongoose, you'd use the raw MongoDB driver, which gives you no validation, no hooks, and no schema enforcement.

**What Mongoose adds on top of MongoDB:**

| Feature | What it does |
|---------|-------------|
| **Schema** | Defines the shape of your documents — what fields exist, what types they are, which are required |
| **Validation** | Automatically checks data before saving; throws an error if something is wrong |
| **Pre/Post Hooks** | Functions that run automatically before or after certain operations (like save, delete) |
| **Query Helpers** | Clean, chainable methods like `.find()`, `.findById()`, `.sort()`, `.select()` |
| **Virtual Fields** | Computed properties that aren't stored in the DB but computed from stored values |
| **Population** | Replace ObjectId references with the actual documents they point to |
| **Timestamps** | Automatically add and update `createdAt` and `updatedAt` fields |

**Mongoose in 3 lines:**
```js
import mongoose from 'mongoose';
const userSchema = new mongoose.Schema({ name: String, email: String });
const User = mongoose.model('User', userSchema); // Creates the 'users' collection
```

---

## Slide 5 — Connecting to MongoDB

The connection setup is straightforward. We create a function in `config/db.js` and call it at the very start of `server.js`. If the connection fails, we exit immediately — there is no point running the server without a database.

```js
// server/config/db.js
import mongoose from 'mongoose';

const connectDB = async () => {
  try {
    // mongoose.connect returns a promise — we await it
    // If successful, we get back a connection object
    const conn = await mongoose.connect(process.env.MONGODB_URI);

    // Log the host so we can visually confirm which database we're connected to
    console.log(`MongoDB connected: ${conn.connection.host}`);
    // Example output: "MongoDB connected: cluster0.abc123.mongodb.net"

  } catch (error) {
    // Log the error message so we know what went wrong
    console.error('MongoDB connection failed:', error.message);

    // Exit the Node.js process entirely with a failure code (1)
    // There's no point running the app if we can't talk to the database
    process.exit(1);
  }
};

export default connectDB;
```

**How to get the MONGODB_URI:**

For **local development**, install MongoDB locally:
```
mongodb://localhost:27017/classmentor
```

For **cloud** (recommended), use MongoDB Atlas (free):
```
mongodb+srv://myUser:myPassword@cluster0.abc123.mongodb.net/classmentor
```

Breaking down the Atlas URI:
- `mongodb+srv://` — The protocol (SRV record, used by Atlas clusters)
- `myUser:myPassword` — The database user credentials you created in Atlas
- `@cluster0.abc123.mongodb.net` — The Atlas cluster hostname
- `/classmentor` — The name of the database (Atlas creates it automatically)

---

## Slide 6 — The User Model

The User model defines exactly what a user document looks like in MongoDB, what rules must be followed, and what happens automatically during certain operations.

```js
// server/models/User.js
import mongoose from 'mongoose';
import bcrypt from 'bcryptjs';

// Step 1: Define the schema — the blueprint for every user document
const userSchema = new mongoose.Schema(
  {
    name: {
      type: String,                              // JavaScript String type
      required: [true, 'Please provide a name'], // Can't save without this
      trim: true,                                // Auto-trim whitespace from both ends
    },
    email: {
      type: String,
      required: [true, 'Please provide an email'],
      unique: true,      // Creates a MongoDB index — enforces no duplicates at database level
      lowercase: true,   // Auto-converts "ANKUSH@GMAIL.COM" → "ankush@gmail.com" before saving
      trim: true,        // Remove accidental leading/trailing spaces
    },
    password: {
      type: String,
      required: [true, 'Please provide a password'],
      minlength: 6,      // Mongoose validates length before saving
      select: false,     // VERY IMPORTANT: excludes password from ALL query results by default
                         // User.findOne({ email }) will NOT return the password field
                         // You must explicitly request it: User.findOne().select('+password')
    },
  },
  {
    timestamps: true,    // Mongoose auto-manages createdAt and updatedAt on every save
  }
);

// Step 2: Pre-save hook — runs AUTOMATICALLY before every user.save() call
userSchema.pre('save', async function (next) {
  // 'this' refers to the user document being saved

  // Only hash if the password field was actually changed
  // This check is crucial — without it, every save (including profile name updates)
  // would re-hash the already-hashed password, making it unverifiable
  if (!this.isModified('password')) return next();

  // Hash the password with a cost factor of 12
  // Cost factor 12 means bcrypt runs 2^12 = 4096 iterations of its algorithm
  // Higher = more secure but slower. 12 is the industry-recommended balance.
  this.password = await bcrypt.hash(this.password, 12);

  next(); // Signal that the hook is done; proceed with saving
});

// Step 3: Create the model from the schema
// 'User' becomes the collection name 'users' (Mongoose auto-pluralises and lowercases)
const User = mongoose.model('User', userSchema);
export default User;
```

---

## Slide 7 — User Schema: Every Design Decision Explained

Every field option in the User schema was chosen intentionally. Here's the reasoning behind each one:

**`email: unique: true`**
This tells MongoDB to create a unique index on the email field. An index is a separate data structure that allows MongoDB to find documents by email in O(log n) time instead of scanning the entire collection. The `unique` constraint means MongoDB will reject any attempt to insert a document with a duplicate email — this is your last line of defence even if the application-level check misses it.

**`email: lowercase: true`**
Without this, a user could register with `Ankush@Gmail.com` and then be unable to log in by typing `ankush@gmail.com` — because they'd be treated as different emails. By forcing lowercase, we treat all variations of an email as the same identity.

**`password: select: false`**
This is a critical security setting. Without it, every query like `User.findById(id)` would return the hashed password in the response — and if that response is accidentally logged or exposed, the hash could be used in dictionary attacks. With `select: false`, the password is completely invisible unless you explicitly ask for it with `.select('+password')`.

**`password: pre-save hook`**
By hashing in the model rather than the controller, we get automatic hashing for ALL password changes — registration, password reset, admin updates. The controller just sets `user.password = newValue; user.save()` and the hook does the rest. This is the DRY (Don't Repeat Yourself) principle.

**`timestamps: true`**
Gives us `createdAt` (when the account was created) and `updatedAt` (when it was last modified) for free, with no manual management needed.

---

## Slide 8 — The Chat Model

The Chat model is more complex than User because it contains an embedded array of messages. Rather than creating a separate Messages collection, each chat document contains all its messages directly inside an array.

```js
// server/models/Chat.js
import mongoose from 'mongoose';

// ─── MESSAGE SUBDOCUMENT SCHEMA ─────────────────────────────────
// A subdocument schema defines the structure of items INSIDE an array
// It's not a standalone collection — it exists only inside Chat documents

const messageSchema = new mongoose.Schema(
  {
    role: {
      type: String,
      enum: ['user', 'assistant'], // ONLY these two values are accepted
                                   // Mongoose will throw a ValidationError
                                   // if anything else is set (e.g. 'system', 'admin')
      required: true,
    },
    content: {
      type: String,
      required: true, // Every message must have content — no empty messages
    },
  },
  { timestamps: true } // Each message gets its own createdAt — useful for showing send times
);

// ─── CHAT SCHEMA ────────────────────────────────────────────────
const chatSchema = new mongoose.Schema(
  {
    user: {
      type: mongoose.Schema.Types.ObjectId, // Stores a special 24-character MongoDB ID
      ref: 'User',   // This is a reference to the User model's collection
                     // Mongoose uses this for .populate() if you want to JOIN the user data
      required: true, // Every chat must belong to a user — orphan chats are not allowed
    },
    title: {
      type: String,
      default: 'New Chat', // If no title is set, MongoDB uses this value automatically
                           // We update it after the first message is exchanged
    },
    messages: [messageSchema], // An array of embedded message subdocuments
                               // Stored directly inside the chat document — no JOIN needed
  },
  { timestamps: true } // Chat gets createdAt and updatedAt as well
);

const Chat = mongoose.model('Chat', chatSchema);
export default Chat;
```

---

## Slide 9 — Embed vs Reference: The Fundamental Choice

This is one of the most important design decisions in MongoDB. **When should you store related data inside the same document (embed), and when should you store it separately and reference it by ID?**

**Option A — Embedded Documents (what we chose for messages):**
```json
{
  "_id": "chat001",
  "user": "user123",
  "title": "How does React work?",
  "messages": [
    { "_id": "msg001", "role": "user",      "content": "How does React work?", "createdAt": "..." },
    { "_id": "msg002", "role": "assistant", "content": "React is a JavaScript library...", "createdAt": "..." },
    { "_id": "msg003", "role": "user",      "content": "What is JSX?", "createdAt": "..." }
  ]
}
```

✅ **Advantages:** One database query to get the chat AND all its messages. No JOINs or multiple queries. Messages and their chat always travel together — they're conceptually one unit.

⚠️ **Limitation:** MongoDB has a 16MB limit per document. For most conversations this is not a problem (a typical chat is a few KB), but an extremely long conversation could theoretically hit this limit.

**Option B — Referenced Documents (what we chose for user → chat relationship):**
```json
// In the chats collection:
{ "_id": "chat001", "user": "user123", "title": "..." }

// In the users collection:
{ "_id": "user123", "name": "Ankush", "email": "..." }
```

✅ **Advantages:** User data is stored once, not duplicated across every chat document. If the user changes their name, only one document needs updating. Chat documents stay small.

**The Rules of Thumb:**
- Embed when the "child" data is always needed with the "parent" data (messages are always needed with their chat)
- Embed when the child data is exclusively "owned" by the parent (messages belong to exactly one chat)
- Reference when the related data is shared across many documents (one user has many chats — store the user ID as a reference)
- Reference when the embedded data could grow unboundedly large

---

## Slide 10 — How Auth Uses the User Model

Let's trace through every auth operation to see exactly how the User model is used:

**Register — Creating a new user:**
```js
// Controller receives validated: { name: 'Ankush', email: 'a@test.com', password: '123456' }

// User.create() does three things:
// 1. Runs Mongoose schema validation (required fields, email format, min length, etc.)
// 2. Triggers the pre-save hook which hashes the password with bcrypt
// 3. Inserts the document into the 'users' collection in MongoDB
const user = await User.create({ name, email, password });
// user._id is auto-generated by MongoDB (a 24-char hex ObjectId)
```

**Login — Finding and verifying a user:**
```js
// .select('+password') is the magic that overrides select:false
// Without '+password', the returned object would have no password field
// and bcrypt.compare would fail with "undefined" vs hash
const user = await User.findOne({ email }).select('+password');

// bcrypt.compare takes the plain-text input and the stored hash
// Internally it re-hashes the input with the same salt and compares
// Returns true if they match, false if not
const isMatch = await bcrypt.compare(passwordFromRequest, user.password);
```

**Update profile — Updating just the name:**
```js
// findByIdAndUpdate: find by _id, apply the $set operation, return the NEW document
// { new: true } = return the updated document (not the one before update)
// { runValidators: true } = run schema validation rules on the update too
const user = await User.findByIdAndUpdate(
  req.user._id,
  { name: req.body.name.trim() },
  { new: true, runValidators: true }
);
// No need to call user.save() — findByIdAndUpdate handles the DB write directly
```

**Delete account — Cascading delete:**
```js
// IMPORTANT: Delete the user's chats FIRST, then delete the user
// If you delete the user first and the chat delete fails, you have orphaned chats
// with a reference to a non-existent user ID
await Chat.deleteMany({ user: req.user._id }); // Delete ALL chats belonging to this user
await User.findByIdAndDelete(req.user._id);     // Then delete the user themselves
```

---

## Slide 11 — How Chats Use the Chat Model

Let's walk through every chat operation:

**Create a new chat:**
```js
// Creates a new document in the chats collection
// user: req.user._id → stores the ObjectId of the logged-in user (from protect middleware)
// title: 'New Chat' → default title; will be updated after first message
// messages: [] → starts with an empty messages array
const chat = await Chat.create({
  user: req.user._id,
  title: 'New Chat',
  messages: [],
});
// Response: the full chat document with its new _id, ready for the frontend to use
```

**List all chats with search:**
```js
const { search } = req.query; // e.g. GET /api/chats?search=react

const filter = { user: req.user._id }; // Always filter by the logged-in user (security!)

if (search && search.trim()) {
  // Escape special regex characters in the search term to prevent regex injection
  const searchRegex = new RegExp(
    search.trim().replace(/[.*+?^${}()|[\]\\]/g, '\\$&'), // escape special chars
    'i' // 'i' flag = case-insensitive search
  );
  // $or means match if EITHER title OR message content contains the search term
  filter.$or = [
    { title: searchRegex },
    { 'messages.content': searchRegex }, // dot notation to search inside embedded array
  ];
}

// .sort({ updatedAt: -1 }) → most recently updated chats appear first (like ChatGPT sidebar)
// .select('_id title updatedAt') → only return these 3 fields, NOT the messages array
// This keeps the response small — we load full messages only when a chat is opened
const chats = await Chat.find(filter).sort({ updatedAt: -1 }).select('_id title updatedAt');
```

**Send message — The key operation:**
```js
// Push user message into the embedded messages array
chat.messages.push({ role: 'user', content: trimmedContent });

// Build the full history — all messages so far — to give the AI context
const messagesForAI = chat.messages.map(m => ({ role: m.role, content: m.content }));

// Call Groq AI with the full history
const aiResponse = await getAIResponse(messagesForAI);

// Push AI response into the same messages array
chat.messages.push({ role: 'assistant', content: aiResponse });

// Auto-title the chat from the first user message (when we have exactly 2 messages)
if (chat.messages.length === 2) {
  chat.title = trimmedContent.slice(0, 50) + (trimmedContent.length > 50 ? '...' : '');
}

// Save the entire chat document back to MongoDB (with the new messages)
await chat.save();
```

---

## Slide 12 — Mongoose Operations Cheat Sheet

Here are the most commonly used Mongoose methods with a plain-English explanation of each:

| Operation | What it does | Example |
|-----------|-------------|---------|
| `Model.create(data)` | Insert a new document | `User.create({ name, email, password })` |
| `Model.find(filter)` | Find ALL matching documents | `Chat.find({ user: req.user._id })` |
| `Model.findOne(filter)` | Find ONE matching document (first match) | `User.findOne({ email })` |
| `Model.findById(id)` | Find by MongoDB's `_id` field | `User.findById(req.user._id)` |
| `Model.findByIdAndUpdate(id, data, options)` | Find by `_id`, apply update, return updated doc | `User.findByIdAndUpdate(id, { name }, { new: true })` |
| `Model.findByIdAndDelete(id)` | Find by `_id` and delete it | `User.findByIdAndDelete(req.user._id)` |
| `Model.findOneAndDelete(filter)` | Find by any filter and delete it | `Chat.findOneAndDelete({ _id: id, user: userId })` |
| `Model.deleteMany(filter)` | Delete ALL matching documents | `Chat.deleteMany({ user: req.user._id })` |
| `.sort({ field: -1 })` | Sort results (-1 = descending, 1 = ascending) | `.sort({ updatedAt: -1 })` |
| `.select('a b -c')` | Include/exclude specific fields (+name to include, -name to exclude) | `.select('_id title updatedAt')` |
| `.populate('field')` | Replace an ObjectId reference with the full document | `.populate('user', 'name email')` |

---

## Slide 13 — Indexes and the Unique Email Constraint

**What is an index?**
An index in MongoDB is a special data structure that stores a sorted copy of specific field values. When you query by an indexed field, MongoDB uses the index like a book's index — jumping directly to the result instead of reading every page.

**Our unique email index:**
When you declare `unique: true` on a field in Mongoose, it automatically tells MongoDB to create a unique index on that field. This index enforces two things:
1. **Speed** — Looking up a user by email is extremely fast regardless of how many users exist
2. **Uniqueness** — MongoDB rejects any insert/update that would create a duplicate value

**What happens when a duplicate is attempted:**
```js
// First user — succeeds
await User.create({ email: 'ankush@test.com', name: 'Ankush', password: '123456' });

// Second user with same email — MongoDB throws an error
await User.create({ email: 'ankush@test.com', name: 'Someone Else', password: 'abcdef' });
// Error: MongoServerError: E11000 duplicate key error collection: classmentor.users
//        index: email_1 dup key: { email: "ankush@test.com" }
// error.code === 11000

// Our controller catches this:
if (error.code === 11000) {
  return res.status(400).json({ message: 'User already exists with this email' });
}
```

---

## Slide 14 — Collections in Our Database

When we use Mongoose to create models, MongoDB automatically creates the corresponding collections. Here's what our database looks like:

| Collection | Created by | What it stores |
|------------|------------|----------------|
| `users` | `mongoose.model('User', ...)` | All registered user accounts |
| `chats` | `mongoose.model('Chat', ...)` | All conversations with embedded message arrays |

**Mongoose auto-pluralises and lowercases the model name:**
- `mongoose.model('User', ...)` → creates the `users` collection
- `mongoose.model('Chat', ...)` → creates the `chats` collection
- `mongoose.model('BlogPost', ...)` → would create the `blogposts` collection

**Our full data model visualised:**

```
Database: classmentor
│
├── Collection: users
│   └── Document: { _id, name, email, password, createdAt, updatedAt }
│
└── Collection: chats
    └── Document: {
          _id,
          user: ObjectId → users._id,   ← reference
          title,
          messages: [                    ← embedded array
            { _id, role, content, createdAt, updatedAt },
            { _id, role, content, createdAt, updatedAt },
            ...
          ],
          createdAt,
          updatedAt
        }
```

---

## Slide 15 — Mongoose Schema Validation in Action

Mongoose validates your data BEFORE saving to MongoDB. If any rule fails, it throws a `ValidationError` and nothing is written to the database. Here's what that looks like:

```js
// Attempting to create a user with missing or invalid fields:
try {
  await User.create({
    name: '',          // Empty after trimming → fails required validation
    email: 'not-valid', // Not an email format → but we handle this in express-validator
    password: '123',   // Only 3 chars → fails minlength:6
  });
} catch (error) {
  console.log(error.name); // 'ValidationError'
  console.log(error.errors.name.message); // 'Please provide a name'
  console.log(error.errors.password.message); // 'Path `password` is shorter than...'
}

// How our controller handles ValidationError:
if (error.name === 'ValidationError') {
  // Get the first error message from the error object
  const msg = error.errors[Object.keys(error.errors)[0]].message;
  return res.status(400).json({ message: msg });
}
```

**Why do we have BOTH express-validator AND Mongoose validation?**
- **express-validator** catches problems at the HTTP layer — before any database operations happen. This is fast and gives friendly error messages.
- **Mongoose validation** is the last safety net at the database layer — it catches anything that slipped through or was set programmatically.

---

## Slide 16 — Environment Variable for the Database

```bash
# server/.env
MONGODB_URI=mongodb+srv://prodUser:StrongPass123@cluster0.abc123.mongodb.net/classmentor
```

**Step-by-step guide to setting up MongoDB Atlas (free):**
1. Go to [mongodb.com/cloud/atlas](https://www.mongodb.com/cloud/atlas) and create a free account
2. Create a new **Project** (call it "classMentor")
3. Build a new Cluster → choose **M0 Free Tier** → pick any region close to you
4. Create a **Database User**: Security → Database Access → Add New Database User
   - Choose "Password" authentication
   - Set a username and strong password
   - Role: "Read and write to any database"
5. Add your **IP to the allowlist**: Security → Network Access → Add IP Address
   - For development: "Add Current IP Address"
   - For production: add `0.0.0.0/0` (allow all — Render uses dynamic IPs)
6. **Get the connection string**: Deployment → Clusters → Connect → Drivers → Node.js
   - Copy the URI and replace `<password>` with your actual password

---

## Slide 17 — Key Concepts Covered Today

**✅ NoSQL Document Model**
MongoDB stores data as documents (JavaScript-like objects) in collections. No rigid table schema — each document can have its own shape, though Mongoose enforces consistency through schemas.

**✅ Mongoose Schema Design**
Defining field types (`String`, `ObjectId`), constraints (`required`, `unique`, `minlength`), and transformations (`lowercase`, `trim`, `default`) gives us automatic validation and consistent data.

**✅ `select: false` for sensitive fields**
The password field is excluded from all queries by default. You must explicitly opt in with `.select('+password')` when you actually need it — prevents accidental exposure.

**✅ Pre-save hooks**
Code that runs automatically before a document is saved. Used for password hashing — the controller just sets the plain password and the hook handles the bcrypt conversion.

**✅ Embed vs Reference**
Messages are embedded in Chat documents (always needed together, owned exclusively, not too large). Users are referenced in Chat documents (shared across many chats, need to stay DRY).

**✅ MongoDB indexes**
`unique: true` creates an index that speeds up queries by email and prevents duplicate accounts. Default `_id` field is always indexed.

**✅ cascading deletes**
When deleting an account, we use `Chat.deleteMany({ user: id })` before deleting the user to prevent orphaned documents in the database.

---

## Slide 18 — What's Next?

Today you added **The Memory** — persistent data that survives server restarts, deployments, and crashes.

**The full app is now working end-to-end:**
- ✅ Beautiful React UI (Masterclass 1)
- ✅ Powerful REST API + Groq AI (Masterclass 2)
- ✅ Persistent MongoDB storage (Masterclass 3)

But it's still only running on your laptop. Sharing a `localhost` URL doesn't work for anyone else. That's what we fix in the final masterclass.

**Next → Masterclass 4: The Launch**

> We'll deploy the frontend to Vercel and the backend to Render — making the app accessible to anyone in the world with a browser.

```bash
git add .
git commit -m "Masterclass 3 complete"
git tag masterclass-3
git push origin main
git push origin masterclass-3
```

---

## Slide 19 — Q&A / Recap

> What did we build in this masterclass?

1. **MongoDB Atlas setup** — free cloud database cluster with proper user and IP configuration
2. **Mongoose connection** — with graceful error handling and process exit if connection fails
3. **User model** — schema validation, unique email index, password hidden by default, auto-hash via pre-save hook
4. **Chat model** — user reference (ObjectId), title with default, embedded messages array with role enum
5. **Full CRUD operations** — create, read (with search), update (profile name), delete (with cascade)
6. **Embed vs Reference design decision** — choosing the right relationship type for each case
7. **Error handling** — duplicate key (code 11000), ValidationError, and not-found scenarios

**Questions?** 🚀
