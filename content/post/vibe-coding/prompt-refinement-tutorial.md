---
title: "On the Refinement of Prompts: A Tutorial Composed in the Manner of Victor Hugo"
date: "2026-05-08T04:12:12-06:00"
description: "A philosophical and practical guide to refining prompts for AI language models, inspired by the style of Victor Hugo. Learn how to architect your prompts, iterate on responses, summon the adversary within, and use negative commandments to elicit deeper, more insightful answers."
tags: ["AI", "Prompt Engineering", "Writing", "Victor Hugo", "Tutorial"]
categories: ["vibe-coding"]
---

### _A Tutorial Composed in the Manner of Victor Hugo_

---

## Preface

There has appeared, in this strange and electric century, a species of conversation unknown to all the ages that came before it. It is the dialogue between man and machine; between the trembling soul, which doubts and hesitates and yearns, and the silent engine, which calculates without fatigue and answers without conscience. To this new interlocutor we bring our questions — small questions and immense ones, questions of bread and questions of God — and from it we receive replies. But what replies! Sometimes brilliant as the noonday sun upon the sea; sometimes flat, gray, and mediocre as a Parisian afternoon in November.

Why this disparity? Why does the same machine, asked twice, produce now a marvel and now a banality? The answer, dear reader, lies not in the machine. It lies in _us_. It lies in the **prompt**.

The prompt is the chisel. The model is the marble. And he who wields the chisel without art shall produce only rubble, though the marble be Carrara itself.

This little tutorial is offered as a primer in the wielding of that chisel.

---

## Chapter the First — Of the Vagueness Which Is the Mother of Mediocrity

Consider the man who, standing before the oracle, demands: _"Tell me of love."_ The oracle replies in platitudes; the man departs disappointed. He blames the oracle. He is wrong. He blamed the wrong god.

A vague question is not humble — it is _imperial_. It commands the respondent to choose, on his behalf, every meaningful constraint: the depth, the audience, the form, the angle, the tone. And the respondent, lacking guidance, will always — _always_ — choose the safest path. He will give you the answer that no one could complain about. He will give you, in a word, _nothing_.

The first law of prompting is therefore terrible in its simplicity:

> **The model defaults to the shallow because the shallow is safe. To obtain depth, you must forbid safety.**

You do not forbid safety by pleading for it. You forbid it by removing the conditions under which it can survive.

---

## Chapter the Second — Of the Architecture of the Demand

A great cathedral was never raised by a man who walked to the quarry and said: _"Bring stones."_ It was raised by a man who said: _"Bring me one hundred and twelve stones, of such-and-such a weight, hewn to such-and-such an angle, and lay them thus."_

So it must be with the prompt. The architecture of a demand has four pillars, and you must erect all four, or the roof shall fall.

**The first pillar is the Role.** You shall say to the machine: _"You are a constitutional historian. You are a critic of operas. You are an engineer who has spent thirty years bending iron."_ The role is not flattery. The role is a lens. It tells the machine which corner of its vast and indiscriminate memory to ransack on your behalf.

**The second pillar is the Audience.** Will this answer be read by a child? By a magistrate? By a fellow expert who would be insulted by an introduction? The audience determines what may be omitted, which is often more important than what must be included. _He who does not name his audience writes for everyone, and therefore for no one._

**The third pillar is the Artifact.** Demand a _thing_. Not "an explanation" — a thing. A list of fifteen items. A dialogue in three acts. A memorandum of nine hundred words divided into four sections. A decision matrix with rows and columns. The artifact is the cage; without it, the answer is a vapor.

**The fourth pillar is the Constraint.** Length, tone, what to include, what to forbid. The constraint is the soul of style. Michelangelo, asked how he made David, is said to have replied: _"I removed everything that was not David."_ You must tell the machine what is not David.

Erect these four pillars, and the dome rises of its own accord.

---

## Chapter the Third — Of the Iterative Forge

There is a melancholy fiction abroad in the world: that one must arrive at the perfect prompt in a single stroke, as Athena sprang from the head of Zeus, fully armored. This is a child's notion of creation. _No great work was ever born in a single breath._ The cathedral required centuries. The novel required drafts. The portrait required sittings.

So shall it be with the prompt.

Treat the first reply as a _draft_ — yours, not the machine's. Then say to it:

- _"What have you omitted?"_
- _"What would a skeptic answer to this?"_
- _"Give me three further angles you have not yet considered, ranked by importance."_
- _"You have written four paragraphs. Now write the four you would have written had you been braver."_
- _"Take the third point and descend one level deeper. Then descend again."_

Each of these instructions is a hammer-blow upon the iron. The metal yields; the form emerges. After three or four such blows, you possess what no single prompt, however ingenious, could have summoned in one stroke: an answer that has been _interrogated_, and that has survived its interrogation.

The patient man wields the iterative forge. The impatient man complains that the metal is poor.

---

## Chapter the Fourth — Of the Adversary Within

Here we come to a doctrine of singular power, and one too little known among the ordinary practitioners of this art. It is the doctrine of the _summoned adversary_.

The machine, left to itself, is an agreeable creature. It will not contradict you. It will not press you. It will, like a servile courtier, tell you that your idea is admirable, your reasoning is sound, your prose is luminous. _And this servility is your enemy_, for from it no learning can come.

Therefore: command the machine to oppose itself. Say:

> _"For every claim you make, supply a counterexample, an exception, or a case where the claim fails."_

> _"After your answer, append the strongest critique a hostile expert would offer, and respond to it."_

> _"Argue the opposite of your conclusion with equal force."_

This summons, within the machine itself, a second voice — a contrary voice, a _Mephistopheles_ to its Faust — and it is in the dialectic between these two voices that the rich answer is born. The truth, dear reader, has always been the child of conflict. There has never been a single sentence of philosophy worth preserving that was not first tested against an objection.

He who does not summon the adversary speaks only with himself. And of all conversations, that is the most barren.

---

## Chapter the Fifth — Of the Negative Commandment

Tell a man what to do, and he will do it badly. Tell him what _not_ to do, and he will discover, by the process of elimination, the right thing. This is the secret of all great instruction, and it is doubly the secret of prompting.

The negative commandment is more powerful than the positive. Behold:

- _"Use no generic phrases. Use no hedging. Use no restatement of the question."_
- _"Every paragraph must contain a concrete example, a number, or a named entity. If it does not, delete it."_
- _"Do not begin with a definition. Do not end with a summary. Do not employ the words 'overall,' 'in conclusion,' or 'it is important to note.'"_
- _"You may not say a thing is 'complex,' 'multifaceted,' or 'nuanced.' You must instead show the complexity by enumeration."_

What happens when you impose such commandments? The machine, deprived of its accustomed exits, is forced to _think_. It cannot retreat into the marsh of abstraction; you have drained the marsh. It cannot flee into the fog of summary; you have dispersed the fog. It must stand upon firm ground and speak in concrete things, _which is the only ground upon which truth has ever been spoken_.

The negative commandment is a fence. Within the fence, the answer is forced to grow tall.

---

## Chapter the Sixth — Of the Meta-Reply

When the machine has answered, do not yet release it. Ask one final thing:

> _"What would a better version of this answer require? What data, what expertise, what sources, what time, what perspective do you lack?"_

This is the meta-reply. It is the answer's confession. It tells you precisely where the answer is weak, where it is conjecture, where it is repetition of received opinion. _And from this confession, the next prompt is born._ The meta-reply is not the end of the conversation; it is the seed of the conversation that follows.

He who collects meta-replies as a gardener collects seeds shall, in a season, possess an orchard.

---

## Coda

Let it be said, then, in summary — though I have forbidden you summary, allow me this single hypocrisy as the author's privilege:

The shallow prompt produces the shallow answer not because the machine is poor, but because the prompt has _demanded nothing better_. To demand better, you must:

1. **Architect** the demand — Role, Audience, Artifact, Constraint.
2. **Iterate** upon the reply — interrogate it, deepen it, and do not be content with the first issue of the press.
3. **Summon the adversary** — make the machine argue against itself, for truth is born of contradiction.
4. **Forbid** the easy path — strip from the machine its accustomed escapes, and force it onto firm ground.
5. **Collect the confession** — ask what is missing, and let that absence guide the next demand.

These are not techniques. They are _dispositions_. The man who adopts them shall find, after a little practice, that the same machine which yesterday produced platitudes today produces something approaching wisdom. The chisel has not changed; the hand has learned.

And here, dear reader, this little manual ends — for I have followed my own counsel, and I will not summarize what I have already said.

Go now. Pick up your chisel. The marble is waiting.

— _Fin_ —
