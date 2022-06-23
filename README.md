# Dynamic-SBM-by-production-technologies

## 1 Background and Motivation
### 1.1 Background
With the rapid development of Taiwan, the demand for energy, such as electricity, power, etc., has also increased. Nonetheless, one of the by-products of using energy is the trash. When the amount of trash is disposed of, there will be more pollution in the atmosphere and soil, causing environmental damage. Since the world aims at zero pollution, Taiwan, as one of the most sustainable countries, has noticed the severity of pollutants and has started to consider this environmental issue.

In Taiwan, the government will not only deal with environmental problems, but also deal with economic problems. The authorities have the responsibility to help each city in Taiwan address these problems. Therefore, the government has to know the overall conditions of each city, including GDP, the amount of labor and trash, and so on. By doing this, the authorities could help improve the cities. The government could know the performance of the cities by using dynamic SBM: by-production technologies, the overall efficiency and dynamic change in the efficiency of the period of cities in Taiwan from 2014 to 2019.

### 1.2 Motivation
Because dynamic SBM: by-production technologies could improve the accuracy of efficiency measurement and solved the problem of not containing slack variables when measuring the inefficiency rate of traditional radial models [[1]](#1). Therefore, we hope this research could give a good estimation of overall and period efficiency of cities, and also give an insight to find the efficiency change of each city over the entire observed period. Especially, we would like to provide the efficiency analysis for **human resources department** which we regarded it as the most important decision-making unit in the government based on the population in all province levels. 

### 1.3 Problem Definition
We would like to calculate the overall and period efficiency of each city in Taiwan from 2014 to 2019. Traditinal methods only calculate the overall efficiency with free disposable input. To measure the efficiency accurately, we used dynamic SBM with by-product technologies to justify the result.


## 2 Methodology
We first constructed a model based on Dynamic DEA: A slack-based measure approach published by Tone and Tsutsui [[2]](#2), and then deal with bad output by by-production method instead of considering it as free disposable inputs.  
We also try to calculate Malmquist-Luenberger Index to find out the efficiency change through the periods.


### 2.1 Dynamic SBM: consider bad output as free disposable input

#### Indices
$i$: input  
$j$: output  
$k$: DMU  
$t$: period  
$l$: carry-over

#### Sets
$I$: inputs  
$J$: outputs  
$K$: DMUs  
$T$: periods  
$L_{free}$: free carry-overs

#### Parameters
$X^t_{ki}$: $i$ th input of DMU $k$ at period $t$  
$Y^t_{kj}$: $j$ th output of DMU $k$ at period $t$  

#### Decision Variables
$\lambda^t_k$: the intensity weights of the linear combination between DMU $r$ and DMU $k$ at period $t$ , multiply τ  
$s_i^{t-}$: slack of $i$ th input at period $t$ , multiply τ  
$s_j^{t+}$: slack of $j$ th output at period $t$ , multiply τ  
$s^{tC}_{l}$: slack of $l$ th free carry-over at period $t$ , multiply τ  
τ: for linearizing the NLP

#### Model

Below is the formulation of dynamic SBM in the paper [[2]](#2), and we choose the non-oriented model.  
The constraint (1) is the **output constraint**, constraint (2) is the **input constraint**, and constraint (3) and (4) are for **conservation of mass** of each carry-overs. We let the left hand side of constraint (5) be 1 so that the objective function can be linear. And constraint (6) is for VRS.

<img width="435" alt="DSBM_free disposable input" src="https://user-images.githubusercontent.com/47711803/175364642-5fea9174-5b57-4e26-936f-df210d63a811.png">

After finding out the overall efficiency for each DMU, we can calculate the corresponding period efficiency by the function below.

<img width="459" alt="period efficiency" src="https://user-images.githubusercontent.com/47711803/175375635-eafafcda-dd54-4d7c-a65a-0bb0fafd9aa9.png">


#### Source Code(python-gurobi)

We use python with gurobi to implement the model.  
**Carry_Labor**, **Input_Tax**, **Input_Population**, **Output_Trash**, **Output_GDP** are the carry-overs, the non-pollution-causing inputs, the pollution-causing inputs, the bad outputs, and the good outputs, respectively. **K** is the set of the Cities of Taiwan (DMU), and **Year** is the set of the periods.

```python
def DynamicSBM(Carry_Labor, Input_Tax, Input_Population, Output_Trash, Output_GDP, K, Year):
    T = len(Year)
    Type = ['VRS_free', 'VRS_fix', 'CRS_free', 'CRS_fix']

    C = {'Labor'} # carry over
    M = {'GDP'} # Outputs
    N = {'Tax', 'Population', 'Trash'} # Inputs

    X = {} #Input
    Z = {} # free carry over
    for k in range(len(K)):
        Z[K[k],'Labor'] = Carry_Labor[k]
        X[K[k],'Population'] = Input_Population[k]
        X[K[k],'Tax'] = Input_Tax[k]
        X[K[k],'Trash'] = Output_Trash[k]
    Y = {} # Ouput
    for k in range(len(K)):
        Y[K[k],'GDP'] = Output_GDP[k]  
        
    D_SBM = {}
    no_year = {}

    for i in range(len(Type)):
        all_temp = {}
        part_temp = {}
        for firm in K:
            model = Model('DynamicSBM')
            model.setParam('OutputFlag', 0)

            Lambda = {}
            a = model.addVar(vtype = "C", name = "a", lb=0.0000001)
            S_free, S_Npos, S_Nmin = {}, {}, {}
            for t in range(T):
                for k in K:
                    Lambda[k, t] = model.addVar(vtype = "C", name = "Lambda(%s)(%s)"%(k, t),lb=0)
                for m in M:
                    S_Npos[m, t] = model.addVar(vtype = "C", name = "S_N+(%s)(%s)"%(m, t),lb=0)
                for n in N:
                    S_Nmin[n, t] = model.addVar(vtype = "C", name = "S_N-(%s)(%s)"%(n, t),lb=0)
                for c in C:
                    S_free[c, t] = model.addVar(vtype = "C", name = "S_free(%s)(%s)"%(c, t),lb=-GRB.INFINITY)
            model.update()
            model.setObjective(quicksum(a - quicksum(S_Nmin[n, t]/X[firm, n][t] for n in N) / len(N) for t in range(T)) / T, GRB.MINIMIZE)
            for t in range(T):
                model.addConstr(1 == a + quicksum(S_Npos[m, t]/Y[firm, m][t] for m in M) / len(M))
                if(i < 2):
                    model.addConstr(quicksum(Lambda[k, t] for k in K) == 1)
                for m in M:
                    model.addConstr(quicksum(Lambda[k, t] * Y[k, m][t] for k in K) - S_Npos[m, t] == a * Y[firm, m][t])
                for n in N:
                    model.addConstr(quicksum(Lambda[k, t] * X[k, n][t] for k in K) + S_Nmin[n, t] == a * X[firm, n][t])
            for t in range(T-1):
                for c in C:
                    if(i % 2 == 0):
                        model.addConstr(quicksum(Lambda[k, t] * Z[k, c][t] for k in K) + S_free[c, t] == a * Z[firm, c][t])
                    else:   
                        model.addConstr(quicksum(Lambda[k, t] * Z[k, c][t] for k in K) == a * Z[firm, c][t]) # fix
                    model.addConstr(quicksum(Lambda[k, t] * Z[k, c][t] for k in K) == quicksum(Lambda[k, t + 1] * Z[k, c][t] for k in K))
            model.optimize()
            all_temp[firm] = model.objVal

            temp = []
            for t in range(T):
                temp.append(LinExpr.getValue(a - (LinExpr.getValue(quicksum(S_Nmin[n, t]/X[firm, n][t] for n in N)/ len(N)))))
            part_temp[firm] = temp
        D_SBM[Type[i]] = all_temp
        no_year[Type[i]] = part_temp
    return D_SBM, no_year
```

With the overall efficiency $\theta$, we can calculate the period efficiency of each DMU.  
However, this method consider bad output as free disposable input, but there will be some problem at free disposable hole, e.g., good output increases while the bad output remain the same. Therefore we try the by-production method.




<a id="2.2"></a> 
### 2.2 Dynamic SBM: by-production technologies
#### Indices
$i$: input  
$j$: output  
$k$: DMU  
$t$: period  
$l$: carry-over

#### Sets
$I^N$: non-pollution-causing inputs  
$I^P$: pollution-causing inputs  
$J^N$: good outputs  
$J^P$: bad outputs  
$K$: DMUs  
$T$: periods  
$L_{free}$: free carry-overs

#### Parameters
$X^t_{ki}$: $i$ th input of DMU $k$ at period $t$  
$Y^t_{kj}$: $j$ th good output of DMU $k$ at period $t$  
$B^t_{kj}$: $j$ th bad output of DMU $k$ at period $t$

#### Decision Variables
$\lambda^t_k$: the intensity weights of the linear combination between DMU $r$ and DMU $k$ at period $t$, multiply τ  
$\mu^t_k$: the intensity weights of the linear combination between DMU $r$ and DMU $k$ at period $t$, multiply τ  
$s_i^{tN-}$: slack of $i$ th non-pollution-causing input at period $t$, multiply τ (frontier of good output)  
$s_j^{tN+}$: slack of $j$ th good output at period $t$, multiply τ (frontier of good output)  
$s_i^{tGP-}$: slack of $i$ th pollution-causing input at period $t$, multiply τ (frontier of good output)  
$s_j^{tP+}$: slack of $j$ th bad output at period $t$, multiply τ (frontier of bad output)  
$s_i^{tBP-}$: slack of $i$ th pollution-causing input at period $t$, multiply τ (frontier of bad output)  
$s^{tC}_{l}$: slack of $i$ th free carry-over at period $t$, multiply τ  
τ: for linearizing the NLP

#### Model
Based on the formulation in [[2.1]](#2.1), we developed dynamic SBM with by-production technology.  
Below is our formulation. Constraint (1), (2), (3) form the **frontier of good output**, constraint (4) and (5) form the **frontier of bad output**, and constraint (6) and (7) are for **conservation of mass** of each carry-overs. We let the left hand side of constraint (8) be 1 so that the objective function can be linear. And constraint (9) and (10) are for VRS.

<img width="491" alt="DSBM_by-production" src="https://user-images.githubusercontent.com/47711803/175364674-e60a1a80-3142-460b-8a8a-9fa550bb99e5.png">

After finding out the overall efficiency for each DMU, we can calculate the corresponding period efficiency by the function below.

<img width="459" alt="period efficiency" src="https://user-images.githubusercontent.com/47711803/175375635-eafafcda-dd54-4d7c-a65a-0bb0fafd9aa9.png">


#### Source Code(python-gurobi)

We use python with gurobi to implement the model.  
**Carry_Labor**, **Input_Tax**, **Input_Population**, **Output_Trash**, **Output_GDP** are the carry-overs, the non-pollution-causing inputs, the pollution-causing inputs, the bad outputs, and the good outputs, respectively. **K** is the set of the Cities of Taiwan (DMU), and **Year** is the set of the periods.

```python
def DynamicSBM_Byproduction(Carry_Labor, Input_Tax, Input_Population, Output_Trash, Output_GDP, K, Year):
    T = len(Year)
    Type = ['bp_VRS_free', 'bp_VRS_fix', 'bp_CRS_free', 'bp_CRS_fix']
    C = {'Labor'} # carry over
    M = {'GDP'} # desirable outputs
    J = {'Trash'} # undesirable outputs
    N = {'Tax'} # sub non-pollution-causing inputs
    P = {'Population'} # sub pollution-causing inputs
    
    X = {} #Input
    Z = {} # free carry over
    for k in range(len(K)):
        Z[K[k],'Labor'] = Carry_Labor[k]
        X[K[k],'Population'] = Input_Population[k]
        X[K[k],'Tax'] = Input_Tax[k]
    Y = {} # Good Ouput
    B = {}  # Bad Output
    for k in range(len(K)):
        B[K[k],'Trash'] = Output_Trash[k]
        Y[K[k],'GDP'] = Output_GDP[k]   
        
    BP_D_SBM = {}
    bp_year = {}

    for i in range(len(Type)):
        all_temp = {}
        part_temp = {}
        for firm in K:
            model = Model('byProductDynamicSBM')
            model.setParam('OutputFlag', 0)

            Lambda, Mu = {}, {}
            a = model.addVar(vtype = "C", name = "a",lb=0.0000001)
            S_free, S_Npos, S_Nmin, S_GPmin, S_Ppos, S_BPmin = {}, {}, {}, {}, {}, {}
            for t in range(T):
                for k in K:
                    Lambda[k, t] = model.addVar(vtype = "C", name = "Lambda(%s)(%s)"%(k, t),lb=0)
                    Mu[k, t] = model.addVar(vtype = "C", name = "Mu(%s)(%s)"%(k, t),lb=0)
                for m in M:
                    S_Npos[m, t] = model.addVar(vtype = "C", name = "S_N+(%s)(%s)"%(m, t),lb=0)
                for j in J:
                    S_Ppos[j, t] = model.addVar(vtype = "C", name = "S_P+(%s)(%s)"%(j, t),lb=0)
                for n in N:
                    S_Nmin[n, t] = model.addVar(vtype = "C", name = "S_N-(%s)(%s)"%(n, t),lb=0)
                for c in C:
                    S_free[c, t] = model.addVar(vtype = "C", name = "S_free(%s)(%s)"%(c, t),lb=-GRB.INFINITY)
                for p in P:
                    S_GPmin[p, t] = model.addVar(vtype = "C", name = "S_GP-(%s)(%s)"%(p, t),lb=0)
                    S_BPmin[p, t] = model.addVar(vtype = "C", name = "S_BP-(%s)(%s)"%(p, t),lb=0)
            model.update()
            model.setObjective(quicksum(a - (quicksum(S_Nmin[n, t]/X[firm, n][t] for n in N) \
                + (quicksum(S_GPmin[p, t]/X[firm, p][t] for p in P) + quicksum(S_BPmin[p, t]/X[firm, p][t] for p in P)) / 2) \
                / (len(N) + len(P)) for t in range(T)) / T, GRB.MINIMIZE)
            for t in range(T):
                model.addConstr(1 == a + (quicksum(S_Npos[m, t]/Y[firm, m][t] for m in M) + quicksum(S_Ppos[j, t]/B[firm, j][t] for j in J)) / (len(M) + len(J)))
                if(i < 2):
                    model.addConstr(quicksum(Lambda[k, t] for k in K) == 1)
                    model.addConstr(quicksum(Mu[k, t] for k in K) == 1)

                for m in M:
                    model.addConstr(quicksum(Lambda[k, t] * Y[k, m][t] for k in K) - S_Npos[m, t] == a * Y[firm, m][t])
                for j in J:
                    model.addConstr(quicksum(Mu[k, t] * B[k, j][t] for k in K) - S_Ppos[j, t] == a * B[firm, j][t])
                for n in N:
                    model.addConstr(quicksum(Lambda[k, t] * X[k, n][t] for k in K) + S_Nmin[n, t] == a * X[firm, n][t])
                for p in P:
                    model.addConstr(quicksum(Lambda[k, t] * X[k, p][t] for k in K) + S_GPmin[p, t] == a * X[firm, p][t])
                    model.addConstr(quicksum(Mu[k, t] * X[k, p][t] for k in K) + S_BPmin[p, t] == a * X[firm, p][t])
            for t in range(T-1):
                for c in C:
                    if(i % 2 == 0):
                        model.addConstr(quicksum(Lambda[k, t] * Z[k, c][t] for k in K) + S_free[c, t] == a * Z[firm, c][t])
                    else:
                        model.addConstr(quicksum(Lambda[k, t] * Z[k, c][t] for k in K) == a * Z[firm, c][t]) # fix
                    model.addConstr(quicksum(Lambda[k, t] * Z[k, c][t] for k in K) == quicksum(Lambda[k, t + 1] * Z[k, c][t] for k in K))
            model.optimize()
            all_temp[firm] = model.objVal

            temp = []
            for t in range(T):
                temp.append((LinExpr.getValue(a - (LinExpr.getValue(quicksum(S_Nmin[n, t]/X[firm, n][t] for n in N)) 
                                + (LinExpr.getValue((quicksum(S_GPmin[p, t]/X[firm, p][t] for p in P)))
                                + LinExpr.getValue(quicksum(S_BPmin[p, t]/X[firm, p][t] for p in P)))/2) / (len(N) + len(P)))))
            part_temp[firm] = temp
        BP_D_SBM[Type[i]] = all_temp
        bp_year[Type[i]] = part_temp
    return BP_D_SBM, bp_year
```

With the overall efficiency $\theta$, we can calculate the period efficiency of each DMU, and observe the trend of efficiency growth in order to find out the way to improve efficiency.




### 2.3 Malmquist-Luenberger Index
When the DMU data is panel data containing multiple time points, the Malmquist total factor productivity index should be used to analyze efficiency change. Chung et al. [[3]](#3) proposed the Malmquist-Luenberger Index (ML index) based on the Malmquist model by applying a DDF containing the undesirable output. Any Malmquist index with undesirable output is called the ML productivity index [[1]](#1). Based on the method proposed by Chung et al. [[3]](#3), Li et al. [[1]](#1) proposed ML equation, combining the SBM model with undesirable output. Since their ML equation combining SBM model with undesirable output corresponds to our Dynamic SBM: by-production technologies [[2.2]](#2.2), we used their ML equation to implement Malquist-Luenberger Index. Its fixed ML index equation constructed in this paper is as follows: the common reference set of each period is $S_f = {(x^f_j, y^f_j, b^f_j)}$ , and $f$ is the number of 1 ~ period $p$. A single ML index is calculated. $ML^f_0 ( x^{t+1}_0 ,  y^{t+1}_0,  b^{t+1}_0,  x^t_0,  y^t_0,  b^t_0)  =  E^f_0 ( x^{t+1}_0 ,  y^{t+1}_0,  b^{t+1}_0 ) / E^f_0 ( x^t_0 ,  y^t_0,  b^t_0 )$. 

The efficiency change of $DMU_0$ from $t$ to $t+1$ period is represented by $ML^f_0 ( x^{t+1}_0 ,  y^{t+1}_0,  b^{t+1}_0,  x^t_0,  y^t_0,  b^t_0)$. Among them, $E^f_0 ( x^t_0 ,  y^t_0,  b^t_0 )$ and $E^f_0 ( x^{t+1}_0 ,  y^{t+1}_0,  b^{t+1}_0 )$ represent the efficiency values of $DMU_0$ in $t$ period and $t+1$ period. $ML^f_0 ( x^{t+1}_0 ,  y^{t+1}_0,  b^{t+1}_0,  x^t_0,  y^t_0,  b^t_0) = 1$ indicates that efficiency remains unchanged, $ML^f_0 ( x^{t+1}_0 ,  y^{t+1}_0,  b^{t+1}_0,  x^t_0,  y^t_0,  b^t_0) > 1$ indicates that efficiency increases, and $ML^f_0 ( x^{t+1}_0 ,  y^{t+1}_0,  b^{t+1}_0,  x^t_0,  y^t_0,  b^t_0)< 1$ indicates that efficiency decreases. [[1]](#1)

However, since we cannot solve the infeasible problems of Malmquist-Luenberger Index, the results of the efficiency change from Malmquist-Luenberger Index will not be shown in the next section.


## 3 Data Collection and Analysis Result
This section will explain how the data we used were collected and the columns of the data. The following is the analysis results of comparison between by-production and free-disposable input.  

### 3.1 Data Collection
Our dataset includes five kinds of data: non pollution input-tax revenue (million), population (people), desirable output-GDP (million), undesirable output-trash (ton), and free/fixed link-labor (thousand people). Each city in Taiwan has the data from 2014 to 2019. The data of tax revenue, trash, population, and labor were obtained by [national statistics](https://winsta.dgbas.gov.tw/DgbasWeb/ZWeb/StateFile_ZWeb.aspx). The amount of GDP were multiplied GDP person by the number of population in cities. For the GDP per person, we first gained GDP per person from 2014 to 2019 from [national statistics](https://winsta.dgbas.gov.tw/DgbasWeb/ZWeb/StateFile_ZWeb.aspx), and then multiplied the number of population by GDP per person to get GDP of each city.

### 3.2 Result of by-production Dynamic SBM (DSBM)
After formulating our dynamic SBM with by-production technology, we got the overall efficiency of each city. According to the result, the most efficient city was 彰化, and we might infer that because recently, 彰化 has been going on large-scale of constructions, and its housing price has been sky-rocketing, so the efficiency would be high. On the other hand, the most inefficient city was 金門, and we might infer that because the economy of 金門 has been relying on tourism industry, and the tourists were not improving, so the efficiency would be low. Also, one of the special results we have get was 台北, because in common, we might consider Taipei as efficient for the reason that it is the capital of Taiwan. However, it might be because the space was not enough and pollution was high, so all in all it’s not a good circumstances for 台北 to develop its economy, and because Taipei is the capital, so it must be full of laborers, thus, the efficiency might be lower than expected. Viewing the abnormal result, the government could pay more attention to the efficiency of Taipei.

Below is the result of dynamice SBM with by-production technology.

![DSBM_BP_result](images/DSBM_BP_result.png) (DSBM stands for the results of by-production Dynamice SBM)

### 3.3 Comparison between by-production and free disposable input
We had also formulated the original dynamic SBM, which consider bad output as free disposable. The chart is about the comparison of our by-production and the free disposable method, the x coordinate is each DMU, and the y coordinate is the overall efficiency. Most of the cities got lower overall efficiency by our method, which show by using our model, we can get a more precise estimation of the efficiency, and the variance of each city was shown more clearly by our method, also. Thus, we can differentiate efficient cites from inefficiency cities.

![Overall.png](images/Overall.png)

After estimating the overall efficiency of each city, we had also derived deeper into the efficiency of each city in each year, to get more insight, and we had selected four cities as examples. 

The two of them were 彰化 and 金門. In the free disposable method, the result might lead to the conclusion that they were both efficient, and we could hardly see the increase or decrease from year to year. On the other hand, using the by-production model, we could see that actually, 彰化 was more efficient than 金門, and 彰化 was the most efficient one and 金門 was the most inefficient one according to the overall efficiency. Also, according to the result we got from our by-production model, we could actually show the change in efficiency in each year, to help further analysis for the government. However, if we consider the original free-disposable method, we could only infer that the two cities were both efficient in each year.

![彰化縣.png](images/彰化縣.png)
![金門縣.png](images/金門縣.png)

The next two of the examples were 宜蘭 and 臺東. In 宜蘭, we could see that in 2017, it had a peak, and by our model, we could more clearly observer that, and the peak according to the raw data, although the GDP were not severely changed, but the tax had decreased, so it would show as a peak. In 臺東, we could see that by our model, it kept decreasing its efficiency, it could a good warning for 台東 to show that the efficiency of it has been actually decreasing, and it might work hard to come out with a solution to deal with it.

![宜蘭縣.png](images/宜蘭縣.png)
![臺東縣.png](images/臺東縣.png)

### 3.4 Comparison between each area

Although we had get the data and result of each city, the input or output, etc, could not easily be divided into cities in the real world. The cities in the same area might interfere with each others. Thus, we had also experiment on the average period efficiency of each period. According to the result, 中部 and 南部 were the most efficient, for the reason that they were still developing rapidly. Also, 離島 and 東部 were the most inefficient, for the reason that they mostly depend on the tourism industry. Last, 北部 fell in the middle.

![AreaPeriod.png](images/AreaPeriod.png)


## 4 Conclusion
In conclusion, we used dynamic SBM to analyze the efficiency of each period. Compared with free-disposable inputs, we used by-production technologies to know more about the difference of efficiency between periods and DMUs. Next, we combined the advantages of the two and proposed dynamic SBM and by-production. With this method, we analyzed the efficiency of each city in Taiwan from 2014 to 2019. The results showed the trend of efficiency over the periods. Take Changhua as an example. eg: even though Changhua’s overall efficiency is high, its trend can show Changhua regresses in some periods. In addition, the results showed Taipei is not as efficient as we expected, Changhua outperformed other cities, and Taitung & Kinmen fall behind. The government could take these results into consideration. Finally, we hope the human resources department of Taiwanese government could use the efficiency analysis we proposed to help the cities improve, and in the long term, Taiwan can get better than before. 
 


## Reference
### Data
[National statistics](https://statdb.dgbas.gov.tw/pxweb/dialog/CityItemlist_n.asp)

### Paper
<a id="1">[1]</a> 
Li, Y., & Chen, Y. (2021). Development of an SBM-ML model for the measurement of green total factor productivity: The case of pearl river delta urban agglomeration. Renewable and Sustainable Energy Reviews, 145, 111131. <br>
<a id="2">[2]</a> 
Tone, K., Tsutsui, M., 2010. Dynamic DEA: A slack-based measure approach. Omega: Int. J. Manage. Sci. 38, 45–156. <br>
<a id="3">[3]</a> 
Chung, Y. H., Färe, R., & Grosskopf, S. (1997). Productivity and undesirable outputs: a directional distance function approach. journal of Environmental Management, 51(3), 229-240.
