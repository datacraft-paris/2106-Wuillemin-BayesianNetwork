# 2106-Wuillemin-BayesianNetwork


# PyAgrum cheatsheet

Useful links :

[aGrUM/pyAgrum's website](https://agrum.gitlab.io/)

[pyAgrum's documentation](https://pyagrum.readthedocs.io/en/0.20.3/)

[aGrUM's gitlab](https://gitlab.com/agrumery/aGrUM)

## Import

### Import pyAgrum

```
import pyagrum as gum
```

### Import notebook specific library

```
import pyAgrum.lib.notebook as gnb
```

  
## In/Out

### Load a BN

```
bn = gum.loadBN({path})
```

### Save a BN

```
gum.saveBN(bn,"file.bif")
```
  

## [Basic Manipulation](https://pyagrum.readthedocs.io/en/latest/BNModel.html)

### BN generation

```
bn = gum.BayesNet("Your BN name")
```

### [Fast BN generation](https://pyagrum.readthedocs.io/en/latest/BNModel.html?highlight=fastPrototype#pyAgrum.BayesNet.fastPrototype)

Arcs, number of modalities and default number of modalities can be directly specified 

```
bn = gum.fastBN("a[2]->b<-c",3)
```

Labels can be directly specified

```
bn = gum.fastBN('A->B[1,3]<-C{yes|No}->D[2,4]<-E[1,2.5,3.9]',6)
```

### [Random variable generation](https://pyagrum.readthedocs.io/en/latest/randomVariables.html)

LabelizedVariable is a discrete random variable with a customizable sequence of labels.
```
va = gum.LabelizedVariable('a','a labelized variable',3)
```

RangeVariable represents a variable with a range of integers as domain.
```
vi = gum.gum.RangeVariable('I','I in [4,10]',4,10)
```

DiscretizedVariable is a discrete random variable with a set of `ticks` defining intervalls.
```
vX=gum.DiscretizedVariable('X','X has been discretized')
vX.addTick(1).addTick(2).addTick(3).addTick(3.1415)
```

### CPT generation
Randomly generate CPT for a given structure or for a given node in a given structure.

```
bn.generateCPTs()
```

```
bn.generateCPT({var_id|var_name})
```

### Adding/removing random variables

```
bn = gum.BayesNet('my BayesNet')
va = gum.LabelizedVariable('a','a labelized variable',3)
vb = gum.LabelizedVariable('b','another labelized variable',3)
vc = gum.LabelizedVariable('c','a third labelized variable',3)
bn.add(va)
bn.add(vb)
bn.add(vc)

bn.erase('c')
```

### Arc adding

```
bn.addArc('a','b')
```

### Arc reversing 

```
bn.reverseArc('a','b')
```

### Arc removing

```
bn.eraseArc('b','a')
```

### CPT definition
Once the network topology is constructed, we must initialize the conditional probability tables (CPT) distributions. Each CPT is considered as a Potential object in pyAgrum. There are several ways to fill such an object.

Consider the following BN
```
bn=gum.fastBN("c->r->w<-s<-c")
```

*low-level approach*
```
bn.cpt('s').fillWith([0.5,0.5,0.9,0.1])
```

*Using the order of variable*
```
bn.cpt("s")[:]=[[0.5,0.5],[0.9,0.1]]
```
Then $P(S|C=0)=[0.5,0.5]$
and $P(S|C=1)=[0.9,0.1]$.

*using a dict*

```
bn.cpt("s")[{'c':0}]=[0.5,0.5]
bn.cpt("s")[{'c':1}]=[0.9,0.1]
 ```

## [Visualization](https://pyagrum.readthedocs.io/en/latest/lib.notebook.html)

Consider the following BN :
```
bn = gum.fastBN("a->b<-c",3)
cpt = bn.cpt("a")
```

Show BN

```
gnb.showBN(bn)
```

Show CPT

```
gnb.showPotential(cpt)
```

Show posterior with or without evidence

```
gnb.showPosterior(bn,target="b",evs={})
gnb.showPosterior(bn,target="b",evs={'a':1,'c':2})
```

Show multiple informations at the same time

```
gnb.sideBySide(bn,cpt)
```

### [Inference](https://pyagrum.readthedocs.io/en/latest/BNInference.html)

Consider the following BN :

```
bn = gum.fastBN("a->b<-c",3)
```

[Exact Inference](https://pyagrum.readthedocs.io/en/latest/BNInference.html#exact-inference)

```
ie = gum.LazyPropagation(bn)
ie.addEvidence('b',1)
ie.posterior('b')
```

[Approximate Inference](https://pyagrum.readthedocs.io/en/latest/BNInference.html#approximated-inference)

```
ie = gum.LoopyBeliefPropagation(bn)
ie.setEpsilon(1e-7)
ie.setMaxIter(1000)
ie.setMaxTime(10)
ie.addEvidence('b',1)
ie.posterior('b')
```

### Generator

Consider the following BN :
```
bn = gum.fastBN("a->b<-c",3)
```

CSV generation

```
gum.generateCSV(bn,"database.csv",50000)
```

### [Learning](https://pyagrum.readthedocs.io/en/latest/BNLearning.html)

Consider the following BN :

```
bn = gum.fastBN("a->b<-c",3)
```

BN learner initialization :

```
learner=gum.BNLearner("database.csv",bn)
```

Parameter learning

```
bn2=learner.learnParameters()
```

Structural learning

```
learner.useLocalSearchWithTabuList() ### Or any other algorithm
bn2=learner.learnBN()
```

Add prior knowledge about the structure

```
learner.setMaxIndegree(1)
learner.addMandatoryArc('a','b')
```

Change score

```
learner.useScoreLog2Likelihood() ### Or other scores
```
