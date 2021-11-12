# NLP Corporate Acquisition Information Extraction System

## ENVIRONMENT
```
pip3 install allennlp==2.1.0 allennlp-models==2.1.0
#Need this to use SRL model https://storage.googleapis.com/allennlp-public-models/structured-prediction-srl-bert.2020.12.15.tar.gz
```
spacy version 3.0.7
python3 -m spacy download en_core_web_sm # Small spacy model
pandas 1.3.1

## NOTES

TODO: Write README

- https://orbifold.net/default/what-is-semantic-role-labeling/
	- This code didn't work!
- https://github.com/masrb/Semantic-Role-Labeling-allenNLP-
	- Used this script to figure out how to use allenNLP SRL model
	- Outputs using PropBank Annotation


An example of semantic role labeling being used in allennlp. Note the allennlp version!!
https://demo.allennlp.org/semantic-role-labeling


### Answer key template

TEXT (ID)
ACQUIRED* (F1)
ACQBUS* (F2)
ACQLOC* (F3)
DLRAMT* (F4)
PURCHASER* (F5)
SELLER* (F6)
STATUS* (F7)

### HYPOTHESES:
- In general:
  - PURCHASER, SELLER, and ACQUIRED will most often be named entities


- For "Buy" verbs
  - Allen SRL:
     - PROTO-AGENT(ARG-0) => PURCHASER
     
 ## TO-DO

TODO: EXPERIMENT:: On the entire data set: Figure out correlations between each semantic role tag group \*-T 
- For each training document:
  - Tag each sentence for semantic role
  - Then group all data into a dictionary D1 mapping a tag group T to a set of tuples I:=(V, S) -- where T is the name of the tag (e.g. ARG0, ARGM-LOC, O [for O], V), V is the verb in the current frame, and S is the associated string (B's and I's of type T concatenated together). Ignore O's
  - Construct another dictionary D2 that is the same as D1, except T maps directly to a set of strings S (no V)
  - Find out what percent of the time a gold string GS in each field GF is approximately equal to some string S in T\[\]
  - Generate a table of the probability of GS being in T[] for all T and all fields F
  
```python
    ARG0  ARGM-LOC   V ...
GF1  P1    P2
GF2  P3    P4
GF3 ..........
GF4
GF5
GF6
GF7

# Therefore P1 would be: 
( (freq(GF1.value in set:ARG0) / (freq(GF1.value)) ) for each gold in golds
#NOTE: we only want to do this where GF1.value is not ---
# We are basically trying to determine the probability that the correct gold answer is in our
# set of candidates for each T
# so that we can determine which T's are most highly correlated with which field.
# At the end of this experiment, we should know things like "There is a 70% chance that the set ARG0 contains F5 (the PURCHASER)".
```

The next step would be to perform the same experiment, but only for frames where the verb V is a buy/sell verb. And to generate the tables depending on if the verb is a buy/sell verb. For example, I hypothesize that when the verb is a buy verb, the AGENT is probably the PURCHASER. Whereas with a sell verb, the AGENT is probably the SELLER.

The initial model (due Nov 10) will just pick some value from the most correlated set. The initial model will probably assign STATUS to the verb. STATUS will probably be the hardest thing to extract.

A subsequent model will pick the best value from the set based on multiple factors. 
- For example, (assuming that the AGENT is probably the purchaser), the PURCHASER field will have a bunch of AGENT (ARG0) candidates to choose from. Of those candidates, it will prefer to be filled out by a named entity. If it can't find a named entity, then it just won't be filled out at all.

- For fields without great correlation, I might have to implement custom logic for extracting it. For example, I think that the DLRAMT might be best extracted by just identifying a string with money related keywords next to it (like "dollars", "$", etc.).

## High level description of solution

### Brainstorm 

1. Semantic Role tagging for the document.
2. Aggregate semantic tags into a two dictionaries for the entire document--A "buy-verb" dictionary, and a "sell-verb" dictionary that takes semantic tags and maps them to a set of strings. Keep 
3. Based on heuristics derived in experiments, assign each field to their most correlated semantic role dictionaies. The dictionaries contain candidates. Apply additional heuristic to select the best candidates. If we cannot get further evidence, simply fill out the field as blank.
 
## NOTES:

- Originally I only looked for AQCLOC in AGM-LOC, but this turned out to be terrible. No fields were filled out. 
- Now switching to not using the SRL


Experiment: just set DLRAMT to be everything with entity tag 'MONEY'
                RECALL             PRECISION          F-SCORE
ACQUIRED        0.00 (0/418)	   0.00 (0/0)         0.00
ACQBUS          0.00 (0/153)	   0.00 (0/0)         0.00
ACQLOC          0.40 (53/134)	   0.11 (53/478)      0.17
DLRAMT          0.04 (7/164)	   0.11 (7/62)        0.06
PURCHASER       0.00 (0/373)	   0.00 (0/0)         0.00
SELLER          0.00 (0/156)	   0.00 (0/0)         0.00
STATUS          0.00 (0/295)	   0.00 (0/0)         0.00
--------        --------------     --------------     ----
TOTAL           0.04 (60/1693)	   0.11 (60/540)      0.05


Experiment: just set DLRAMT to be everything with entity tag 'QUANTITY'
                RECALL             PRECISION          F-SCORE
ACQUIRED        0.00 (0/418)	   0.00 (0/0)         0.00
ACQBUS          0.00 (0/153)	   0.00 (0/0)         0.00
ACQLOC          0.40 (53/134)	   0.11 (53/478)      0.17
DLRAMT          0.03 (5/164)	   0.02 (5/201)       0.03
PURCHASER       0.00 (0/373)	   0.00 (0/0)         0.00
SELLER          0.00 (0/156)	   0.00 (0/0)         0.00
STATUS          0.00 (0/295)	   0.00 (0/0)         0.00
--------        --------------     --------------     ----
TOTAL           0.03 (58/1693)	   0.09 (58/679)      0.05

Experiment: just set DLRAMT to be everything with entity tag 'QUANTITY' OR 'MONEY'
                RECALL             PRECISION          F-SCORE
ACQUIRED        0.00 (0/418)	   0.00 (0/0)         0.00
ACQBUS          0.00 (0/153)	   0.00 (0/0)         0.00
ACQLOC          0.40 (53/134)	   0.11 (53/478)      0.17
DLRAMT          0.07 (12/164)	   0.05 (12/263)      0.06
PURCHASER       0.00 (0/373)	   0.00 (0/0)         0.00
SELLER          0.00 (0/156)	   0.00 (0/0)         0.00
STATUS          0.00 (0/295)	   0.00 (0/0)         0.00
--------        --------------     --------------     ----
TOTAL           0.04 (65/1693)	   0.09 (65/741)      0.05

Experiement: 
- entity tag 'GPE' => ACQLOC
- entity tag 'QUANTITY' OR 'MONEY' => DLRAMT
- entity tag 'ORG' => ACQUIRED, PURCHASER, SELLER
                RECALL             PRECISION          F-SCORE
ACQUIRED        0.59 (245/418)	   0.14 (245/1748)    0.23
ACQBUS          0.00 (0/153)	   0.00 (0/0)         0.00
ACQLOC          0.40 (53/134)	   0.11 (53/478)      0.17
DLRAMT          0.07 (12/164)	   0.05 (12/263)      0.06
PURCHASER       0.63 (234/373)	   0.13 (234/1748)    0.22
SELLER          0.69 (108/156)	   0.06 (108/1748)    0.11
STATUS          0.00 (0/295)	   0.00 (0/0)         0.00
--------        --------------     --------------     ----
TOTAL           0.39 (652/1693)	   0.11 (652/5985)    0.17

Experiement: 
- entity tag 'GPE' => ACQLOC
- entity tag 'QUANTITY' OR 'MONEY' => DLRAMT
- entity tag 'ORG' => ACQUIRED, PURCHASER
                RECALL             PRECISION          F-SCORE
ACQUIRED        0.59 (245/418)	   0.14 (245/1748)    0.23
ACQBUS          0.00 (0/153)	   0.00 (0/0)         0.00
ACQLOC          0.40 (53/134)	   0.11 (53/478)      0.17
DLRAMT          0.07 (12/164)	   0.05 (12/263)      0.06
PURCHASER       0.63 (234/373)	   0.13 (234/1748)    0.22
SELLER          0.00 (0/156)	   0.00 (0/0)         0.00
STATUS          0.00 (0/295)	   0.00 (0/0)         0.00
--------        --------------     --------------     ----
TOTAL           0.32 (544/1693)	   0.13 (544/4237)    0.18
