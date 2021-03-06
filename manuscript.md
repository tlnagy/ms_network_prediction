---
  bibliography: [references.bib]
---
# Introduction

Ecosystems *are* interactions -- organisms interact with one-another and with
their environment, either directly or indirectly. Between organisms, these
interactions form networks of varying complexity, drive ecological and
evolutionary dynamics, and maintain ecosystem diversity and functioning
[@Delmas2018AnaEco; @Landi2018ComSta; @Albrecht2018PlaAni]. Networks of species
interactions underpin our understanding of key ecological processes
[@Pascual2006EcoNet; @Heleno2014EcoNet]. Yet, even basic knowledge of species
interactions (like being able to list them, or guess which ones may exist) is
one of the most severe shortfalls in biodiversity science [@Hortal2015SevSho].
This is due in large part to the tedious, time-consuming, and expensive data
collection process. As with many ecological systems, networks of species
interactions have entered their "long now" [@Carpenter2002EcoFut], where
contemporary actions have long-term, low-predictability consequences
[@Burkle2013PlaInt].

Therefore, our field needs a conceptual path forward toward models that enable
prediction (for the present) and forecasting (for the future) of species
interactions and the networks they form [@McCann2007ProBio; @Seibold2018NecMul].
Here we provide a data-driven illustration to show how machine learning
approaches can enable unreasonably effective prediction of interactions whereby
we construct a metaweb of host-parasite interactions across space, which serves
as a proof-of-concept for this conceptual framework. We then propose a roadmap
forward for how to improve predictions using this approach, and provide a
primer on the relevant tools and methods that could be incorporated into models
of this type in the future in order to account for the spatial, temporal, and
climatic dimensions of network prediction [@Burkle2011FutPla].



# Proof-of-Concept: can we predict ecological networks?

The core premise of this manuscript is that ecological networks can be
predicted. In this section, we provide a proof-of-concept, in which we (i)
aggregate a series of networks collected across space into a metaweb, (ii)
extract features based on species co-occurrence, (iii) use these
features to train a neural network to predict interactions, and (iv) apply this
classifier to the original features to predict possibly missing interactions
across the entire species pool. The entire analysis is presented in
@fig:example, and the code to reproduce it is available at **TODO OSF LINK**;
the entire example was carried out in *Julia 1.5.3* [@Bezanson2017JulFre], and
notably uses the *Flux* machine learning framework [@Innes2018FluEle]. Note that
this analysis is meant to serve as an *example only*, and should in practice be
fined-tuned according to the state of the art [*e.g.* @Goodfellow2016DeeLea].

We used data from @Hadfield2014TalTwo, describing 51 host-parasite networks,
where not all species pairs co-occur across sites. This implies that there are
"negative associations" that might be biologically feasible but not observed
because the two species have not been observed in co-occurrence. As this dataset
has no features like species traits on which to base a predictive model, we have
aggregated all interactions into a binary metaweb [@Dunne2006NetStr]. We then
transformed the (undirected) metaweb through a probabilistic PCA
[@Tipping1999ProPri], so as to create a number of latent features for the
species in a context where the dataset is both unbalanced and likely to have
many missing values. This frames the problem as predicting a binary outcome, the
interaction $M_{xy}$ represented as `true` or `false`  based on a features
vector $v_{xy} = [v_x, v_y]$ where $v_x$ is the values of the selected features
for the parasite and $v_y$ is the features of the host. In the following
example, we used the first 15 components of the latent sub-space created by the
probabilistic PCA. This features vector is then fed into the input layer of a
neural network, which uses three hidden layers with appropriate dropout rates
($0.5$), and finally a two-neurons output layer whose result is softmaxed to
pick the most likely outcome,  *i.e.* the interaction bit describing an
interaction when equal to 1, and no interaction when equal to 0.

![ (A) A conceptual overview of the process of network prediction. Beginning with data of observed interaction between species, we aim to predict the metaweb of interaction across the entire species pool, even those that have not been observed together. (B) Proof-of-Concept: An empirical network [from
@Hadfield2014TalTwo] is converted intro latent features using probabilistic PCA,
then used to train a deep neural network to predict species interactions. The
initial and imputed networks are represented as their tSNE embedding, and the
colors of nodes are the cluster to which they are assigned based on a $k$-means
clustering of the tSNE
output.](figures/example_network_prediction.png){#fig:example}

During the training of this neural network, we exploited ecological constraints
in two ways. First, the selection of features was done so that absent
interactions in a species pair with no co-occurrence were removed from the data.
This ensures that the network is trained only on the subset of the data for
which we have minimal ecological information. Second, the batches of 16 items
used for training were constrained to have at least 10 positive interactions.
The reasoning for this choice was made based on three observations: the network
is sparse (*i.e.* the prevalence of interactions is low); negative interactions
have a chance of being false negatives due to lack of reporting in the field;
there are no true negative interactions reported, *i.e.* interactions for which
we know that they almost never happen. Therefore, slightly inflating the dataset
with positive interactions enables us to counterbalance these biases.

After the training ($2.5\times 10^4$ epochs in @fig:example), our model reached
an accuracy of $\approx 0.8$, with no marked deviation between the training and
testing sets (respectively 80% and 20% of the data), suggesting no to minimal
overfitting. Applying this model to the entire dataset (including species pairs
never observed co-occuring in the dataset) identified 1831 new possible
interactions -- 382 of which were in species pairs where the pair of species was
never considered prior. This suggests that meaningful information about
ecological interaction is structured within network data, and our core argument
here is that we should embrace the prediction of species interaction networks as
a worthy topic of concept , and specifically strive to adopt an explicitly
spatial and temporal perspective on this question. Now, the question becomes:
home do we make our prediction of ecological networks _better_?

# A Roadmap Toward Better Prediction of Ecological Networks across Space and Time

Below we focus on and discuss integrating what we envisage to be the conceptual
and methodological pathway towards better prediction of
ecological networks (@fig:conceptual).


## Challenges: the many constraints on prediction

### Ecological network data are scarce and hard to obtain

At the moment, our understanding of the structure of ecological networks is
limited by the availability of data. Although we have seen a growth in species
occurrence data, this growth is much slower for ecological interactions because
species interactions are challenging to sample comprehensively
[@Bennett2019PotPit; @Jordano2016SamNet] and sampling methodology has strong
effects on the resulting data [@deAguiar2019RevBia]. In turn, the difficulty of
sampling interactions can lead to biases in our understanding of network
structure [@deAguiar2019RevBia]. This knowledge gap has motivated a variety of
approaches to deal with interactions in ecological research based on assumptions
that do not always hold, such as the assumption that co-occurrence is equivalent
to meaningful interaction strength, when it is known that co-occurrence is not the
only prerequisite for an interaction to occur [@Blanchet2020CooNot]. Spatial
biases in data coverage are prevalent at the global scale (with South America,
Africa and Asia being under-represented) and different interaction types show
biases towards different biomes (or environmental conditions)
[@Poisot2020EnvBia]. These "spatial gaps" serve as a limitation to our ability
to confidently make predictions when accounting for real-world environmental
conditions, especially in environments for which there are no analogous data.

Further, the analysis of interaction strength from empirical estimation is
highly prone to bias as existing data quantifying interaction strength are
usually lumped together, making it difficult to differentiate the strength in
per-individual interactions from the strength of a whole species interaction
[@Wells2013SpeInt]. Empirical estimations of interaction strength are still
crucial [@Novak2008EstNon], but are a hard task to quantify in natural
communities [@Wootton1997EstTes; @Sala2002ComDis; @Wootton2005MeaInt],
especially as the number of species composing communities increases,
compounded by the possibility of higher-order interactions or non-linear responses in
interactions [@Wootton2005MeaInt]. Furthermore, interaction strength is
extremely variable and context dependent and can be influenced by density
dependence and spatiotemporal variation in abundances and community composition
[@Wootton2005MeaInt]. A better understanding of interaction strengths in
communities is a key step in linking species interactions to ecosystem processes
and functioning.

### Powerful predictive tools work better on large data volumes

This scarcity of data limits the range of computational tools than can be used
by network ecologists. Most deep learning methods, for instance, are very data
expensive. The paucity of data is compounded by a collection of biases that can
be found in existing datasets. Species interaction datasets are typically
dominated by food webs, pollination, and host-parasite networks
[@Ings2009EcoNet; @Poisot2020EnvBia]. This could prove to be a limiting factor
when trying to understand or predict networks of *underrepresented* interaction
types or when trying to integrate networks of different types
[@Fontaine2011EcoEvo], especially given the inherent structural variation of
ecological networks [@Michalska-Smith2019TelEco]. This stresses the need for an
integrated, flexible, and data-efficient set of computational tools which will
allow us to predict ecological networks accurately from existing and imperfect
datasets, but also enable us to perform model validation and comparison with
more flexibility than existing tools. We argue that @fig:example is an example
of the promise of these tools *even* when facing datasets of small size. When
carefully controlling for overfitting machine learning systems are at least
adequate at generalizing. The ability to extract and engineer features also
serves to bolster our predictive power. In short, the current lack of massive
datasets must not be an obstacle to prediction; it is an ideal testing ground to
understand how little data is sufficient to obtain actionable predictions.

### Scaling-up predictions requires scaled-up data

We are also currently limited by the the level of biological organisation at
which we can describe ecological networks. For instance, our understanding of
individual based networks [*e.g.* @Araujo2008NetAna; @Tinker2012StrMec] is still
in its infancy [@Guimaraes2020StrEco] and acts as a resolution-limit. On the
note of scale, the resolution of environmental (or landscape) data would also
limit our ability to predict networks at finer scales, although current trends
in e.g. remote sensing would suggest that with time this would become less of a
hindrance [@Makiola2020KeyQue]. Ecosystems are a quintessential
complex-adaptive-system [@Levin1998EcoBio] with a myriad of ways in which
processes at different spatial, temporal, and organizational scales can
influence and respond to one another. Understanding how the product of these
different processes drive the properties of ecosystem across different scales
remains a central challenge of ecological research, and we should strive to work
on methods that will integrate different empirical "snapshots" of this larger
system.

## Opportunities: the emerging ecosystem of open tools and data

If we wish to predict the interactions between species we have not observed
together, using our knowledge of the structure of ecological networks to
interact in a particular ecosystem is one of our most useful assets. We are able
to infer species interactions using proxies such as traits, phylogenies,
geographical data and other frameworks [@Morales-Castilla2015InfBio]. Drawing on
elements that contribute to the realization of an interaction such as abundance
and traits matching in space and time, and the combination of these elements
allow us to infer potential from realized interactions and empirical data about
populations [@Poisot2016StrPro]. In turn, this effort is supported by a thriving
ecosystem of data sources and computational tools. In this section, we give a
brief overview of these resources.

### Open Data

The acquisition of biodiversity and environmental data has tremendously
increased over the past decades thanks to the rise of citizen science
[@Dickinson2010CitSci] and of novel technology [@Stephenson2020TecAdv],
including wireless sensors [@Porter2005WirSen], next-generation DNA sequencing
[@Creer2016EcoSF], and remote sensing [@Skidmore2015AgrBio; @Lausch2016LinEar].
Open access databases, such as [GBIF](https://www.gbif.org/) (for biodiversity
data), [NCBI](https://www.ncbi.nlm.nih.gov/) (for taxonomic and genomics data),
[TreeBASE](https://www.treebase.org/treebase-web/home.html) (for phylogenetics
data), [CESTE](https://icestes.github.io/) [@Jeliazkov2020GloDat] (for
metacommunity ecology and species traits data), and
[WorldClim](https://www.worldclim.org/data/bioclim.html) (for bioclimatic data)
contain millions of data points that can be integrated to monitor and model
biodiversity at the global scale. For species interactions data, at the moment
[Mangal](https://mangal.io/#/) is the most comprehensive open database of
published ecological networks [@Poisot2016ManMak], and
[GloBI](https://www.globalbioticinteractions.org/about) is an extensive database
of realized and potential species interactions [@Poelen2014GloBio]. Developing
standard practices in data integration and quality control [@Kissling2018BuiEss]
and in next-generation biomonitoring [NGB; @Makiola2020KeyQue] would improve our
ability to make reliable predictions of ecosystem properties on increasing
spatial and temporal scales. The advancement of prediction techniques coupled
with a movement towards standardising data collection protocols (e.g.
@Perez-Harguindeguy2013NewHan for plant functional traits) and metadata (e.g.
[DarwinCore](https://www.tdwg.org))---which facilitates interoperability and
integration of datasets---as well as a growing interest at the government level
[@Scholes2012BuiGlo] paints a positive picture for data access and usability in
the coming years.

### Open Tools and Methods

Machine learning encompasses a broad variety of techniques applied with or
without human supervision. These techniques can often be more flexible and
perform better than classical statistical methods, and can achieve a very high
level of accuracy in many predictive and classification tasks in a relatively
short amount of time [e.g. @Cutler2007RanFor; @Krizhevsky2017ImaCla]. Increasing
computing power combined with recent advances in machine learning techniques and
applications shows promise in ecology and environmental science (see
@Christin2019AppDee for an overview). Moreover, ongoing developments in the
field of artificial intelligence are aimed at using deep learning more
efficiently in low-data regimes [e.g. @Antoniou2018DatAug] and with unbalanced
datasets [@Chawla2010DatMin]. Machine learning is emerging as the new standard
in computational ecology in general [@Olden2008MacLea; @Christin2019AppDee], and
in network ecology in-particular [@Bohan2017NexGlo], as long as sufficient
relevant data are available. Many ecological and evolutionary processes underlie
species interactions and the structure of their ecological networks [e.g.
@Vazquez2009UniPat; @Segar2020RolEvo]. It can thus be difficult to choose
relevant variables and model species interactions networks explicitly. A
promising application of machine learning in natural sciences is
Scientific-Machine Learning (SciML), a framework that combines machine learning
with mechanistic models [@Chuang2018AdvCon; @Rackauckas2020UniDif]. Considering
the current biases in network ecology [@Poisot2020EnvBia] and the scarcity of
data of species interactions, the prediction of ecological networks will
undoubtedly benefit from these improvements. Many studies have used machine
learning models specifically with ecological interactions. Relevant examples
include species traits used to predict interactions and infer trait-matching
rules [@Desjardins-Proulx2017EcoInt; @Pichler2020MacLea], automated discovery of
food webs [@Bohan2011AutDis], reconstruction of ecological networks using
next-generation sequencing data [@Bohan2017NexGlo], and network inference from
presence-absence data [@Sander2017EcoNet].

![A conceptual roadmap highlighting key areas for the prediction of ecological
networks. Starting with the input of data from multiple sources, followed by a
modelling framework for ecological networks and the landscape, which are then
ultimately combined to allow for the prediction of spatially explicit
networks.](figures/conceptual_v2.png){#fig:conceptual}

# A Primer on Predictive Network Ecology

Below we provide a primer on the current state of predictive network ecology,
with particular focus on using machine learning approaches in the modelling
process. Here adopt a question-driven approach to serve as a guide through the
path toward building models to predict and forecast the structure of ecological
networks across space, and to identify the next steps in the research regime.


## Models

### What is a predictive model?

Models are used for many purposes, and the term "model" embodies a wide variety
of meanings in scientific discourse. All models can be thought of as a function,
$f$, that takes a set of inputs $x$ (also called features, descriptors, or
independent variables) and some parameters $\theta$, and maps them to predicted
output states $y$ (also called label, response, or dependent variable) based on
the input to the model: $y=f(x,\theta)$. However, any given model $f$ can be
used for either descriptive or predictive purposes. Many forms of scientific
inquiry are based around using models _descriptively_ (also called inference,
the inverse problem, fitting a model, or training a model)
[@Stouffer2019AllEco]. In this context, the goal of using a model is to estimate
the parameters, $\theta$, that best explain a set of empirical observations,
$\{\hat{x}, \hat{y}\}$. In some cases, these parameter values are themselves of
interest (e.g the strength of selection, intrinsic growth rate, dispersal
distance), but in others cases, the goal is to compare a set of competing models $f_1,
f_2, \dots$ to determine which provides the most parsimonious explanation for a dataset. The quantitative
representation of "effects" in these models---the influence of each input on the
output---is often assumed to be linear, and in the frequentist context, the goal
is often to determine if the coeffecient corresponding with an input is non-zero
to determine its "significance" in influencing the outcome. Models designed for
inference have utility, however, in order for ecology to develop as a predictive
science [@Evans2012PreEco], interest has grown in developing models that are
used not just for description of data, but also for prediction. Predictive
models use _the forward problem_, where the aim is to predict new values of the
output $y$ given an input $x$ and our estimate value of $\theta$
[@Stouffer2019AllEco]. Because the forward problem relies on an estimate of
$\theta$, then, the problem of inference is nested within the forward problem
(@fig:models).

![The nested nature of developing predictive and forecasting models, showcases
the _forward problem_ and how this relies on a hierarchical structure of the
modelling process.](figures/forecasting_v3.png){#fig:models}

### What do you need to build a predictive model?

In order to build a predictive machine-learning model, one needs the following:
first, **data**, split into features $\hat{x}$ and labels $\hat{y}$ (Box Figure
Label). Second, a **model** $f$, which maps features $x$ to labels $y$ as a
function of parameters $\theta$, i.e. $y = f(x, \theta)$. Third, a loss function
$L(\hat{y}, y)$, which describes how far a model's prediction $y$ is from an
empirical estimate $\hat{y}$. Lastly, **priors** on parameters, $P(\theta)$.
Another important step before fitting a model is feature engineering: adjusting
and reworking the predictors to better uncover
predictor-response relationships [@Kuhn2019FeaEng]. This can include projecting
the predictors into a lower dimensional space, as in our proof-of-concept.

### How do we validate a predictive model?

After model fitting, we inevitably want to see how "good" it is. One of the
context for validation is _model comparison_, where we aim to see which of a
competing set of models provides the best explanation for a data set. A naive
initial approach is to simply compute the average error between the model's
prediction and the true data we have, and choose the model with the smallest
error---however this approach inevitably results in _overfitting_. One approach
to avoid overfitting is using information criteria (*e.g.* AIC, BIC, MDL) based
around the heuristic that good models maximize the ratio of information provided
by the model to the number of parameters it has. However, when the intended
use-case of a model is prediction the relevant form of validation is
_predictive accuracy_, which should be tested with _crossvalidation_. Crossvalidation methods divide the
original dataset into two---one which is used to fit the model (called the _training_ set) and one
 used to validate its predictive accuracy on the data that is hasn't "seen" yet
 (called the _test_ set) [@Bishop2006PatRec]. This procedure is often repeated
 for different subdivisions of the dataset [@Arlot2010SurCro].

## Networks and Interactions

### Why predict networks and interactions at the same time?

Ecological networks are quite sparse [@MacDonald2020RevLin]---composed of a set
of interactions, but also a larger set of non-interactions. If we aim to predict
the structure of networks from the "bottom-up"--- by considering each pairwise
combination of $S$ different species---we are left with $S^2$ interaction values
to estimate. Instead, we can use our existing understanding of the mechanisms
that structure ecological networks to whittle down the set of feasible adjacency
matrices, thereby reducing the amount of information we must predict, and making
the problem of predicting interactions less daunting. The processes that
structure ecological networks do not only occur at the scale of
interactions---there are also processes at the network level which limit what
interactions are possible. The realized structure of a network is the synthesis
of the interactions forming the basis for network structure, and the network
structure refining the possible interactions---"Part makes whole, and whole
makes part" [@Levins1987DiaBio].

### What network properties should we should use to inform our predictions of interactions?

There are many dimensions of network structure [@Delmas2018AnaEco]. This might
make the task of network structure prediction look daunting, as the number of
properties one could predict is immense. Yet there are two reasons to begin with
a single property, connectance (the ratio of actual edges to possible edges in
the network). First, connectance is ecologically informative---it relates to
resilience to invasion [@Baiser2010ConDet; @Smith-Ramesh2016GloSyn], can
increase robustness to extinction in food webs [@Dunne2002NetStr], while
decreasing it in mutualistic networks [@Vieira2015SimSto], and connectance
relates to network stability [@Landi2018ComSta]. Second, most (if not all)
network properties co-vary with connectance [@Poisot2014WheEco;
@Dunne2002FooStr]. We have models to estimate species richness over space
[@Jenkins2013GloPat], and because we can predict connectance from species
richness, [@MacDonald2020RevLin], we can then derive distributions of network
properties from estimates of richness alone. Therefore we suggest that
predicting the value of network connectance across space (and eventually time)
is most likely to be the most practical to formulate at the moment.

### How do we predict how species that have never co-occurred will interact?

The probability of an interaction occurring depends on the likelihood of
co-occurrence in space and time [@Poisot2015SpeWhy; @Pichler2020MacLea]. Given
two species co-occur, a neutral approach to probabilistic interactions would
assume that the effect of abundances and trait matching would have no effect
[@Canard2012EmeStr]. However, functional-trait based proxies could enable better
predictions of ecological interactions. Selection on functional
traits could cause interactions to be conserved at some evolutionary scales, and
therefore predictions of interaction could be informed by phylogenetic analyses.
[@Elmasri2020HieBay; @Gomez2010EcoInt]. Phylogenetic matching in bipartite networks is consistent
across scales [@Poisot2018IntRet], even absent strong selective pressure
[@Coelho2017NeuBio].

A separate family of methods are based on network embedding (as in the
proof-of-concept). A network embedding projects each node of the network into a
lower-dimensional latent space. This enables us to represent the structure of a
network, which previously required the $S^2$ dimensions of an adjacency matrix,
with a smaller number of dimensions. The position of each node in this lower
dimensional space is then treated as a latent measurement corresponding to the
role of that species in the network [@Becker2020PreWil]. Species close together
in the latent space should interact with similar set of species
[@Rossberg2006FooWeb; @Rohr2010ModFoo]. However, these models are sensitive to
sampling biases as they are limited to species for which there is already
interaction data [@Becker2020PreWil].

### What is an interaction, really?

Interactions between species can be conceptualized in a multitude of ways
(mutualistic vs. antagonistic, strong vs. weak, symmetric vs. asymmetric, direct
vs. indirect) [@Jordano2016ChaEco; @Morales-Castilla2015InfBio]. What is common
to all definitions of interaction is that *at least* one of the species
is affected by the presence of another, either positively or negatively
[@Morales-Castilla2015InfBio]. Networks can
be used to represent a variety of interaction types, including: *unipartite
networks,* where each species can be linked any other species, species (these
are typically used to represent food webs), *bipartite networks* where there are
two pools of species, and all interactions occur between species in each pool,
are typically used for pairwise interactions (e.g. hosts and parasites), and
*k-partite networks:,* which serve as a way to expand to more than two discrete
sets of interacting species (e.g. parasitoid webs, seed dispersal networks, and
pollination networks) [@Pocock2012RobRes].

### What about interaction _strength_?

Species interaction networks can also be used as means to quantify and
understand _interaction strength_. Interaction strength, unlike the qualitative
presence or absence an interaction, is a continuous measurement which attempts
to quantify the effect of one species on another. Interaction strength can
generally be divided into two main categories (as suggested by
@Berlow2004IntStr): either the strength of an interaction between individuals of
each species, or the effect that changes in one species population has on the
dynamics of the other species. Further, it can be measured either as the direct
effect of one species on another over a period of time (often in the units of
biomass) or the relative importance of one species on another
[@Heleno2014EcoNet; @Berlow2004IntStr; @Wootton2005MeaInt]. One recurring
observation throughout studies of interaction strengths is that networks are
often composed of many weak links and few strong links [@Berlow2004IntStr]. The
distribution of interaction strength within a network informs on its stability
[@Neutel2002StaRea; @Ruiter1995EnePat], influences on the ecosystem functioning
[@Duffy2002BioEco; @Montoya2003FooWeb] and our potential to improve multispecies
models [@Wootton2005MeaInt]. Seeing interaction strength within a network as
energy fluxes could also possibly lead to its integration within the
Biodiversity-Ecosystem Functioning (BEF) framework, which could in return
further improve even our understanding of community dynamics and ecosystem
functioning [@Barnes2018EneFlu].

### How are interaction strengths actually estimated?

Before we attempt to make inferences from data, we must adapt a conceptual
framework to model interaction strength. One such framework is functional
foraging [@Portalier2019MecPre], where the primary basis for inferring
interaction is based on an organism's traits, the environment, and foraging
behavior like searching, capture and handling times. A different conceptual
alternative, applicable in food-webs, is metabolic based models, where body
mass, metabolic demands, and energy loss are used to infer energetic energy
fluxes between organisms [@Yodzis1992BodSiz;@Berlow2009SimPre]. Energy can be
seen as the common currency that links every level of biology from individual
organisms to the whole ecosystem [@Brown2004MetThe;@Barnes2018EneFlua]. A
(metabolically) bottom-up approach first estimates basal species biomasses and
compute the higher trophic-level species biomasses and energetic demands from
there [@Berlow2009SimPre]. Alternatively, a metabolically top-down approach
computes energy fluxes starting from the top consumer downward toward producers
[@Barnes2018EneFlu]. Food-web energetics models can be incorporated at various
resolutions for a specific network, ranging from individual-based data to more
lumped data at the species level or trophic group, depending on data
availability [@Barnes2018EneFlu].


### What about indirect and higher-order interactions?

Although network ecology often assumes that interactions go strictly from one
node to the other, the web of life is made up of a variety of interactions.
Indirect interactions---either higher-order interactions between species, or
interaction strengths that themselves interact --- has gained interest in recent
years [@Golubski2016EcoNet; @Golubski2011ModMod]. One mathematical tool to
describe these situations is hypergraphs: hypergraphs are the generalization of
a graph, allowing a broad yet manageable approach to complex interactions
[@Carletti2020DynSys], allowing for particular interactions to occur beyond a
pair of nodes. An additional degree of complexity is introduced by multi-layer
networks [@Hutchinson2019SeeFor]. Multi-layer networks include edges across
"variants" of the networks (timepoints, locations, or environments). These can
be particularly useful to account for the metacommunity structure
[@Gross2020ModMod], or to understand how dispersal can inform conservation
action [@Albert2017AppNet].  Ecological networks are intrinsically multi-layered
[@Pilosof2017MulNat]. *Prima facie*, increasing the dimensionality of the object
we need to predict (the multiple layers rather than a single network) may make
the problem complicated. But multi-layer networks encode ecological constraints
-- of dispersal, of evolution, and of niche suitability. It is worth
investigating if the multi-layer structure of ecological networks could improve
the predictibility of interactions, as in social networks [@Jalili2017LinPre;
@Najari2019LinPre; @Yasami2018NovMul].


### How do we determine what interaction networks are feasible?

For several decades, ecologists have aimed to understand how networks of many
interacting species persist through time. The diversity-stability paradox,
first explored by @May1974StaCom, shows that under a neutral set of
assumptions, ecological networks should become decreasingly stable as the number
of species increases. However, in the natural world we observe networks of
interactions that consist of far more species than May's model predicts
[@Albouy2019MarFis]. As a result, understanding what aspects of the neutral
assumptions of May's model are incorrect has branched many investigations into
the relationship between ecological network structure and persistence
[@Allesina2012StaCri]. These assumptions can be split into dynamical assumptions
and topological assumptions.

Topologically, we know that ecological networks are not structured randomly.
Some properties, like the aforementioned connectance, are highly predictable
[@MacDonald2020RevLin]. Various generative models of food-webs have been shown
to fit empirical networks more effectively than random models. These typically
rely on network embeddings, where each node (species) in the network is assigned
a value in a latent space, and the resulting network topology is generated
stochastically based on properties of the position of nodes in that latent
space. Generative network models have long used allometry as a
single-dimensional latent space---naturally we want to extend this to traits in
general [@Allesina2008GenMod]. The second approach to understand stability is
through _dynamics_. Early models of community dynamics rely on the assumption of
linear interaction effects. However, models of bioenergetic community dynamics
have shown promise in basing our understanding of dynamics in food-webs, where
the functional response of one species on another is grounded in the understood
relationship between allometry and metabolism [@Delmas2017SimBio]. An additional
consideration is the multidimensional nature of "stability" and "feasibility"
(e.g resilience to environmental change vs extinctions)
[@Dominguez-Garcia2019UnvDim] and how different disturbances propagate across
levels of biological organization [@Kefi2019AdvOur; @Gravel2016StaCom].

## Space

Although networks were initially used to describe the interactions *within* a
community, interest in the last decade has shifted towards understanding their
structure and variation over space [@Trojelsgaard2016EcoNet; @Baiser2019EcoRul],
and has established network ecology as an important emerging component of
biogeography and macroecology.

### How much do networks vary over space?

Networks can vary across space either in their structural properties (e.g.
connectance or degree distribution) or in their composition (identity of nodes
and edges). Interestingly, variation in the structural properties of ecological
networks primarily responds to changes in the size of the network. The number of
links in ecological networks scales with the number of species
[@MacDonald2020RevLin; @Brose2004UniSpa], and connectance and size drive the
rest of the structural properties of ecological networks [@Poisot2014WheEco;
@Dunne2002FooStr; @Riede2010ScaFoo]. Species turnover in space results in
changes in the composition of ecological networks. But, this is not the only
reason network composition varies [@Poisot2015SpeWhy]. Intraspecific variation
can result in interaction turnovers without changes in species composition
[@Bolnick2011WhyInt]. Similarly, changes in species abundances can lead to
variation in interaction strengths [@Canard2014EmpEva; @Vazquez2007SpeAbu].
Variation in the abiotic environment and indirect interactions
[@Golubski2016EcoNet] could modify the occurrence and strength of
individual interactions. Despite this, empirical networks tend to share a common
backbone [@Mora2018IdeCom] and functional composition [@Dehling2020SimCom]
across space.

### How do we predict what the species pool at a particular location is?

As the species pool forms the basis for network structure, predicting which
species are present at a particular location is essential to predict networks
across space. Species distribution models (SDMs) are increasingly ubiquitous in
macroecology--- these models predict the range of a species based on known
occurrences and environmental conditions, such as climate and land cover
[@Guisan2005PreSpe; @Elith2006NovMet]. Including interactions or co-occurrences
in SDMs generally improves predictive performance [@Wisz2013RolBio].Several
approaches exist to combine multiple SDMs: community assemblage at a particular
site can be predicted either by combining independent single-species SDMs
(stacked-SDMs, SSDMs) or by directly modelling the entire species assemblage and
multiple species at the same time (joint SDMs, JSDMs) [@Norberg2019ComEva].
Building on the JSDM framework, hierarchical modeling of species communities
[@Ovaskainen2017HowMak] has the advantage of capturing processes that structure
communities. Spatially Explicit Species Assemblage Modeling (SESAM) constrains
SDM predictions using macro-ecological models [@Guisan2011SesNew] --- for
example, variation in species richness across space can constrain assemblage
predictions [@DAmen2015UsiSpe].

The next step is to constrain distribution predictions using network properties.
This would also build on previous calls to adopt a probabilistic view: there
have been calls for a probabilistic species pool [@Karger2016DelPro], and more
importantly including interactions through Bayesian networks and propagating
conditional dependencies has generated better range predictions
[@Staniczenko2017LinMac]. @Blanchet2020CooNot argue that the probabilistic view
avoids confusion between interactions and co-occurrences, but that it requires
prior knowledge of interactions. This could potentially be solved through our
framework of predicting networks first, interactions next, and finally the
realized species pool.

### How do we combine spatial and network predictions?

In order to predict networks across space, we need to combine multiple
models---one which predicts what the species pool will be at a given location,
and one to predict what interaction networks composed from this species pool are
likely (see @fig:conceptual). Both of these models contain uncertainty, and when
we combine them our uncertainty from each to propagate into the combined model.
The Bayesian paradigm provides a convenient solution to this---if we have a
chain of models where each model feeds into the next, we can sample from the
posterior of the input models. A different approach is _ensemble modeling_ which
combines the predictions made be several models, where each model is predicting
the same thing [@Parker2013EnsMod]. Error propagation, an important step in
building any ecological model, describes the effect of the uncertainty of input
variables on the uncertainty of output variables [@Draper1995AssPro;
@Parysow2000EffApp]. @Benke2018ErrPro identifies two broad approaches to model
error propagation: analytically using differential equations or stochastically
using Monte-Carlo simulation methods. Errors induced by the spatial or temporal
extrapolation of data also need to be taken into account when estimating the
uncertainty of a model's output [@Peters2004StrEco].

### What is the spatial scale suitable for the prediction of species interactions?

As described above, we can use different trait-based proxies to predict
potential interactions. The choice of such proxies should be theoretically
linked to the spatial scale we are using in our prediction [@Wiens1989SpaSca].
At some scales we can use morphological traits of co-occurring species to assess
the probability of interaction between them [@Bartomeus2016ComFra]. This
translates to a spatial extent that does not necessarily capture the entire
distribution of a given set of species, with a resolution that is sufficient to
capture the phenotypical variability of the species. At some scales, we can
infer interactions through the phylogenetic similarity between species, assuming
their functional traits are themselves are conserved
[@Gomez2010EcoInt]. On evolutionary scales where the niche is conserved, we
can think of the probability that one species will interact with another as the
distance between them in niche-space [@Desjardins-Proulx2017EcoInt]. At
the smallest scales, we may be interested in predicting behavioral traits like
foraging behavior [@Bartomeus2016ComFra]. At this point, the spatial resolution
in this case should is fine enough that a model may be precise in a given
system, but much less generalizable. At this scale it is also important to
consider abundance's effect on probability of an encounter [@Wells2013SpeInt].


## Time

### Why should we forecast species interaction networks?

Forecasting species interactions are critical for informing ecosystem management
[@Harvey2017BriEco] and systematic conservation prioritization
[@Pollock2020ProBio], and for anticipating extinctions and their consequences
[@McDonald-Madden2016UsiFoo; @McWilliams2019StaMul]. Ecological interactions
shape species distributions at both local and broad spatial scales, and including
interactions in SDM models typically improves predictive performance
[@Araujo2007ImpBio; @Wisz2013RolBio; @Pigot2013SpeInt]. However, these tend to rely on
approaches involving estimating pairwise dependencies based on cooccurance,
using surrogates for biotic-interaction gradients, and hybridizing SDMs with
dynamic models [@Wisz2013RolBio]. Most existing models to predict the future
distribution of species ignore interactions [@Urban2016ImpFor]. Changes in
species ranges and phenology will inevitably create spatiotemporal mismatches
and affect encounter rates between species [@Gilman2010FraCom], which will
further shift the distribution of species across space. New interactions will
also appear between species that are not currently co-occuring
[@Gilman2010FraCom]. Only by forecasting how species will interact can we hope
to have an accurate portrait of how biodiversity will be distributed under the
future climate.

Forecasting how climate change will alter biodiversity is also crucial for
maximizing conservation outcomes. Improving SDMs through interactions is crucial
for conservation, as nearly 30% of models in SDM studies are used to assess
population declines or landscape ability to support populations
[@Araujo2019StaDis]. Reliable predictions about how ecological networks will
change over time will give us critical information that could be communicated to
decision-makers and the scientific community about what are future environmental
risks awaiting and how to mitigate them [@Kindsvater2018OveDat]. Not only this,
but how biodiversity is structured influence the functioning of the whole
ecosystem, community stability and persistence [@Thompson2012FooWeb,
@Stouffer2010UndFoo]. Will climate change impact the distribution of network
properties (e.g. connectance)? If so, which regions or species groups need
special conservation efforts? These overarching questions are yet to be answered
[but see @Albouy2013ProCli; @Kortsch2015CliCha; @Hattab2016ForFin]. We believe
that the path toward forecasting ecological networks provides useful guidelines
to ultimately better predict how climate change will affect the different
dimensions of biodiversity and ecosystem functioning.

### How do we turn a predictive model into a forecasting model?

On some scales, empirical time-series encode enough information about ecological
processes for machine-learning approaches to make accurate forecasts. However,
there is an intrinsic limit to the predictability of ecological time-series
[@Pennekamp2019IntPre]. A forecast inherently has a _resolution limit_ in space,
time, and organization. For example, one could never hope to predict the precise
abundance of every species on Earth on every day hundreds of years into the
future. There is often a trade-off between the resolution and horizon of
forecast, *e.g.*, a lower resolution forecast, like primary production will be
at a maximum in the summer, is likely to be true much further into the future
than a higher resolution forecast. If we want to forecast the structure of ecological networks beyond
the forecasting horizon of time-series based methods, we need forecasts of our
predictive model's inputs--- a forecast of the distribution of both
environmental conditions and the potential species pool across space
(@fig:models).

### How can we validate a forecasting model?

Often the purpose of building a forecasting model to inform _present_ action
[@Dietze2018IteNea]. Yet, the nature of forecasting---trying to predict the
future---is that you can only know if a forecast is "right" once it is too late
to change it. If we want to maximize the chance that reality falls within a
forecasting model's predictions, there are two directions to approach this
problem: the first is to extend model validation techniques to a forecasting
context, and the second is to attempt to maximize the amount of uncertainty in
the forecast without compromising its resolution. Crossvalidation (see _How do
we validate a predictive model?_) can be used to test the efficacy of a
forecasting model. Given a time-series of $N$ observations, a model can
iteratively be trained on the first $n$ time-points of data, and the forecasting
model's accuracy can be evaluated on the remaining time-points it hasn't "seen"
[@Bishop2006PatRec]. This enables us to understand both how much temporal data
is required for a model to be robust, and also enables us to explore the
_forecasting horizon_ of a process. Further, this approach can also be applied
in the opposite temporal direction--- if we have reliable data from the past,
"hindcasting" can also be used to test a forecast's robustness.

However, these methods inevitably bump into a hard-limitation on what is
feasible for a forecasting model. The future is uncertain. Any empirical
time-series we use to validate a model was collected in past conditions that may
not persist into the future. Any system we wish to forecast will undergo only
one of many possible scenarios, yet we can only observe the realized outcome of
the system under the scenario that actually unfolds. It is therefore impossible
to assess the quality of a forecasting model in scenarios that remain
hypothetical. If the goal is to maximize the probability that reality will fall
within the forecast's estimates, forecasts should incorporate as much
uncertainty about the future scenario as possible---one way to do this is
ensemble modeling [@Parker2013EnsMod]. However, as we increase the amount of
uncertainty we incorporate into a forecasting model, the resolution of the
forecast's predictions could shrink [@Lei2017EvaTra], and therefore the modeler be mindful of the
trade-off between resolution and accuracy where developing any forecast.

# Conclusion: why should we predict species interaction networks?

Because we almost can, and because we definitely should. A better understanding
of species interactions, and the networks they form, would help unify the fields
of community, network, and spatial ecology; improve the quantification of the
functional relationships between species [@Dehling2018BriElt;
@OConnor2020UnvFoo]; re-evaluate metacommunities in light of network structure
[@Guzman2019MulExt]; and enable a new line of research into the biogeography of
species interactions [@Massol2017ChaFou; @Braga2019SpaAna] which incorporates a
synthesis of both Eltonian and Grinnellian niche [@Gravel2019BriElt]. Further,
the ability to reliably predict and forecast species interactions would inform
conservation efforts for protecting species, communities, and ecosystems.
Integration of species interactions into the assessment of vulnerability to
climate change is a needed methodological advance [@Foden2016IucSsc].
International panels draw on models to establish scientific consensus
[@Araujo2019StaDis], and they can be improved through more effective prediction
of species distributions and interactions [@Syfert2014UsiSpe]. Further, recent
studies argue for a shift in focus from species to interaction networks for
biodiversity conservation to better ecosystem processes [@Harvey2017BriEco].


We should invest in network prediction because the right
conditions to do so reliably and rapidly, including forecasting, are beginning
to emerge. Given the possible benefits to a variety of ecological disciplines
that would result from an increased ability to predict ecological networks and
their structure, we feel strongly that the research agenda we outline here
should be picked up by the community. Although novel technologies are bringing
massive amounts of data to some parts of ecology (primarily environmental DNA
and remote sensing, but now more commonly image analysis and bioacoustics), it
is even more important to be intentional about *reconciling* data. This involves
not only the scholarly work of understand the processes that the data encode,
and whether they speak to scales that are compatible, but also the groundwork of
developing pipelines to bridge the ever-expanding gap between "high-throughput"
and "low-throughput" sampling methods. An overall increase in the volume of data
will not result in an increase of our predictive capacity as long as this data
increase is limited to specific aspects of the problem. In the areas we
highlight in @fig:conceptual, many data steps are still limiting: documenting
empirical interactions is natural history work that doesn't lend itself to
systematic automation; expert knowledge is by design a social process that may be
slightly accelerated by text mining and natural language processing (but is not
yet, or not routinely or at scale). These limitations are affecting our ability
to reconstruct networks.

But the tools to which we feed these data, incomplete as they may be, are
gradually getting better; that is, they can do predictions faster, they handle
uncertainty and propagate it well, and they can accomodate data volumes than are
lower than we may expect [@Pichler2020MacLea]. It is clear attempting to predict the structure
of ecological networks at any scale is a methodological and ecological
challenge; yet it will result in qualitative changes in our understanding of
complex adaptive systems, as well as changes to our ability to leverage
information about network structure for conservation decision. It is perhaps
even more important to forecast the structure of ecological networks because it
is commonly neglected as a facet of biodiversity that can (and should) be
managed. In fact, none of the Aichi targets mention biostructure or its
protection, despite this being recognized as an important task
[@McCann2007ProBio], either implicitely or explicitely. Being able to generate
reliable datasets on networks in space or time will make this information more
actionable.

**Acknowledgements:** TS, NF, TP are funded by a donation from the Courtois
Foundation; FB, NF, and TP are funded by IVADO; BM is funded by the NSERC
Alexander Graham Bell Canada Graduate Scholarship and the FRQNT master's
scholarship; FB, GD, NF, and GH are funded by the NSERC BIOS² CREATE program; 
DC, TS, LP, and TP are funded by the Canadian Institute of Ecology & Evolution; 
this research was enabled in part by support provided by Calcul Québec
(www.calculquebec.ca) and Compute Canada (www.computecanada.ca).

# References
