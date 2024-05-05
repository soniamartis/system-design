## Concepts
- Trie DS
- Prefix Hash Tree
- Redis data structures like sorted set and list
- Autocomplete for other languages using Unicode character set instead of ASCII(used for english characters)
- Real world autocomplete design : Prefixy

---
## Terminology of an autocomplete engine
- prefix: what a user has typed so far
- completions: all possible ways a given prefix can be completed in order to form a word or phrase
- suggestions: the top ranked completions which will be presented to the user
- score: an integer representing the popularity of a given completion
- selection: the suggestion that the user chooses or a new completion that the user submits

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

---
## Naive approach
- We can use a relational DB to store every input word and increment count of that word whenever we get a user query for it
- For returning top k, we can run a sql query that will return top k suggestions based on frequency of each word
- This solution will not scale well

table structure:
query | frequency

query:
```
select * from word_frequency_table
where query like 'prefix%'
order by frequency desc
limit 5
```

## Better approaches
### Option 1: Trie
Tries are good fit for prefix-based search, so we could consider it as one of the possible solutions:
- L = length of prefix
- N = # of nodes in trie
- K = no of suggestions to return to user

 ![image](https://github.com/soniamartis/system-design/assets/12456295/54acede4-3dc1-434b-a9e9-75e4d8254f9a)

- Time complexity of returning the top k = O(L) + O(N) + O(KlogK)
  Where O(N) is the time taken to get all possible completions for the prefix
  This can be optimised by precomputing and storing all the completions for every prefix
  This will occupy significant space though
  We can reduce the amount of space occupied by defining some limits
  - Limit the length of prefix: this means smaller number of leaf nodes as we will not store any character nodes greater than this predefined length
  - Limit the number of completions stored at each node to just K, since we will be returning only K completions per prefix

- Trie optimised for returning completions in O(1) time
- L and K are now constant
  
![image](https://github.com/soniamartis/system-design/assets/12456295/7e0492fe-881d-4a53-ac4e-240f2767e1b4)

 
### Option 2: Prefix Hash Tree
- We can leverage the hash table structure to hold all the prefixes along with the top K completions for each prefix
- This will return results in O(1) time
- Writes will be slower


![image](https://github.com/soniamartis/system-design/assets/12456295/a0c207fe-20fe-4648-bb1a-5be622e5f4ca)

Time complexities of the 2 approaches:

![image](https://github.com/soniamartis/system-design/assets/12456295/81957713-699e-44f0-9c4a-3f791f3bc270)

----
### 


