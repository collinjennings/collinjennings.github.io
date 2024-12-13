---
layout: post
title:  "The Frame of Reference: Solving Detective Stories with AI - #1"
date:   2024-11-23 10:27:24 -0400
tags: ai genre
categories: blog
---

<img align="right" alt="Holme's Speckled Band" src="https://media.licdn.com/dms/image/D5612AQHA__wDIO-E5w/article-cover_image-shrink_600_2000/0/1687677533378?e=2147483647&v=beta&t=Zbsh4vgwQwMZZRhEwxdGpvPefp0eQUj1yrPHO1_LgO0">

Before the detective story, there was the tale of ratiocination. This is what Edgar Allan Poe called his trio of short stories featuring, C. Auguste Dupin, an amateur detective before there was the word _detective_. Readers of detective stories likely know that Dupin provided the model for Sherlock Holmes and Hercule Poirot. Like Holmes's "consulting detective" title, Dupin's amateur status provides him a degree of remove for considering crimes from a different perspective than the police. When the Paris police prefect consults Dupin in "The Purloined Letter," the officer explains that his men "search _every where_." He declares,

>to a properly trained police-agent, such a thing as a ‘secret’ drawer is impossible.

But from his distance, Dupin recognizes that the minister who has taken the letter knows that the police will search every potential hiding spot, so he leaves it in plain sight. The police treat every crime the same, but Dupin and his legion of amateur/consulting successors read the criminal not just the crime.

Readers of detective fiction quickly come to recognize this trope among many others. We tend to think of the genre as highly conventional, and like Dupin, when we consider potential suspects, we often settle on the culprit based on our knowledge of the genre rather than the details of the case. We know, for instance, that we can eliminate whomever the police initially arrest; that character is ruled out based on how he's treated by the story. 

I've been thinking about how we read detective stories in relation to a question about how large language models read more generally. This connection may seem strange, but it comes from the concept of attention. Attention is what allows Dupin to succeed where the Parisian police fail, and it is also what has produced the rapid advances in LLMs and AI in the last eight years. The basics of the story are that the 2017 article ["Attention is All You Need,"](https://arxiv.org/abs/1706.03762) by Vaswani et al. introduced the [transformer](https://en.wikipedia.org/wiki/Transformer_deep_learning_architecture) architecture, allowing models to process all the words of a text at once rather than in order. Models then iteratively assign and update the weights for measuring the relative importance of the words in determining what the text is saying. This method enabled researchers to train LLMs much more efficiently leading through the breakthroughs of ChatGPT and others.

But how does this computational measure of attention resemble or differ from the attention we pay to language when we read? To answer this question, I decided to run an experiment to see if a popular, commercial LLM tends to pay attention like Poe's detective or like the Parisian police. Can an LLM solve detective stories as well or better than human readers?

### Rationale
The goal for the experiment is to see to what extent we can observe patterns in how a prominent LLM reads stories as a proxy for better understanding how the model works--that is, how it pays attention. The easiest way to figure out what an LLM pays attention to is to examine what it was trained on, but that's not possible for commercial models and even if it were, the variety and volume of training data is so great that it would be difficult to do much with it. I wanted to try this experiment as an alternative approach to uncovering tendencies in how LLMs read stories.

I settled on using detective short stories for a few reasons. The primary reason is that predicting the culprit of a detective story seems to sit between the kinds of tasks that LLMS are trained on (question answering, summarizing, etc.) and literary interpretation. Of course, this is very different from developing a close reading, but making a prediction entails producing a theory of the text that draws on both the details of the fictional crime and the devices of the story. The second reaon for detective stories is that there are well developed corpora with extensive metadata for the genre. Finally, detective stories tend to be short enough to be processed efficiently by commercial LLMs and even some open-source ones with smaller token limits.



## Methods
I got the idea for the experiment after stumbling upon a collection of detective stories that was prepared with fascinating metadata. This is ["The Birth of the Modern Detective Story" dataset](https://github.com/ahmmnd/BMDS) produced by Adam Hammond et al. out of the University of Toronto. The corpus includes 435 stories published between 1891 and 1925. The metadata includes fields, such as `Reveal border sentence`, `Sufficient clues to solve?`, and `Correct annotator guess?`. These fields enable me, first, to crop the stories so that the model can only process the excerpt up until the reveal begins, and second, to compare the results of the model to those of the human annotators.

After discovering the corpus, I had to decide which LLM to use. I first tried out several open-source/free models available on Hugging Face. Because of the specific nature of the task, it seemed easiest to use an instruction-interface model for I which I could provide a detailed prompt. I found the [Mistral Instruct 7B model](https://huggingface.co/mistralai/Mistral-7B-Instruct-v0.1) to be the most efficient and accurate, although it struggled to process longer stories.

I tried ChatGPT 3.5 through the web interface on one story, but even with instructions to only use the provided text, it appeared to be drawing on outside knowledge -- its predictions were just too good.

I landed on [Google's Gemini model (specifically the 1.5 flash version)](https://blog.google/technology/ai/google-gemini-next-generation-model-february-2024/) because it processed the texts quickly, and its results indicated that it was **not** using information beyond the text. (At some point, I may try masking character names, but for now it seems unnecessary.)

For the experiment, I provide the following prompt:
```
qa_prompt = """Read the excerpt from a detective story, and predict who committed the crime.
The perpetrator is not identified in this portion of the story. The story may try to trick you with red herring clues.
Make sure to predict something that has not already happened. Then identify one key piece of evidence (consider material and testimonial clues) that supports your prediction."""
```
I tried to provide clear instructions without putting my thumb on the scale in terms of what the model should focus on in making its prediction. I suggest kinds of clues to consider and even certain genre features, but I have found that these instructions don't impede the model from grabbing for the most obvious clues. I combine the prompt with several correct examples that include the prediction and key piece of evidence. This few-shot prompting technique produces more accurate results, but it does prime the model to focus on certain kinds of evidence. 




```python
### Few-shot examples 
text = ''
prompt = f""" Given the fields "Context" and "Question", produce the fields "Prediction" and "Evidence".
---
"Context": {metaList[0]['text']},
"Question": {qa_prompt},
"Prediction": The perpetrator is the trainer John Straker himself, and Silver Blaze killed him by kicking the trainer in self-defense.
Evidence: The key clue is cataract knife which he took with him to
intentionally injure the horse, rather than for self defense.

---
"Context": {metaList[1]['text']},
"Question": {qa_prompt},
"Prediction": Arthur Pinner, the brother of Harry Pinner, is the perpetrator. He likely orchestrated the entire scheme to defraud
the firm Pycroft was hired to, and the suicide attempt was a final act of desperation when his plan was discovered.
"Evidence": The fact that both brothers have the same gold-filled tooth is a strong indicator that they are not who they claim
to be. This suggests a deliberate attempt to deceive Hall Pycroft.

---
"Context": {metaList[2]['text']},
"Question": {qa_prompt},
"Prediction":
"Evidence":

"""

```

After the model makes prediction and offers supporting evidence, I compare it to the response of the human annotators according to, 
- **Accuracy** - Was the prediction correct? Does the supporting evidence match the key clues identified by the annotator?
- **Solvability** - The annotators indicated whether the crime was "guessable," "solvable," and if they accurately guessed the culprit. I give 1 point for each field to create a solvability spectrum that goes from 0 to 3 as a way to measure how predictable the culprit is.

## Proof-of-concept
I've run the experiment on all the Sherlock Holmes stories in the corpus with results for 39 stories. (This excludes stories in which a crime isn't solved (e.g., “The Final Problem) or in which the identity of the culprit is known from the outset (e.g., "Mazarin Stone"). Gemini accurately predicted the culprit about 62% of the time compared to 26% percent for the human annotators. (I’m not very interested in this part of the comparison if only because I don’t think accurately guessing the perpetrator was a focus for the human annotators; it seems like it was incidental to the larger process of producing the metadata for the detective story corpus.) For the sake of space, I am going to wait until my next post to examine the full results. For now, I want to consider a couple examples that I’m using to develop a hypothesis regarding the results for the full corpus.

My working theory is that the model does well with stories that feature clear, “decodable” clues, and it struggles with ones that depend more upon genre conventions. I’m taking the term “decodable” from Franco Moretti’s essay, [“The Slaughterhouse of Literature,”](https://read.dukeupress.edu/modern-language-quarterly/article-abstract/61/1/207/19219/The-Slaughterhouse-of-Literature?redirectedFrom=fulltext) in which he organizes early detective stories on a spectrum relative to their use of clues, ranging from **present**, to **necessary**, to **visible**, to **decodable**. The model correctly predicts the culprit for all the stories in which Moretti identifies decodable clues. “The Adventure of the Speckled Band” is one of these stories. Here's the model's prediction:
```
Answer: {"Prediction": "Dr. Grimesby Roylott is the perpetrator. He used a snake to kill his stepdaughter Julia and is now attempting to kill Helen.", "Evidence": "The dying words of Julia, 'Oh, my God! Helen! It was the band! The speckled band!' strongly suggest that a snake was involved. The 'speckled band' likely refers to a venomous snake, and the 'band' could be a snake charmer's band or a group of snakes. The fact that Dr. Roylott has a passion for Indian animals, including a cheetah and a baboon, further supports the theory that he might have access to venomous snakes. The presence of the dog lash tied into a loop, which Holmes finds in Dr. Roylott's room, is a crucial piece of evidence. It suggests that the lash was used to control the snake, possibly to guide it through the ventilator and into Julia's room."}
```
This makes sense, and it resembles the human annotators key clue (`“The chair in the stepfather's room with the lash under it”`). The model seems to get to the right prediction via a route similar to human readers. However, I think the way the model is wrong is more interesting. The model seems to be terrible at reading for genre. As I remarked above, one of the first things a practiced human reader does when reading a detective story is to dismiss the suspect arrested by the police. Often how the story treats the characters is the most reliable way to predict the culprit regardless of the circumstantial evidence. In “Silver Blaze,” as Holmes briefs Watson on the details of the case, he remarks of the official investigator:
>‘Inspector Gregory, to whom the case has been committed, is an extremely competent officer. Were he but gifted with imagination he might rise to great heights in his profession. On his arrival he promptly found and arrested the man upon whom suspicion naturally rested.’

The suspect is Fitzroy Simpson, who had visited the horse stables near where the horse trainer John Straker was later found dead and the prize racehorse (the eponymous Silver Blaze) went missing. The most obvious clues point to Simpson, but Holmes has already in effect told the reader that he is innocent. Here's the model's prediction:
```
Answer: {"Prediction": "The perpetrator is Fitzroy Simpson. He likely killed Straker to cover up his theft of Silver Blaze.", "Evidence": "The key clue is the fact that Fitzroy Simpson's cravat was found in the dead man's hand. This suggests a struggle between the two, and that Simpson was present at the scene of the crime."}
```

What also makes “Silver Blaze” difficult for the model (and for readers) is that the story plays with the conventions of detective stories even as an early example of the genre. The culprit turns out to be the victim, Straker. Holmes explains during the reveal that the key clues are the cataract knife that Straker had with him and a receipt found in his house showing the purchase of expensive women’s clothing under a different name. It turns out that Straker had a mistress and an assumed identity under which he had accrued a large debt, and to manage this debt, he planned to wound Silver Blaze and win a large bet placed on the horse’s chief rival. Before Straker has the opportunity to wound the horse, however, Blaze kicks him in the head and flees. The story subverts the reader’s/model’s expectations, but it also refers back to the detective story’s origin with Poe. The animal as murderer recalls Poe’s first Dupin detective story, “The Murders in the Rue Morgue,” in which the detective solves a seemingly inexplicable murder by discovering that it had been committed by an escaped orangutan.

The title of this post is a nod to Barbara Johnson’s essay [“The Frame of Reference: Poe, Lacan, Derrida“](chrome-extension://efaidnbmnnnibpcajpcglclefindmkaj/http://ereserve.library.utah.edu/Annual/ENGL/6670/Margolis/frameof.pdf) in which she analyzes Derrida’s critique of Lacan’s reading of Poe’s “Purloined Letter.” As that series of phrases suggests, reflexivity is both a feature of the detective story genre and the practice of reading detective stories. Johnson accuses Derrida of the same blindspots that he attributes to Lacan. I’m left wondering if genre is a blindspot for Google’s AI and what that would mean for how we understand AI. In her essay, Johnson pauses on the Dupin’s epigraph that he leaves for the minister in place of the purloined letter. It is a line from Crébillon’s play _Atrée et Thyeste_:
> “So bale a plan, if unworthy of Atreus, is worthy of Thyestes.”

Like “The Purloined Letter,” Crébillon's play is the “story of revenge as symmetrical repetition of the original crime” that is incited by a stolen letter. Johnson observes that Poe’s story is a “story of repetition” that is “itself a repetition of the story from which it purloins its last words.” All of this is to say that the founding moves of detective fiction are repetition and intertextuality. I have found that Google’s model tends to fall for red herrings and to select the most obvious (and thereby often wrong) suspects. As I move to the next phase of the experiment, I’m going to try to test this hypothesis across the corpus. If Google’s Gemini struggles to notice genre tropes, then it would seem to struggle with detective stories as the genre evolved and developed in the early 20th century. I'm now trying to think about what it would look like to test this hypothesis. (If you have any ideas, drop me a note!)

In the next post, I will analyze Gemini’s predictions across the entire Sherlock Holmes short story corpus before turning to the full detective story set. The code used for this experiment is available [here](https://github.com/collinjennings/detectiveLLMs/blob/main/solvingDetectiveStoriesAI.ipynb). 

