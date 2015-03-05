#On Explainability in Machine Learning


A few days ago, Gartner's [Anton Chuvakin](http://blogs.gartner.com/anton-chuvakin/) posted an article to his blog called [Killed by AI Much? A Rise of Non-deterministic Security!](http://blogs.gartner.com/anton-chuvakin/2015/03/03/killed-by-ai-much-a-rise-of-non-deterministic-security/).  In this post, he (rightly) points out that Machine Learning has gotten to the point where we can produce judgements that cannot be easily explained.  His big question is *My dear security industry peers, are we OK with that?*

I am. Here's why.

###Is ML Deterministic?
One of Dr. Chuvakin's key points is that ML is nondeterministic (that is, if you repeat it with the same input data, you may not get the same output). In my experience, nondeterminism is rare.  True, there is a role for randomness in training models (they don't call it a **Random** Forest for nothing!) but once the model is trained and moved into production, it's usually fixed so you can reliably reproduce results (at least until the model is retrained).  In other words, that model is now deterministic. 

Having a deterministic model is necessary prerequisite to being able to explain the judgements made by that model.  After all, if there's any randomness in the judging, it's clearly not something you can explain.  

###ML is Probably Not Explainable
So determinism is a prerequisite for explainable systems, but it's not sufficient by itself.  In fact, I believe (as does Dr. Chuvakin) that most non-trivial ML systems probably **cannot** easily explain their judgements.  

Imagine a simple binary classifier, perhaps based on one of those Random Forests I mentioned earlier.  It accepts some inputs (say, HTTP server logs) and tries to judge whether there is any evidence of a SQL injection attack.

>As a sidenote, Dr. Chuvakin's example classifier is just oriented towards finding "BAD" things. I think this is far too broad a task. A good model should be specific in what it's looking for.  Narrow scopes help build trust in the models, and also make them conceptually easier to understand.

Now there are many features one could extract from a typical HTTP server log.  Here's a partial list of ones that I can think of without much effort:

* HTTP method
* User-Agent
* Request status code (sebas comment:should this be Response status code?)
* Target Domain or IP
* Document Location
* Document type 
* Request parameter names & values
* Number of bytes sent
* Number of bytes received
(sebas comment: Not sure if it is good to put so many technical and specific details in this more 'general' discussion)

It seems like given those, you should be able to look at an individual entry and decide whether it has SQL in the request somewhere (probably in the parameter values).  As a human, you certainly can (sebas comment: you most probably can?).  Keep in mind, though, that most of these "features" are actually not very useful to a computer.  Most of these are strings (e.g., the HTTP method might be a "GET" or a "POST") but your classifier probably needs to operate on numbers.  Thus, we might establish a mapping for the HTTP method, where "GET" is represented by a 1, and "POST" is represented by 2 (and other methods are other numbers, of course).  (sebas comment: To base the discussion in a specific example, such as the mapping, may be dangerous. Here some may argue that the mapping is optional depending on the goal and technique)

Finally, we have something that we can do math with!  However, we also just broke the linkage between the original data and the feature that our classifier is computing against. Just as we established a mapping to convert the human-readable data into something the machine can do math with, we would need to maintain the reverse mapping if we want to audit what happened in a meaningful way.  

For a more complicated case, consider what we need to do with any request parameters (that is, anything after the *?* in the URI, such as */test.php?foo=bar*) (sebas comments: Be careful, previously you defined parameters _and_ values, and now you say that everything after the ? are parameters).  It's not correct to just map each unique string to a number as we did with the HTTP method, and yet we can't directly operate on the string itself.  What we actually have to do is to find a set of numbers that *describe* the string without actually reproducing it. You could consider easy descriptions, such as the length of the string (7, in this example), or more complicated ones such as the ratio of SQL special characters to the length of the string (1/7 in this case, because of the *=* sign).  

Here again, though, we run into a mapping problem.  Given a description of just a bunch of numbers, it's actually not possible to "go back" and recreate the original string.  Right there, we're running into trouble with explainability. Even if you know that the ratio was 1/7, you have no idea which character appeared in the string, nor where it appeared. We can't tell what the original input was since we're operating at a different level of abstraction now.    

Given all this, it's not really a surprise that most ML systems cannot explain how they arrived at their conclusions.  It's almost like trying to reverse a one-way hash (as with password cracking).  We don't know how to work backwards from the finished result in order to recover the original data, and it's likely impossible to do so.  And even if you did the work to fully document how each input was processed through all the decision trees in our forest, they are just numbers and don't mean much in terms that humans can understand anyway.

###So What?
Given the very nature of ML and the problems we use it to solve, though, I question whether "explainability" is even a reasonable expectation.  

The fact is, we use ML when we operate at scales that would explode human heads ("Big Data" can be dangerous!).  Compared to the type of analysis we're used to doing in the security space (network forensics, NSM alert validation, etc.), ML solves fundamentally different problems in a fundamentally different (due to scale) space.  

With any reasonably complex system, the model is dealing with so many features and so many data points that the idea of translating such a complicated set of information into something a human can understand in detail is ludicrous.  It's akin to trying to draw a [hypercube](http://en.wikipedia.org/wiki/Hypercube) on paper.  You can't even draw the third dimension, let alone the fourth (or higher) dimensions.  

That's not to say that we can't come close, though.  It's up the vendors to provide documentation about the general logic of their models.  For example, it's entirely within the realm of possibility for an analytics product to say something like *Based on an historical baseline of this user's communication patterns with the network, and those with similar job titles in the same location, it is likely that this new communication pattern represents a threat actor using thee user's credentials to move through the network.*  You can validate this assertion by examining those baselines and the questionable activity (perhaps visualized as a time series graph) without knowing every data point in the baselines or graph.

###Conclusion
So I agree with Dr. Chuvakin that explainable ML systems are hard, and in most practical cases, impossible.  However, this just doesn't bother me because I never expected detailed explanations in the first place.  Even if I were able to get them, I *certainly* wouldn't have time to evaluate all of them.  

What I *do* expect, though, is that the system provide me some level of acceptable evidence, even at a highly digested level.  This could be the aforementioned time series graphs to prove that the "shape" of the series was anomalous (even though I don't necessary know *how* that shape was computed).  In some cases, it could even be as simple as just showing me the original log entries that were the inputs to the ML algorithm in the first place and letting me make up my own mind (essentially evaluating how much I trust the machine's judgement, rather than doing a full analysis from scratch). I wouldn't want to have to manually validate everything, since that would probably destroy most of the value I wanted to get out of the system in the first place, but it's critical that I **always** have the ability to do so.

We use ML and other analytics techniques to help us make sense of datasets that are too big for us to process, to complex for us to understand, or both.  It's not effective (and is probably a waste of time) to expect that we have to understand every step in the process.  Rather, we need to understand our goals, our data and our algorithms in enough detail that we can develop a comfort level with the fact that we're just not equipped to deal with the problems we developed ML to solve in the first place. 

And that's OK.


## Seba's general comments 

- About explainability: It may be possible to compare ML explainability to human behavior. Do you understand why other people make decisions? Sometimes yes, sometimes no. And do you trust them? Well, again, sometimes. People need to gain your trust if you want to blindly accept what they do. It may be possible to do the same with algorithms. 
- Chuvakin is ok with recommendation systems. Although you also can't explain the results there, I think that he likes them because the cost of the errors in those algorithms is negligible. So nobody is being bothered too much.
- From the point of view of the creators of the algorithms, it is very desirable to be able to understand why your algorithm produced those results. Unfortunately, most of the algorithms can not explain them and therefore the researcher don't know *how* to improve the algorithms.




