---
status: new
tags: [crdt, yjs, collaboration, real-time, distributed-systems, ot, conflict-resolution]
source: Y.js docs, CRDT papers, "CRDTs and the Quest for Distributed Consistency" by Marc Shapiro, various deep-dives
created: 2026-06-05
difficulty: intermediate
---

# 🧠 Topic: Y.js and CRDT - Collaborative Editing

> [!abstract] Executive Summary (ELI5)
> Imagine you and your friend are both drawing on the same coloring book, but you're in different rooms and can only pass messages through a tiny hole in the wall. CRDT is the **magical coloring book** that makes sure both drawings end up looking good **no matter what order the crayon strokes arrive** — even if you both color the same spot at the same time, the book figures out a way to keep both colors visible. Y.js is the friendly elf that carries those crayon messages through the hole super fast.

---

## 🎯 The "Why"

### Problem Solved
You're building a Google Docs clone. Alice types "Hello" on her laptop in Tokyo. Bob types "World" on his phone in London. At the exact same millisecond.

**The old way (before CRDT):**
1. Alice's server says: "The document is `Hello`"
2. Bob's server says: "The document is `World`"
3. They fight. Alice wins. Bob's "World" vanishes into the void.
4. Bob rage-quits your app and leaves a 1-star review.

**The CRDT way:**
1. Alice adds "Hello"
2. Bob adds "World"  
3. The magic happens. Both changes survive.
4. Document becomes either "Hello World" or "World Hello" (deterministically — the same on both sides)
5. Bob and Alice are happy. You get VC funding.

**The core problem CRDTs solve:** How do multiple people edit the same thing simultaneously **without a central referee**, without locking anyone out, and without losing anyone's work?

### Alternatives
- **Operational Transformation (OT)** — what Google Docs originally used. Think of it as a very smart translator: "Alice typed 'a' at position 3. But wait, Bob just typed 'b' at position 2, so Alice's 'a' is now at position 4." OT requires a central server to coordinate. Complex, fragile, and the reason Google Docs sometimes glitches.
- **Central server with locks** — "Only one person can edit at a time." Like a bathroom with a lock. Simple but annoying.
- **Last-Write-Wins (LWW)** — "Whoever saved last wins." Like two people editing a shared text file in Dropbox. Painful. Data loss guaranteed.
- **CRDTs (Conflict-free Replicated Data Types)** — "Everyone's changes merge automatically, like a magical document." No central server needed. Works offline. Always consistent.

### Trade-offs
| Approach | Needs Server? | Offline? | Complexity | Data Loss? |
|----------|--------------|----------|------------|------------|
| OT | Yes (central) | No | Very High | Possible |
| Central Locks | Yes | No | Low | No, but blocked |
| Last-Write-Wins | Optional | Kinda | Low | Yes, guaranteed |
| **CRDT** | **No** | **Yes** | **Medium** | **No** |

> [!tip] Senior Insight
> CRDTs trade **central coordination** for **local autonomy**. Every client is a fully capable replica. This is why they work offline — your phone is not a second-class citizen waiting for the server to approve its edits.

---

## 📜 History

| Year | Milestone |
|------|-----------|
| **1989** | The first collaborative editors appear. They use locks. It is terrible. |
| **1995** | Operational Transformation (OT) is invented by Ellis and Gibbs. Smart but complex. |
| **2000s** | Google Docs adopts OT. Thousands of engineers cry maintaining it. |
| **2006** | **CRDTs are born!** Marc Shapiro and colleagues at INRIA formalize "Conflict-free Replicated Data Types." The academic world rejoices. Engineers don't notice yet. |
| **2011** | Shapiro publishes the definitive CRDT survey paper. The term enters mainstream distributed systems vocabulary. |
| **2015-2018** | CRDTs start appearing in real products: Riak (database), Redis, various research editors. |
| **2019** | **Kevin Jahns releases Y.js** — a production-ready CRDT library for JavaScript. Suddenly, adding real-time collaboration to web apps becomes possible for mere mortals. |
| **2020-2022** | Y.js explodes in popularity. Used by: Excalidraw, Relm, AFFiNE, Hocuspocus, various Figma clones, and countless startups. |
| **2023-2025** | CRDTs go mainstream. AutoMerge, Yjs, Electric SQL. Even large companies start migrating from OT to CRDTs for new features. |

---

## 🏗️ What CRDTs Are Made Of

### The Core Idea: Math Makes Conflicts Disappear

CRDTs are based on a simple but brilliant mathematical insight:

> If every operation is **associative**, **commutative**, and **idempotent** (or a subset of these), then operations can be applied in **any order** and still produce the **same result**.

Don't panic. Let's translate this into human language with cats.

### Analogy: The Cat Treat Bag 🐱

You have a bag of cat treats. Two roommates both add treats to it at the same time.

```
Alice adds: [🐟, 🐟]
Bob adds:   [🍗, 🍗]
```

**Associative:** `(Alice's treats + Bob's treats) + Charlie's treats` = `Alice's treats + (Bob's treats + Charlie's treats)`
> The bag doesn't care in what groups you add treats. The total is the same.

**Commutative:** `Alice's treats + Bob's treats` = `Bob's treats + Alice's treats`
> It doesn't matter who added treats first. The bag has the same treats.

**Idempotent:** `Alice adds [🐟]` then `Alice adds [🐟]` = `Alice adds [🐟]` once
> If Alice accidentally adds the same treat twice (duplicate message), the bag doesn't explode with extra treats.

If your data structure has these properties, congratulations — it's a CRDT! Conflicts are mathematically impossible.

### Types of CRDTs

There are two main families:

#### 1. State-based CRDTs (Convergent Replicated Data Types)
Everyone shares their **entire state**. Like showing your whole cat treat bag to your roommate.

```javascript
// State-based: "Here is my entire document"
const myState = {
  content: "Hello World",
  version: 42
};
// Merge function combines two states into one
const merged = merge(myState, roommatesState);
```

**Pros:** Simple to reason about. Works over any network (even email!).
**Cons:** Can be large. If your document is a novel, you're sending the novel every time.

#### 2. Operation-based CRDTs (Commutative Replicated Data Types)
Everyone sends **only the changes** (deltas). Like telling your roommate: "I added a fish treat." Not the whole bag.

```javascript
// Op-based: "I added the letter 'a' at position 3"
const operation = {
  type: 'insert',
  position: 3,
  character: 'a',
  timestamp: '...'
};
```

**Pros:** Tiny messages. Efficient for real-time collaboration.
**Cons:** Requires reliable broadcast (no lost messages). More complex to implement.

> [!tip] Y.js Hybrid Approach
> Y.js uses a **hybrid**: it sends small delta updates in real-time, but can also sync full state when needed. Best of both worlds.

---

## 🎭 Y.js: The Friendly CRDT Library

Y.js is not a CRDT itself — it is a **library that implements CRDTs** for JavaScript/TypeScript. It gives you collaborative data structures that feel like normal JS objects, but secretly do CRDT magic under the hood.

```javascript
import * as Y from 'yjs';

// Create a document — this is your shared coloring book
const doc = new Y.Doc();

// Create a collaborative text (like a shared string)
const ytext = doc.getText('my-text');

// Alice types
ytext.insert(0, 'Hello');

// Bob types (on a DIFFERENT device, different Y.Doc)
const doc2 = new Y.Doc();
const ytext2 = doc2.getText('my-text');
ytext2.insert(0, 'World');

// Magical merge — both changes survive!
const update1 = Y.encodeStateAsUpdate(doc);
const update2 = Y.encodeStateAsUpdate(doc2);

Y.applyUpdate(doc, update2);
Y.applyUpdate(doc2, update1);

console.log(doc.getText('my-text').toString()); // "HelloWorld" or "WorldHello"
// But BOTH docs have the SAME text — guaranteed!
```

### Y.js Data Structures

Y.js gives you collaborative versions of familiar data types:

| Y.js Type | Normal JS Equivalent | What It Does |
|-----------|---------------------|--------------|
| `Y.Text` | `string` | Collaborative text with rich formatting (like a mini Google Doc) |
| `Y.Array` | `Array` | Ordered list where everyone can push/pop/splice |
| `Y.Map` | `Map` / `Object` | Key-value store where last-writer-wins per key |
| `Y.XmlFragment` | DOM fragment | Collaborative XML/HTML (for rich text editors) |
| `Y.Doc` | — | The container that holds all shared types |

### Y.Text: The Star of the Show

```javascript
import * as Y from 'yjs';

const doc = new Y.Doc();
const ytext = doc.getText('editor-content');

// Insert text at position
ytext.insert(0, 'Hello ');
ytext.insert(6, 'World');

// Insert with formatting!
ytext.insert(6, 'beautiful ', {
  bold: true,
  color: '#ff0000'
});

console.log(ytext.toString()); // "Hello beautiful World"
console.log(ytext.toDelta());  // [{ insert: 'Hello ' }, { insert: 'beautiful ', attributes: { bold: true, color: '#ff0000' } }, { insert: 'World' }]
```

The `toDelta()` format is compatible with **Quill**, **Slate**, **ProseMirror**, and other rich text editors.

### The Y.Doc Subdocuments System

```javascript
// You can nest documents inside documents
const rootDoc = new Y.Doc();
const subDoc = new Y.Doc();

// Subdocuments are lazy-loaded — great for large docs with many pages
rootDoc.getMap('pages').set('page-1', subDoc);
```

### How Y.js Handles Network

Y.js itself is **network-agnostic**. It gives you updates as binary blobs. You transport them however you want:

```javascript
// Get binary update
const update = Y.encodeStateAsUpdate(doc);

// Send via WebSocket
websocket.send(update);

// Or via WebRTC (P2P!)
webrtcDatachannel.send(update);

// Or save to disk
fs.writeFileSync('document.yjs', update);

// Or email to your grandma
// (State-based CRDTs work over ANY transport!)
```

#### Providers: Ready-Made Network Solutions

Y.js has official providers for common transports:

```javascript
// WebSocket provider (most common)
import { WebsocketProvider } from 'y-websocket';
const wsProvider = new WebsocketProvider(
  'wss://demo.yjs.dev',
  'my-document-room',
  doc
);

// WebRTC provider (P2P, no server needed!)
import { WebrtcProvider } from 'y-webrtc';
const rtcProvider = new WebrtcProvider(
  'my-document-room',
  doc,
  { signaling: ['wss://signaling.yjs.dev'] }
);
```

**Providers handle:**
- Connecting to other users
- Broadcasting updates
- Awareness (who is online, where are their cursors)
- Persistence (saving to database)

### Awareness: Knowing Who's Typing Where

```javascript
// Get the awareness API from your provider
const awareness = wsProvider.awareness;

// Set your local state (cursor position, name, color)
awareness.setLocalStateField('user', {
  name: 'Alice',
  color: '#ff0000',
  cursor: { index: 5, length: 0 }
});

// Listen to other users' states
awareness.on('change', () => {
  const users = Array.from(awareness.getStates().values());
  users.forEach(state => {
    console.log(`${state.user.name} is at position ${state.user.cursor.index}`);
  });
});
```

This is how Figma shows other people's cursors in real time!

---

## 🛠️ Core Principles & How It Actually Works

### The Y.js Internal Structure: A Forest of Items

Under the hood, Y.js does NOT store text as a simple string. It stores a **doubly-linked list of items**, where each item has:
- A unique ID (client ID + clock)
- Content (the actual text/element)
- Origin (the ID of the item before it)
- Right origin (the ID of the item after it)
- Deleted flag

```
┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐
│ ID: 1@0 │───→│ ID: 2@1 │───→│ ID: 1@2 │───→│ ID: 3@0 │
│ "H"     │←───│ "e"     │←───│ "l"     │←───│ "l"     │
└─────────┘    └─────────┘    └─────────┘    └─────────┘
      │
      ▼
┌─────────┐
│ ID: 2@0 │
│ "o"     │   ← Bob inserted "o" concurrently with Alice's "l"
└─────────┘
```

When two users insert at the same position, Y.js uses the **client IDs** to deterministically decide the order. Lower client ID goes first. This is why conflicts are mathematically resolved — no ambiguity.

### The Insertion Algorithm (Simplified)

```
Alice's doc: "Hell" (items: H-e-l-l)
Bob's doc:   "Hell" (items: H-e-l-l)

Both try to insert 'o' after the last 'l':

Alice inserts 'o' → creates item with ID (alice-client-id, clock=5)
                    → origin = ID of last 'l'
                    → sends update to Bob

Bob inserts 'o'   → creates item with ID (bob-client-id, clock=5)  
                    → origin = ID of last 'l'
                    → sends update to Alice

On Alice's machine:
  - Her 'o' already there
  - Receives Bob's 'o': checks origin, sees same position
  - Compares IDs: alice-id vs bob-id
  - Deterministic rule: lower ID wins, or some other tie-breaker
  - Both 'o's are kept, ordered consistently

On Bob's machine:
  - His 'o' already there
  - Receives Alice's 'o': same logic
  - SAME result as Alice's machine!

Both docs end up identical: "Hello" or "Hello" with two 'o's (depending on the tie-breaker rule)
```

> [!warning] Important
> Y.js actually KEEPS both items (even if they have the same content) unless you explicitly delete. Deletion is also a CRDT operation — items are marked as deleted, not actually removed. This is crucial for undo/redo!

### Deletion Is Also a CRDT Operation

```javascript
// "Soft delete" — the item stays but is marked as deleted
ytext.delete(0, 5); // Delete 5 characters starting at position 0

// Under the hood, Y.js sends:
const deleteOp = {
  type: 'delete',
  targetId: 'item-id-to-delete',
  clock: 42
};

// Why soft delete?
// 1. Undo/redo works (just unmark as deleted)
// 2. If someone inserts at position 0 concurrently, 
//    we know WHERE their insertion belongs relative to the deleted item
// 3. Deleted items don't show in toString(), but they're in the CRDT
```

### Undo/Redo with Y.UndoManager

```javascript
import { UndoManager } from 'yjs';

const undoManager = new UndoManager(ytext);

ytext.insert(0, 'Hello'); // Type something
undoManager.undo();       // Magically removes 'Hello'
undoManager.redo();       // Magically brings it back

// The undo manager tracks operations and can reverse them
// even across multiple users!
```

---

## ⚠️ Pitfalls & Anti-patterns

### 1. Forgetting Y.js Is Eventually Consistent

```javascript
// ❌ Don't expect immediate consistency
ytext.insert(0, 'Hello');
console.log(ytext.toString()); // "Hello" — OK, this is local

// But on another device:
// If the update hasn't arrived yet, they see ""
// If the network is slow, it might take seconds

// ✅ Always assume some delay for remote users
provider.on('synced', () => {
  console.log('Now we are in sync with the server');
});
```

### 2. Using Plain Objects Instead of Y Types

```javascript
// ❌ This will NOT sync
const doc = new Y.Doc();
const map = doc.getMap('state');
map.set('user', { name: 'Alice', age: 30 }); // Plain object!

// Changes to the nested object are NOT tracked
map.get('user').age = 31; // Silent mutation, no CRDT magic

// ✅ Use Y.Map for nested objects
const userMap = new Y.Map();
userMap.set('name', 'Alice');
userMap.set('age', 30);
map.set('user', userMap); // Now changes are tracked!
```

### 3. Not Using Awareness for Presence

```javascript
// ❌ Don't track "who is online" manually
const onlineUsers = new Set(); // This won't work across tabs/devices

// ✅ Use Y.js awareness
awareness.setLocalStateField('user', { name: 'Alice' });
// Automatically synced to all peers
```

### 4. Forgetting About Max Memory with Large Documents

```javascript
// ❌ CRDTs keep ALL history (including deleted items)
// A document edited for months can grow huge

// ✅ Use Y.encodeStateAsUpdate(doc) to periodically compact
// Or use a provider with persistence that handles this
```

### 5. Expecting Transactional Semantics

```javascript
// ❌ Y.js operations are NOT database transactions
const ymap = doc.getMap('data');

// This is TWO operations, not one atomic operation:
ymap.set('a', 1);
ymap.set('b', 2);

// Another user might see 'a' = 1 but 'b' is still old

// ✅ Use doc.transact() for atomic operations
doc.transact(() => {
  ymap.set('a', 1);
  ymap.set('b', 2);
});
```

### 6. Not Handling Binary Updates Correctly

```javascript
// ❌ Don't send Y.js updates as JSON
websocket.send(JSON.stringify(update)); // Binary data gets corrupted!

// ✅ Send as binary (ArrayBuffer/Uint8Array)
websocket.send(update); // update is already a Uint8Array
```

---

## 🔗 Deep Connections

- **Parent Topic:** [[Distributed Systems]], [[Real-time Collaboration]]
- **Related Concepts:**
  - [[Operational Transformation (OT)]] — the older, more complex alternative
  - [[Eventual Consistency]] — the consistency model CRDTs provide
  - [[WebSockets]] — common transport for Y.js updates
  - [[WebRTC]] — P2P transport for Y.js (no server!)
  - [[Quill Editor]] — rich text editor that works with Y.js
  - [[Slate.js]] — another editor framework with Y.js integration
  - [[ProseMirror]] — yet another rich text editor, also Y.js compatible
  - [[Figma]] — famous product using CRDT-like technology for real-time design
  - [[Automerge]] — another CRDT library (Rust + JS), alternative to Y.js
  - [[Electric SQL]] — database that uses CRDTs for sync
- **Prerequisites:**
  - JavaScript/TypeScript fundamentals
  - Understanding of async programming (promises, events)
  - Basics of network programming (WebSockets)
- **Next Steps:**
  - [[Automerge vs Y.js]] — detailed comparison of the two main CRDT libraries
  - [[Building a Collaborative Editor]] — practical guide with Y.js + React
  - [[CRDT Performance Optimization]] — handling large documents
  - [[Conflict Resolution Strategies]] — beyond CRDTs

---

## ❓ Active Recall Questions

> [!faq]- What problem do CRDTs solve that simple "last-write-wins" doesn't?
> Last-write-wins loses data when two people edit simultaneously. CRDTs preserve ALL changes by mathematically merging them, so no one's work is ever lost.

> [!faq]- What are the three mathematical properties that make CRDTs work?
> Associativity (grouping doesn't matter), Commutativity (order doesn't matter), and Idempotency (duplicates don't matter). These ensure all replicas converge to the same state regardless of the order operations arrive.

> [!faq]- What is the difference between state-based and operation-based CRDTs?
> State-based CRDTs share the entire data state (like showing your whole document). Operation-based CRDTs share only the changes/deltas (like saying "I added a letter"). Y.js uses a hybrid approach.

> [!faq]- Why does Y.js use soft deletes instead of actually removing items?
> Because soft deletes enable undo/redo, preserve positional information for concurrent insertions, and maintain the CRDT's mathematical properties. Deleted items don't show in output but stay in the internal structure.

> [!faq]- What is a Y.Doc and why do you need it?
> A Y.Doc is the container for all collaborative data structures. It handles the CRDT state, generates updates, and applies remote updates. Every peer has their own Y.Doc instance that converges with others.

> [!faq]- How does Y.js handle two users typing at the exact same cursor position?
> Y.js uses a deterministic tie-breaker (typically based on unique client IDs and logical clocks). Both insertions are kept and ordered consistently across all replicas using the same rule.

> [!faq]- What is the difference between CRDTs and Operational Transformation (OT)?
> OT transforms operations against each other at a central server to maintain order. CRDTs use mathematical properties so operations can be applied in any order locally without transformation. CRDTs are simpler, work offline, and don't need a central server.

> [!faq]- Why is Y.js "network-agnostic"?
> Because Y.js only produces binary update blobs. It doesn't care how they travel — WebSocket, WebRTC, HTTP, email, or carrier pigeon. You provide the transport layer.

> [!faq]- What does Y.js's Awareness API provide?
> It tracks ephemeral, non-persistent state like cursor positions, user names, selection ranges, and online status. Unlike document data, awareness state doesn't need to persist forever.

> [!faq]- What happens if you put a plain JavaScript object inside a Y.Map?
> Changes to the nested plain object are NOT tracked by the CRDT. You must use Y.Map or Y.Array for nested collaborative data, or the CRDT magic stops working.

> [!faq]- Why can CRDT documents grow very large over time?
> Because they keep all history — every insertion, every deletion, every edit — to maintain merge correctness. Deleted items are marked, not removed. Periodic compaction or state snapshots are needed for long-lived documents.

> [!faq]- What is `doc.transact()` used for in Y.js?
> It groups multiple Y.js operations into a single atomic transaction. Without it, other users might see partial changes (e.g., 'a' updated but 'b' not yet).

---

## 📝 References & Further Reading

- [Y.js Official Documentation](https://docs.yjs.dev/)
- [Y.js GitHub](https://github.com/yjs/yjs)
- [CRDTs and the Quest for Distributed Consistency](https://queue.acm.org/detail.cfm?id=2917756) — Marc Shapiro
- ["A comprehensive study of CRDTs"](https://hal.inria.fr/hal-00932836/file/CRDTs_SSS-2011.pdf) — Shapiro et al.
- ["Local-first software"](https://www.inkandswitch.com/local-first.html) — Ink & Switch (the manifesto for CRDT-powered apps)
- [Automerge Documentation](https://automerge.org/)
- [Y-Websocket Provider](https://github.com/yjs/y-websocket)
- [Y-Webrtc Provider](https://github.com/yjs/y-webrtc)
- [Hocuspocus](https://www.hocuspocus.dev/) — Y.js backend with persistence
- [Figma's Approach to Real-time Collaboration](https://www.figma.com/blog/how-figmas-multiplayer-technology-works/)
- [[Node.js - Architecture and Internals]] — if you need to build the WebSocket server
- [[Bun - Architecture and Internals]] — for a faster WebSocket server option
