---
{"dg-publish":true,"permalink":"/differential-privacy/","created":"2024-06-24T23:26:44.927+08:00","updated":"2024-06-25T12:05:11.304+08:00"}
---

#Differential_Privacy 
### What is Differential Privacy?  
Differential privacy protects user data from being traced back to individual users. The parameters involved are known as the [[Privacy budget\|privacy budget]]. This is a metric of privacy loss based on adding or removing one entry in a data set

Differential Privacy is a _privacy definition_ that was originally developed by Dwork, Nissim, McSherry and Smith, with major contributions by many others over the years. Roughly speaking, what it states can summed up intuitively as follows:

> Imagine you have two otherwise identical databases, one with your information in it, and one without it. Differential Privacy ensures that the probability that a statistical query will produce a given result is (nearly) the same whether it’s conducted on the first or second database.  

- Strange to think first, why can't the adversary utilise the approximate data to do something bad?

But first, let see what Apple do to protect user privacy.
> There are situations where Apple can improve the user experience by getting insight from what many of our users are doing, **for example**: *What* new words are trending and might make the most relevant suggestions? *What* websites have problems that could affect battery life? *Which* emoji are chosen most often? The challenge is that the data which could drive the answers to those questions—such as what the users type on their keyboards—**is personal**

Differential privacy transforms the information shared with Apple before it ever leaves the user’s device such that Apple can never reproduce the true data.

The differential privacy technology used by Apple is rooted in the idea that statistical noise that is slightly biased can **mask a user’s individual data before it is shared with Apple**. If many people are submitting the same data, the noise that has been added can **average out over large numbers of data points**, and Apple can see meaningful information emerge.
- Wait!!! That means the noise in uncontrollable situation? What if someone make MIMT attack of the low noise individuals? 
	- Local differential privacy guarantees that it is difficult to determine whether a certain user contributed to the computation of an aggregate by adding slightly biased noise to the data.
	- Which says every number is same possibility?
	- Maybe $f(x \mid y) = \frac{1}{2 \ln(2) (2n - |y - x|)}$?

Differential privacy is used as the first step of a system for data analysis that includes robust privacy protections at every stage. The system is opt-in and designed to provide transparency to the user. 
- The first step we take is to **privatize the information using local differential privacy** on the user’s device. The purpose of privatization is to assure that Apple’s servers don't receive clear data. 
- Device identifiers are removed from the data, and it is transmitted to Apple **over an encrypted channel**. The Apple analysis system **ingests** the differentially private contributions, dropping IP addresses and other metadata. 
- The final stage is **aggregation**, where the privatized records are processed to compute the relevant statistics and the aggregate statistics are then shared with relevant Apple teams. Both the ingestion and aggregation stages are performed in a restricted access environment so even the privatized data isn’t broadly accessible to Apple employees.
### A tradeoff between privacy and accuracy  
Obviously calculating the total number of 💩 loving users on a system is a pretty silly example. 
The neat thing about DP is that the same overall approach can be applied to much more interesting functions, including complex statistical calculations like the ones used by Machine Learning algorithms. It can even be applied when many different functions are all computed over the same database.

**But there’s a big caveat here.** Namely, while the amount of **“information leakage”** from a single query can be bounded by a small value, this value is not zero. Each time you query the database on some function, the total “leakage” increases — and can never go down. Over time, as you make more queries, this leakage can start to add up.

This is one of the more challenging aspects of DP. It manifests in two basic ways:
- **The more information you intend to “ask” of your database, the more noise has to be injected in order to minimize the privacy leakage.** This means that in DP there is generally a fundamental tradeoff between accuracy and privacy, which can be a big problem when training complex ML models.
- **Once data has been leaked, it’s gone.** Once you’ve leaked as much data as your calculations tell you is safe, you can’t keep going — at least not without risking your users’ privacy. At this point, the best solution may be to just to destroy the database and start over. If such a thing is possible.

The total allowed leakage is often referred to as a “[[Privacy budget\|privacy budget]]”, and it determines how many queries will be allowed (and how accurate the results will be). The basic lesson of DP is that _the devil is in the budget._ Set it too high, and you leak your sensitive data. Set it too low, and the answers you get might not be particularly useful.

### Ways to implement a Differentially Private system
- Executed by a trusted database operator who has access to all of the “raw” underlying data.
- On the theoretical side, statistics can be computed using fancy cryptographic techniques (such as [secure multi-party computation](https://en.wikipedia.org/wiki/Secure_multi-party_computation) or [fully-homomorphic encryption](https://blog.cryptographyengineering.com/2012/01/very-casual-introduction-to-fully.html).) Unfortunately these techniques are probably too inefficient to operate at the kind of scale Apple needs.
- A much more promising approach is _not to collect the raw data at all_
	- RAPPOR
		- Apple’s Craig Federighi [describes Differential Privacy as](https://www.wired.com/2016/06/apples-differential-privacy-collecting-data/) “_using hashing, subsampling and noise injection to enable…crowdsourced learning while keeping the data of individual users completely private.”_
### Reference
- [Differential Privacy in Apple](https://images.apple.com/privacy/docs/Differential_Privacy_Overview.pdf)
- [What is Differential Privacy?](https://blog.cryptographyengineering.com/2016/06/15/what-is-differential-privacy/)