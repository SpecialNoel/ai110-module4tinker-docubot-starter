# DocuBot Model Card

This model card is a short reflection on your DocuBot system. Fill it out after you have implemented retrieval and experimented with all three modes:

1. Naive LLM over full docs
2. Retrieval only
3. RAG (retrieval plus LLM)

Use clear, honest descriptions. It is fine if your system is imperfect.

---

## 1. System Overview

**What is DocuBot trying to do?**

DocuBot is trying to return the texts that are relevant to the input query. The returned texts are ordered by most relevant to least relevant to the query.

**What inputs does DocuBot take?**

Input for DocuBot will be user question (input query), docs in folder as a mini "database" used to retrieve related information.

**What outputs does DocuBot produce?**

Output of DocuBot will be sample output produced by a deployed llm (Mode 1), a list of filenames and paragraphs that are most relevant to the query (Mode 2), and the output produced by the deployed llm with the knowledge of filenames and paragraphs retrieved (Mode 3).

---

## 2. Retrieval Design

**How does your retrieval system work?**

- To convert documetns into an index, use a dictionary to store each word in documents as keys, and each filename whose file contains the word as values.
- For calculaing the score, add 1 point for every matching word in query to a given word list (text).
- To get top retrieval results, first get a list of filenames for which each file with its filename contained in this list has a query word contained in the file. Then, extract related paragraph instead of the whole document text. For each filename and paragraph, calculate and add the score along with the filename and paragraph to a result list. Sort the result list by highest score first. Add a railguard that returns an empty list if the very top score is lower than a threshold. Finally, return the top-k results from the result list.

**What tradeoffs did you make?**  
For example: speed vs precision, simplicity vs accuracy.

One tradeoff I made was that, the implementation on extracting relevant small pieces takes more code to write, but it is very effective in cutting down the size of relevant text returned.

---

## 3. Use of the LLM (Gemini)

**When does DocuBot call the LLM and when does it not?**

- Naive LLM mode: generate a sample answer without retreving information from provided "database".
- Retrieval only mode: retrieve information from provided "database" and rank in to produce top-k results.
- RAG mode: combining both mode 1 and 2 by giving the llm content about information retrieved in from "database".

**What instructions do you give the LLM to keep it grounded?**

The prompts uses snippets, and say "I do not know" as needed. When a matching result is found in mode 2, it will cite which file contains the query word.

---

## 4. Experiments and Comparisons

| Query                                               | Naive LLM: helpful or harmful? | Retrieval only: helpful or harmful? | RAG: helpful or harmful? | Notes |
| --------------------------------------------------- | ------------------------------ | ----------------------------------- | ------------------------ | ----- |
| Example: Where is the auth token generated?         | helpful                        | helpful                             | harmful                  |       |
| Example: How do I connect to the database?          | helpful                        | helpful                             | harmful                  |       |
| Example: Which endpoint lists all users?            | helpful                        | helpful                             | harmful                  |       |
| Example: How does a client refresh an access token? | helpful                        | helpful                             | harmful                  |       |

**What patterns did you notice?**

- The naive LLM look impressive but untrustworthy for the most of time.
- Retrieval only approach is clearly better when the database contains lots of docs that contains query words.
- RAG is unfortunately never clearly better than both.

---

## 5. Failure Cases and Guardrails

**Describe at least two concrete failure cases you observed.**

_Failure case 1:_

- Question: Where is the auth token generated?
- Mode 3 returns "I do not know based on the docs I have."
- It should return a llm-generated output enhanced with information retrieved from database.

_Failure case 2:_

- Question: How do I connect to the database?
- Mode 3 returns "I do not know based on the docs I have."
- It should return a llm-generated output enhanced with information retrieved from database.

**When should DocuBot say “I do not know based on the docs I have”?**

Question1: Where is the auth token generated?
Question2: How do I connect to the database?

**What guardrails did you implement?**

I implemented a threshold that if the score of the very top result sorted by the scoring system is less than the threshold, the model will return an empty list indicating that it knows nothing about the query.

---

## 6. Limitations and Future Improvements

**Current limitations**

1. Mode 1's output is very generic, meaning that it cannot get into the query in detail, and it does not provide sources it is based on.
2. Mode 2's scoring system only counts for matching words, meaning that "useless" words can affect the score dramatically.
3. Mode 3 has similar issues Mode 1 and Mode 2 have.

**Future improvements**

1. Revise the scoring system so that it puts less weights on unimportant words, and more weights on "elite" words.
2. Revise input sent to llm agent in mode 1 to ask for the source articles/documents it bases its answer on.

---

## 7. Responsible Use

**Where could this system cause real world harm if used carelessly?**

Careless usage of this system could give unexpected or wrong answers, as this system is incompetent without a large database for retrieving (unbiased) information.

**What instructions would you give real developers who want to use DocuBot safely?**

Guideline 1: take the output as a grain of salt only; do not trust the output completely.
Guideline 2: enhance the system by considering possible improvements listed above can greatly increase the trust-worthiness and the accuracy of returned results.

---
