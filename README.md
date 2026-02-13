# MovieRaterDB – Sparse Matrix Rating Database

**A custom movie rating database using an orthogonal sparse matrix with O(K) memory and similarity search over reviewers and movies.**

> ⚠️ **Source code is private per course policy.** Available upon request.

## Contact

**For source code access or questions:**
- Source Code: https://github.com/ahmadbitaarr/movieraterdb-private
- Email: ahmadbitar2005@gmail.com
- LinkedIn: www.linkedin.com/in/ahmad-f-bitar

---

## Highlights

- **Orthogonal sparse matrix** with sorted row (reviewer) and column (movie) headers
- **O(K) memory** where K = number of ratings (not R×M theoretical matrix size)
- **Bidirectional pointer structure** enabling row and column traversal without auxiliary collections
- **Similarity algorithms** compute reviewer/movie similarity via average absolute difference on overlapping ratings
- **Comprehensive unit testing** validates structural invariants, edge cases, and deletion safety

**Tech:** Java • Custom pointer-based data structure • Unit testing

---

## Table of Contents

- [Getting Started](#getting-started)
- [Problem & Motivation](#problem--motivation)
- [Architecture & Design](#architecture--design)
- [Similarity Algorithms](#similarity-algorithms)
- [Testing](#testing)
- [What I Learned & Challenges](#what-i-learned--challenges)
- [Future Improvements](#future-improvements)

---

## Getting Started

### Prerequisites
- Java 8+
- Standard JDK (no external dependencies)

### Compile & Run
```bash
javac *.java
java MovieRater
```

### Example Usage & Output
```java
// Add ratings: addReview(reviewerID, movieID, rating)
addReview(1, 10, 5);
addReview(2, 10, 4);
addReview(1, 12, 3);
addReview(3, 10, 2);

// Find most similar reviewer to reviewer 1
similarReviewer(1);
// → Returns reviewer 2 (both rated movie 10; |5-4| = 1.0 average difference)

// Find most similar movie to movie 10
similarMovie(10);
// → Returns movie 12 (both rated by reviewer 1; |5-3| = 2.0 average difference)
```

All operations execute **directly on the sparse structure**—no flattening or temporary collections.

---

## Problem & Motivation

Movie rating systems naturally form a sparse matrix where **rows = reviewers**, **columns = movies**, and **cells = ratings**. Most reviewer–movie pairs are empty.

**Why not use standard approaches?**

- **Dense 2D array:** Wastes O(R×M) memory for mostly-null cells; doesn't scale
- **HashMap<Reviewer, HashMap<Movie, Rating>>:** Simplifies coding but abstracts away pointer mechanics and sorted-order reasoning

**Goal:** Build the underlying structure manually to understand sparse matrix pointer manipulation, maintain sorted invariants, and achieve O(K) space where K = actual ratings.

---

## Architecture & Design

### Data Structure

**Orthogonal linked sparse matrix** with:
- **Reviewer header nodes** (sorted linked list)
- **Movie header nodes** (sorted linked list)
- **Rating nodes** with four pointers:
  - `nextRow` / `prevRow` – horizontal links across a reviewer's movies
  - `nextCol` / `prevCol` – vertical links across a movie's reviewers

### Structural Invariants

1. Reviewer and movie headers remain **sorted** at all times
2. **No duplicate ratings** (one rating per reviewer-movie pair)
3. **Automatic header cleanup** – empty headers removed when last rating deleted
4. **No dangling references** – all pointers updated correctly during insertion/deletion

**Result:** Traversing a reviewer's movies or a movie's reviewers only visits actual ratings.

### Complexity

| Operation | Time | Space |
|-----------|------|-------|
| Memory | — | **O(K)** where K = number of ratings |
| Add/Delete | O(R + M) worst-case | Traverses headers to maintain sorted order |
| Similarity | O(overlap) | Only visits shared ratings |

The system **scales with real data**, not theoretical matrix dimensions.

---

## Similarity Algorithms

### Reviewer Similarity
**Goal:** Find the reviewer most similar to a given reviewer based on overlapping movie ratings.

**Method:**
1. Traverse reviewer's rated movies
2. For each overlapping rating, compute `|rating1 - rating2|`
3. Return reviewer with minimum average absolute difference
4. Deterministic tie-breaking (first in sorted order)

### Movie Similarity
**Goal:** Find the movie most similar to a given movie based on overlapping reviewer ratings.

**Method:**
- Same logic as reviewer similarity, but traverse movie's reviewers instead
- Operates entirely on sparse links—no auxiliary data structures

**Example:**
```
Reviewer 1 rates movies [10:5, 12:3]
Reviewer 2 rates movies [10:4, 15:5]

similarReviewer(1) → Reviewer 2
  Overlap: movie 10
  Difference: |5 - 4| = 1.0
```

---

## Testing

Extensive unit tests validate both **correctness** and **structural integrity** (90%+ mutation coverage):

- **Empty matrix behavior** – operations on structure with no ratings
- **Single-node edge cases** – adding/removing the only rating
- **Header cleanup** – verifying headers deleted when last rating removed
- **Sorted traversal** – confirming headers and links maintain order
- **Similarity edge cases** – handling no overlap, ties, single shared rating
- **Invalid input** – rejecting malformed reviewer/movie IDs

Tests confirm the sparse structure maintains invariants across all operations.

---

## What I Learned & Challenges

**Key Learnings:**
- **Pointer invariant maintenance:** Ensuring four-way linking (nextRow, prevRow, nextCol, prevCol) stays consistent during concurrent updates
- **Sorted insertion/deletion:** Maintaining sorted headers while inserting/deleting nodes without breaking chains
- **Sparse traversal algorithms:** Designing similarity logic that works directly on pointer links rather than materializing arrays
- **Test-driven invariant validation:** Writing tests that verify structural properties (no dangling refs, sorted order) beyond input/output correctness

**Challenges:**
- **Deletion complexity:** Safely removing a rating node requires updating up to six pointers while preserving sorted order and cleaning up empty headers
- **Tie-breaking determinism:** Ensuring similarity returns consistent results when multiple reviewers/movies have identical overlap scores
- **Edge case coverage:** Handling corner cases like single-rating matrices, zero-overlap similarity, and header-only structures

This project deepened my understanding of **low-level data structure design** and the tradeoffs between abstraction and structural control.

---

## Future Improvements

- [ ] **Cosine similarity** – switch from absolute difference to cosine-based metrics
- [ ] **Weighted similarity** – incorporate rating frequency or recency
- [ ] **Persistence layer** – serialize/deserialize sparse matrix to disk
- [ ] **REST API wrapper** – expose operations via HTTP endpoints
- [ ] **Performance benchmarking** – compare against HashMap and dense array implementations at scale

---

*Built with Java*
