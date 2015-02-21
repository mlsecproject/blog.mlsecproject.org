Title: The Importance of Good Labels in Security Datasets
Date: 2015-01-20 05:40
Category: Reseach
Status: draft

Working as security researchers is common to create a new machine learning algorithm that we want to evaluate. It may be that we are trying to detect malware, identify attacks or analyze IDS logs, but at some point we figure it out that we need a **good dataset** to complete our task. But not any dataset; in fact we need **a labeled dataset**. The dataset will be used not only to learn the features of, for example, malware traffic, but also to verify how good our algorithm is. Since getting a dataset is difficult and time consuming, the most common solution is to get a third-party dataset; although some researchers with time and resources may create their own. Either way, most usually we obtain a dataset of malware traffic (continuing with the malware traffic detection example) and we assign the label **Malware** to all of its instances. This looks good, so we make our training and testing, we obtain results and we publish. However, there are important **problems in this approach** that can **jeopardize the results** of our algorithm and the verification process. Let's analyze each problem in turn.

## The need for Normal Labels
> Without Normal labels we can not compute most error metrics nor compare our algorithm.

The first problem is the use of an insufficient number of types of labels. Assigning only _malware_ labels is good to look at candidate features, maybe to run some tests and to compute **True Positives** and **False Negatives**. However, without **normal** labels we miss a serious amount of information and we lose the possibility to compute most errors values because we can _**not**_ compute **False Positives** or **True Negatives**. In consequence, we can not compute any error metric such as True Positive Rate, True Negative Rate, False Positive Rate, False Negative Rate, F-Measure, Error Rate, Precision and Accuracy. Without these metrics it is impossible to verify and compare the performance of a given algorithm. 


### Getting Normal Labels
To compute all the error metrics **we need** labels for the **positive class** and the **negative class**. However, if obtaining a malware dataset is difficult, obtaining a normal dataset is much more difficult. To continue with the malware traffic example, to generate a normal dataset is **not enough to execute a non-infected computer for some time**. A normal dataset needs normal behaviors, which means real normal people acting normally. We should think on the normal actions used in the environment where our algorithm is going to work. A normal computer may need to check email, use P2P, chat with partners, use the network, etc.  

> If obtaining a malware dataset is difficult, obtaining a normal dataset is much more difficult.

A related issue with getting a normal dataset is that **it is more time consuming** than to obtain malware data. We can automate the capture of most malware, but we **can't automate** the capture of normal data. All the attempts at automating normal behavior are _not_ useful because humans beings can not be easily mimicked. We can't just sit down in a computer and type _as_ a normal user because our behavior will not be exactly normal and we will be adding a large bias to our dataset. Ideally, we would like to capture a normal user working but this is difficult  since **we need to verify that the normal users are really normal**. This verification may involve checking that the normal computers are clean and checking that the owners are not attacking. In summary, we should **get normal data that is representative of our working context** and we should make sure that the data is in fact **coming from normal sources**.

## The Origin of the Traffic Defines the Labels 
Unfortunately, it is **not enough** to assign a malware label to all the traffic in the malware experiment and to assign a normal label to all the traffic in the normal experiment. The real origin of the traffic defines the label. Consider the case of normal traffic in the malware traffic detection scenario. We may be tempted to assign the normal label to **all the traffic of the IP address of a normal computer**. However, the assumption that all the traffic of a normal IP address is _normal_ is **false**. This is because most of the times an IP address receives traffic from a large number of unknown computers in a network. For example, **it may receive attacks from computers in the Internet**, receive attacks from the internal network, it may receive broadcast NetBIOS request from infected computers, it can have its ports scanned, or it can just receive harmless ARP packets from an incorrectly configured router. **All this traffic is coming to and from the normal IP, but it is certainly not a result of a normal action**. Even more, the normal computer generates traffic in response to those requests, but it **should not be considered normal** since it was not generated as a result of a normal user activity. So, if this traffic is not normal or malware, what is it? It is **background traffic** and as such should be given the **Background label**.

> It is more difficult to assign correct labels than to obtain a dataset.

The same idea must be applied to the malware traffic because is common to find assumptions such as: _if it was a malware capture, all the traffic is malware_. However, a computer infected with the Zeus malware may be still by attacked by an external Bobax bot or receive normal multicast packets from the local printer. The issue of the origin of traffic suggests that it may be a good idea to **differentiate** the **traffic going to** an IP address and the **traffic coming from** an IP address. This is a first approach to separate the normal traffic, the malware traffic and the background traffic.  

### Getting Background Labels
Obtaining background traffic is fairly easy since it corresponds to data that we don't know its origin. Getting this type of traffic **force us to differentiate** between what is malware or normal, and what is not. This traffic should be captured along with the normal and malware traffic in order **to confuse and make things harder for the detection algorithms**. It is **not** the same to find 10 malware flows mixed with 10 normal flows, than to find 10 malware flows mixed with 10 normal flows and **1,000,000 background** flows. The background labels may not be used in some algorithms, but they are a good way of getting a real context.

## Some Labeling Errors are Unavoidable 
Even using a very detailed labeling process we should be aware that **there will be some errors** in the labels. We already discussed that normal traffic may be not completely normal and that malware traffic may include normal traffic. A way of dealing with this is to write down all the possibilities and determine why they may appear in the dataset. Using the malware traffic scenario we may want to find:

- Malware traffic that was labeled as Normal.
    - Because the supposed normal computer was not clean.
- Background traffic that was labeled as Normal.
    - For example external attacks or port scans.
- Normal traffic that was labeled as Malware.
    - Usually because the traffic generated by the infected computer is being automatically generated by the Operating System. Could also be other normal computers contacting the infected one.
- Background traffic that was labeled as Malware.
    - Attacks from Internet to the infected computers.

Listing these errors may help identify possible sources of problems in the experiments and therefore help obtain a better labeled dataset.

## An Instance may Change its Label Due to Evolving Conditions
One of the most common assumptions is that an instance that was, for example, labeled as normal **should always be labeled normal**. In the malware traffic scenario it means that an IP address should not change its label. However, the very idea of detecting malware traffic is that a normal computer may get infected. To reflect this idea of changing labels we can think of a more realistic dataset were a **normal IP address** generates traffic from some time, then gets infected, **starts generating malware** traffic for some days, and finally gets cleaned and generates **normal traffic again**. The important decision is not **which** label this IP may get, but **when should it get the label**. Changing conditions in the experiments should reflect changes in the labels.

## Consider the Balance of the Datasets
Our last problem to consider is the balance of the dataset. It is usually the case that we have more data with the positive label than with the negative label. This is due to the difficulty of obtaining normal data. However, this does not usually reflect a real environment. In a malware traffic scenario most of the traffic is normal and only some small amount is malicious. If our dataset is not correctly balanced, our classification algorithm may benefit the majority class [1]. Although this is related with the [base rate fallacy](http://en.wikipedia.org/wiki/Base_rate_fallacy), it suggests that a **wrongly balanced** dataset may result in **biased** error metrics.

## Conclusion: A Labeling Methodology
A **wrongly labeled** dataset may result in algorithms using the wrong training features, in an insufficient number of metrics reported, in **highly biased detection results**, in **wrong performance metrics** and in a **mistaken idea** of how good the algorithm is. The task of correctly labeling our dataset may seem time consuming but it can teach us a lot about our data and it will provide us with a strong base to evaluate our algorithm. 

To deal with most of these problems it may be a good idea to **have a labeling methodology**. The methodology should at least include a way of testing for the origin of data, the IP addresses in the dataset, the malware types, the amount of labels and the uniqueness of labels. Such a methodology may also assist in the semi-automatic assignment of labels if we use, for example, the [ralabel](http://nsmwiki.org/Argus) tool of the Argus suite that labels network flows using rules. Such a methodology may help us avoid most of the common issues of labeling a dataset.


[1] Kotsiantis, S., Kanellopoulos, D., & Pintelas, P. (2006). Handling imbalanced datasets : A review. GESTS International Transactions on Computer Science and Engineering, 30(1), 25–36.

