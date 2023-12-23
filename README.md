# Building a Bayesian Network for Credit Risk Assessment
Case study dedicated to the application of Bayesian Network model for credit risk assessment.
This is an overview of the master thesis about building a probabilistic graph model and its application in riskology. 
### Introduction
Credit risk assessment consists in calculating the probability that the borrower will not fulfill his financial obligations, such as, for example, repayment of the loan. For financial institutions, this is a key process, as it serves as the basis for making informed decisions. By conducting a comprehensive assessment of credit risk, these institutions have the opportunity to make informed choices about granting credit, setting interest rates, etc.
Credit risk assessment is inherently a complex task. This requires the synthesis of a large amount of diverse information, for example, financial, economic indicators and quality factors, into a coherent structure. To solve such a task, Bayesian networks appear as a powerful analytical tool that offers a modern and reliable approach to credit risk assessment. A Bayesian network is a probabilistic graphical model that provides a formal, holistic representation of knowledge and information that contains uncertainty. This mathematical model is able to reflect the interdependencies and conditional probabilities that underlie credit risk. Using a Bayesian network, it is possible to model the relationships between various factors affecting credit risk, such as income level, employment, credit history, etc. The graphical representation of Bayesian networks allows transparent visualization of these complex relationships, aiding in decision-making and risk management.
### Tools
- Python programming language was chosen for the development and training of the Bayesian network. 
- JupyterLab was used as an IDE for building a model. 
- The Angular frontend framework and the Flask backend microframework were chosen to develop the app for Bayesian network visualization.
- Visual Studio Code code editor was used to develop a web app.
### Bayesian network
A Bayesian network (BN) is a probabilistic graphical model for representing knowledge about an uncertain domain where each node corresponds to a random variable and each edge represents the conditional probability for the corresponding random variables. It is represented in form of Directed Acyclic Graph (DAG) and looks like this:

![image](https://github.com/paulohloblin/bayesian-net-credit-risk/assets/54881219/05f08fdb-1b61-42cb-afc3-f9aab6d781fe)

The Bayesian network requires defining a set of vertices and establishing connections between them. The vertex from which the edge originates is called the parent vertex, while the vertex into which the edge enters is called the child vertex. Child vertices are conditionally dependent on parent vertices, and vertices without parents are conditionally independent. Each vertex of the graph corresponds to a random variable, which is limited by the number of possible states, and these states are mutually exclusive. Tables of conditional probabilities are also defined for each variable network, which indicate the probability of occurrence of certain states of the variable depending on the combinations of known states of the parent variables. Conditional probability tables of a Bayesian network are also called parameters, which are either manually set using expert knowledge or calculated using machine learning algorithms based on a specific data set. Similarly, the structure of the Bayesian network, that is, the number and directions of edges in a directed acyclic graph, can be determined either by knowledge of the subject area, or by machine learning algorithms. The processes of building a structure (or learning it based on data) and determining parameters (or learning them based on data) collectively form a complex process of developing and learning a Bayesian network.
### Training data
The dataset for training the Bayesian network is taken from Kaggle, an open source platform containing datasets for machine learning algorithms. The [Credit Risk dataset](https://www.kaggle.com/datasets/laotse/credit-risk-dataset) contains more than thirty thousand records, each of which represents a separate loan that the client took from the bank. There are a total of 12 different signs that each entry (line) in the table contains. These features contain information about the customer and the loan he took (for example, the customer's age, his income, the amount of the loan, the interest rate, etc.). Among these attributes, there is also the attribute "loan_status", that is, the status of the loan: this is a binary value that takes the value "0" if the client has repaid the loan, and "1" otherwise. The set of signs is combined: among them there are both categorical and numerical ones.
Due to the fact that the data set is synthetic, erroneous data (for example, records where the customer's age is greater than 120 years) were additionally removed and filled with average values where the data were missing (3115 rows). 
Since there are limitations for working with hybrid data and modern libraries do not support work with both hybrid and continuous Bayesian networks, all continuous data was discretized.
### Structure learning
A Bayesian network structure for credit risk assessment was learned from the data set using a machine learning algorithm that incorporated constraints related to the construction of graph arcs. Due to the lack of expert knowledge in the field of lending and credit risks, as well as the presence of a large set of data, it was decided to use the Hill Climb Search algorithm.
This algorithm searches the graph space starting from some initial solution and performing a finite number of steps. At each step, the algorithm considers only local changes, and selects the one that leads to the greatest improvement in the f estimate. The algorithm stops when there are no local changes leading to an improvement in f.
In order to reduce the graph search space, constraints were set at the input of the algorithm. These constraints are a list of arcs that should never be contained in a graph. The list was formed according to the principle of dividing the nodes of the graph (that is, all 12 features from the data set) into the following levels (from the highest to the lowest, respectively): "input data" (`person_age`), "related to age" (`person_emp_length`) , "customer related" (`person_income`, `person_home_ownership`), "previous loan related" (`cb_person_default_on_file`), "current loan related" (`loan_intent`, `loan_grade`, `loan_amnt`, `loan_int_rate`, '`loan_percent_income`, `cb_person_cred_hist_length`), "output" (`loan_status`). The limitations of the algorithm are that it is forbidden to draw directed arcs from nodes of a lower level to any node belonging to a higher level. Also, arcs from `loan_percent_income` to `loan_amnt` and from `loan_percent_income` to `person_income` are prohibited, as loan size and income affect loan size to income, but not vice versa. By incorporating this knowledge into the Hill Climb Search algorithm from the pgmpy library and running this algorithm for execution, we will get a list of arcs that are the searched Bayesian network. 
**Visualize the resulting graph using the GraphWiz tool**:

![graphviz](https://github.com/paulohloblin/bayesian-net-credit-risk/assets/54881219/9899ed48-2005-4d62-9d81-5a2b33cbff15)

### Parameter learning
In the `Pgmpy` library, it is possible to study the parameters, that is, to calculate the values of all conditional probabilities of a given Bayesian network, using Bayesian estimation or using maximum likelihood estimation (MLE). The first method uses an a priori distribution from the data set, the second makes no initial assumptions.
For the MLE method, the phenomenon of overtraining may occur for small data sets, since they may not have enough observations, and therefore the observed frequencies may not be representative. Bayesian estimation, on the other hand, relies not only on the data set to train the network parameters, but also uses prior knowledge expressed in terms of the prior distribution. Thus, this estimate makes some initial assumptions that compensate for missing data.
Although MLE may appear to be optimal in many cases, it may be too crude, whereas the Bayesian approach is inherently more robust and reliable. Therefore, Bayesian estimation was chosen for parameter training.

![image](https://github.com/paulohloblin/bayesian-net-credit-risk/assets/54881219/0cd08169-c7ad-4812-9631-c8c94454bc99)
_CPT for the `person_emp_length` variable_
![image](https://github.com/paulohloblin/bayesian-net-credit-risk/assets/54881219/622fb327-066d-4377-9e76-e11513f32aed)

_CPT for the `person_age` variable_
### Results and inference
After learning the structure and parameters, the Bayesian network can be applied for statistical inference. In Bayesian networks, in order to learn the posterior distribution of a certain variable, not all the vertices of the network graph are needed, but only the parent, child vertices, and the parent vertices of the child vertices. This set is called a Markov blanket, and pgmpy allows you to obtain a Markov blanket for any node. For example, for the `loan_status` node (marked as green), the Markov blanket nodes (marked as blue) look like this

![image](https://github.com/paulohloblin/bayesian-net-credit-risk/assets/54881219/5420a903-e2c7-4b38-ba96-6b4b2aa8efb3)

Thus, in order to assess credit risks, that is, to calculate the probability that the client will not repay the loan, it is enough to know only 4 variables out of 11, namely: the intention of the loan, the ratio of the size of the loan to the client's income, the client's rating and the type of housing in which he lives client. Other variables will affect the posterior distribution only if at least one of the variables from the Markov coverage does not have a certificate.
By default, the pgmpy library uses the variable elimination algorithm for inference.
Some examples of such queries and the results of these queries:
- general distribution among all customers of those who repay loans (value "0") and those who do not repay them (value "1") ![image](https://github.com/paulohloblin/bayesian-net-credit-risk/assets/54881219/4db45e71-51a1-4747-9f23-5a13428c0d0b)
- the probability that the client will and will not pay off the credit debt if he lives in a rented apartment ![image](https://github.com/paulohloblin/bayesian-net-credit-risk/assets/54881219/647348b2-a975-402d-b4ad-46c64da76860)
- the probability that the client will and will not repay the loan if he lives in a rented apartment, but at the same time has the highest credit rating ![image](https://github.com/paulohloblin/bayesian-net-credit-risk/assets/54881219/27fa5944-de0a-4c2f-ac47-14101386b0b5)
- the probability that the client will and will not repay the loan, if he lives in a rented apartment, has the highest credit rating, and the amount of his loan is 40-80% of the amount of annual income
![image](https://github.com/paulohloblin/bayesian-net-credit-risk/assets/54881219/fdee9cb8-7725-47e6-91da-89b44fcbe7bb)

### A web application based on the Bayesian network
A web application was created that visualizes a Bayesian network (BN) and allows interactive interaction with it, making inferences. The application is built on the basis of the Angular and Flask frameworks and allows you to visualize the inference of the Bayesian network and, accordingly, credit risks.
- the app is available by this url: https://paulohloblin.github.io/bayes-credit/
- the source code of the front-end part: https://github.com/paulohloblin/bayes-credit
- the source code of the back-end part: https://github.com/paulohloblin/bayes-credit-backend

The user interface includes:
- nodes of the Bayesian network, which can work in two modes: evidence variable and query variable;
- graph arcs which represent the links between the variables;
- the `current queries` panel, in which the user can conveniently review all the queried variables.

Only one state can be selected for a evidence variable (using a checkbox). To make a probabilistic query to a variable, you need to press the "+" button. In the query variable mode, the user can see the probabilities for all states of the variable, taking into account all available evidence (if there is even one). As evidences change, the probabilities for all polled variables are recalculated automatically. To switch back to evidence variable, press the "-" button.

The main purpose of using BM in this application is to perform credit risk analysis based on available data and uncertainties. This web application provides opportunities to effectively manage loan portfolios and make informed decisions about granting loans. The use of BM allows you to take into account various factors that affect credit risks and conduct analysis based on real data. In addition, the web application can be a useful tool for customers applying for credit. They can use the app to assess the likelihood of getting a loan and identify potential risks.
