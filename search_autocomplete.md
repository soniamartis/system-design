## Concepts
- Trie DS
- Redis data structures like sorted set and list
- Autocomplete for other languages using Unicode character set instead of ASCII(used for english characters)
- Real world autocomplete design : Prefixy

---
## Terminology of a autocomplete engine


----
## Functional Requirements
- Match beginning of the input search query
- Support only englisg characters
- Auto correct is out of scope
- Return 5 top k suggestions
- 10 million DAU

## Non functional requirements
- Fast response times
- Sorted by popularity
- Highly available

---
## Back of the envelope estimations

10 million DAU, assume each user runs 10 searches. Avg size of each search(# of words in the input search is 20), QPS = 20*10*10 * 10 ^6 = 20,000 
Peak QPS = 2*QPS

## Naive approach



 
