---
layout: default
title: Project 2
---

**Due Tuesday October 6**

Posted: 9/25/2015
Last Update: 9/29/2015

## Programming Questions ##

Submit your answers to Problems 4 and 5 in the
[Rosalind final submission page](http://rosalind.info/classes/233/).

**NOTE: THESE PROGRAMS NEED TO BE SUBMITTED TO ROSALIND BY Monday 10/5 at 5:00PM**

**Question 1. (5 pts)** Write three or four sentences as a postmortem on the implementation of the two randomized algorithms. What was the biggest challenge in
  implementing and testing these algorithms?

**Note**: *Postmortem* refers to short reviews usually done after
completion of a project to set down lessons learned through the
project. Implementation methods and ideas that worked, and those that did
not. More info: [here](http://blog.codinghorror.com/the-project-postmortem/)
and [here](http://www.uio.no/studier/emner/matnat/ifi/INF9181/h11/undervisningsmateriale/reading-materials/Lecture-10/post-mortems.pdf).

### Code Grading (60 pts) ###

Same guidelines as [Project 1](projects/projects_1.html).

##Entropy ##

Consider length $$k$$ profiles estimated from a set of
$$t$$ $$k$$-mers.

**Question 2. (5 pts)** What is the *minimum* entropy score one of these profiles
can achieve? Provide an example set of $$t=5$$ $$4$$-mers that would
produce a profile with minimum entropy.

**Question 3. (5 pts)** What is the *maximum* entropy score one of these profiles
can achieve? Provide an example set of $$t=5$$ $$4$$-mers that would
produce a profile with maximum entropy.

## Entropy Game ##

I mentioned in class that the reason we would use (low) entropy and
self-similarity as a score in motif finding is that evolution would
produce low entropy profiles for DNA binding proteins that are
perform important biological processess (e.g., TFs that regulate
circadian genes). In this exercise you will empirically play with this
idea.

Suppose we have a brand new fictional universe with the first ever
living thing: a single cell, single gene organism that the only thing it does,
besides reproducing itself, is produce a DNA binding protein (HcbP) that induces expression of itself, and
that's how it keeps itself alive (it's a stretch I know). Suppose that
the following 12-mer is the sequence of DNA where the protein binds:
`TCGTACGGTATT`.

Now let's play the following evolution game:

1. Evolution for this organism works as follows:  
  (a) every year each individual in the population reproduces five times (see how reproduction works in step 2) to produce five offspring.  
  (b) each individual only survives for 10 years, and  
  (c) at the end of the year at most 100 of the individuals that made it to the end of the year randomly selected to continue living on to the next year.

2. Reproduction: we represent an individual strictly by the 12-mer in
the HcbP binding site. When an individual reproduces each position of the offspring's 12-mer is randomly mutated with probability 1/15 to a different
nucleotide.

3. Survival: remember that binding of HcbP is essential for survival
of this organism. Turns out that HcbP can bind to sequences that
satisfy the following: `T[C|G]GTNNNNT[A|G]NT`, this representation
means: in position 2 either `C` or `G` allows binding, in positions
with `N` any base allows binding. You can think of this as a `regular
expression`  and a match allows binding. However, no match means
the individual does not survive. (See below for more on regular expressions).

Ok, now code two versions of this game:
  (1) a version where step 3 (survival) is implemented, and
  (2) a version where step 3 is not implemented.

I.e., in version 1, if an individual's $$k$$-mer does not
match the above regular expression it dies, in version 2 it
survives. Remember that at most 100 individuals are retained at the
end of the year.

Run each version of your game 10 times for 100 years each time and
calculate the entropy of the profile corresponding to the surviving population. The outer loop will look something like this:

{% highlight python %}
nruns = 10
nyears = 100
num_offspring = 5
mutation_rate = 1. / 15
max_age = 10
max_population_size = 100

entropies = []
for i in xrange(nruns):
	# age and sequence of primodial organism
	population = [(0, 'TCGTACGGTATT')]
	for j in xrange(nyears):
		# five new individuals per
		# individual in the population, with random mutations at each
		# position with given probability
		population = reproduce(population, num_offspring, mutation_rate)

	    # remove members of the population with non-binding sequence
		# (only in version 1 of the game)
		population = remove_nonbinding(population)

	    # remove members of the population that are too old (10 years old)
		population = remove_elders(population, max_age)

        # increase the age of each individual and keep at most 100
        #  individuals, choose randomly if populations is larger that 100
		population = yearend(population, max_population_size)
	entropies.append(calculate_entropy(population))
{% endhighlight %}

**Question 4 (15 pts)**. Are the entropies generated by two versions of the
game different? Are the entropies more or less similar the longer you run the simulation (`nyears`). Make a plot or table that shows how entropies differ.

**Question 5. (10 pts)** How does the matching regular expression for the
binding site affect the
difference in resulting entropies? For instance, you can change the number of positions where matching
doesn't matter, or change the choice of nucleotides that allow binding.

(a) Suggest a change to the binding regular expression and a
hypothesis of how the entropies for the two versions of the game will
vary after you change the matching expression.

(b) Now, run your experiment again using your new matching rule and calculate
entropies again. Do these results agree with your hypothesis?

## Notes ##

The goal of this exercise is for you to understand the concept of
entropy and how it may be a reasonable measure to use when thinking
about the evolution of DNA sequences. So, here are some hints on how
to implement this:

- To check if a mutation should be inserted in a given position in an
  offspring string you can use `if random.random() <= mutation.rate`  

- The `filter` function is very useful to modify lists according to
  some predicate. For example, to remove individuals from the
  population based on age (using the tuple representation above) you
  can use `population=filter(lambda x: x[0] <= max_age, population)`

- To check for binding it's easiest to use the regular expression
  library in python. The binding expression above can be used as
  follows:

{% highlight python %}
import re

# the rule for pattern T[C|G]GTNNNNT[A|G]NT
# match C or G in the second position
# match A,C,G or T in positions 5-8 ([ACGT]) is the set of characters
# that match {4} is the exact number of matches
# match A or G in position 10
# match A,C,G or T in position 11
matchRE = re.compile("T[CG]GT[ACGT]{4}T[AG][ACGT]T")

# this code checks if string 'kmer' binds according to the rule
matchRE.match(kmer) is not None
{% endhighlight %}

- To randomly select 100 items from a list you can use

{% highlight python %}
# this shuffles in place
random.shuffle(population)
population = population[:100]
{% endhighlight %}

- Remember that we define $$0 \cdot log2(0) = 0$$. You can use code
  like this to enforce this: `numpy.log2(p) if p > 0 else 0`

- Finally, you need to think about how to summarize the entropies you
  calculate from the 10 runs you perform. The `numpy` library can be
  helpful here, e.g., `numpy.mean`.

- Remember that the easiest way to install libraries in python is to
  use `pip`:

{% highlight bash %}
$ pip install numpy
{% endhighlight %}


## How to submit ##

Submit your answers to the five questions above in writing as a pdf to ELMS. Submit along with that your entropy game code.
