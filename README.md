# Dynamic-SBM-by-production-technologies

## 1 Background and Motivation
### 1.1 Background
With the rapid development of Taiwan, the demand for energy, such as electricity, power, etc., has also increased. Nonetheless, one of the by-products of using energy is the trash. When the amount of trash is disposed of, there will be more pollution in the atmosphere and soil, causing environmental damage. Since the world aims at zero pollution, Taiwan, as one of the most sustainable countries, has noticed the severity of pollutants and has started to consider this environmental issue.

In Taiwan, the government will not only deal with environmental problems, but also deal with economic problems. The authorities have the responsibility to help each city in Taiwan address these problems. Therefore, the government has to know the overall conditions of each city, including GDP, the amount of labor and trash, and so on. By doing this, the authorities could help improve the cities. The government could know the performance of the cities by using dynamic SBM: by-production technologies, the overall efficiency and dynamic change in the efficiency of the period of cities in Taiwan from 2014 to 2019.

### 1.2 Motivation
Because dynamic SBM: by-production technologies could improve the accuracy of efficiency measurement and solved the problem of not containing slack variables when measuring the inefficiency rate of traditional radial models [[1]](#1). Therefore, we hope this research can give a good estimation of overall and period efficiency of cities, and can also give an insight to find the efficiency change of each city over the entire observed period.

### 1.3 Paper Reproduction
Tone, K., & Tsutsui, M., 2014. Dynamic DEA with network structure: A slacks-based measure approach. Omega, 42(1), 124-131.

### 1.4 Problem Definition
We would like to calculate the overall and period efficiency of each city in Taiwan from 2014 to 2019. Traditinal methods only calculate the overall efficiency with free disposable input. To measure the efficiency accurately, we use dynamic SBM with by-product technologies to justify the result.


## 2 Methodology
This section 

<a id="2.1"></a> 
### 2.1 Dynamic SBM: by-production technologies
### 2.2 Free Disposable Input
### 2.3 Malmquist-Luenberger Index
When the DMU data is panel data containing multiple time points, the Malmquist total factor productivity index should be used to analyze efficiency change. Chung et al. [[2]](#2) proposed the Malmquist-Luenberger Index (ML index) based on the Malmquist model by applying a DDF containing the undesirable output. Any Malmquist index with undesirable output is called the ML productivity index [[1]](#1). Based on the method proposed by Chung et al. [[2]](#2), Li et al. [[1]](#1) proposed ML equation, combining the SBM model with undesirable output. Since their ML equation combining SBM model with undesirable output corresponds to our Dynamic SBM: by-production technologies [[2.1]](#2.1), we used their ML equation to implement Malquist-Luenberger Index. Its fixed ML index equation constructed in this paper is as follows: the common reference set of each period is $S_f = {(x^f_j, y^f_j, b^f_j)}$ , and $f$ is the number of 1 ~ period $p$. Because the referenced EPPF in each period is fixed, a single ML index is calculated. $ML^f_0 ( x^{t+1}_0 ,  y^{t+1}_0,  b^{t+1}_0,  x^t_0,  y^t_0,  b^t_0)  =  E^f_0 ( x^{t+1}_0 ,  y^{t+1}_0,  b^{t+1}_0 ) / E^f_0 ( x^t_0 ,  y^t_0,  b^t_0 )$. 

The efficiency change of $DMU_0$ from $t$ to $t+1$ period is represented by $ML^f_0 ( x^{t+1}_0 ,  y^{t+1}_0,  b^{t+1}_0,  x^t_0,  y^t_0,  b^t_0)$. Among them, $E^f_0 ( x^t_0 ,  y^t_0,  b^t_0 )$ and $E^f_0 ( x^{t+1}_0 ,  y^{t+1}_0,  b^{t+1}_0 )$ represent the efficiency values of $DMU_0$ in $t$ period and $t+1$ period. $ML^f_0 ( x^{t+1}_0 ,  y^{t+1}_0,  b^{t+1}_0,  x^t_0,  y^t_0,  b^t_0) = 1$ indicates that efficiency remains unchanged, $ML^f_0 ( x^{t+1}_0 ,  y^{t+1}_0,  b^{t+1}_0,  x^t_0,  y^t_0,  b^t_0) > 1$ indicates that efficiency increases, and $ML^f_0 ( x^{t+1}_0 ,  y^{t+1}_0,  b^{t+1}_0,  x^t_0,  y^t_0,  b^t_0)< 1$ indicates that efficiency decreases. 


## 3 Data Collection and Analysis Result
This section will explain how the data we used were collected and the columns of the data. The following is the analysis results of comparison between by-production and free-disposable input as well as the efficiency change from Malmquist-Luenberger Index.  

### 3.1 Data Collection
Our dataset includes five kinds of data: non pollution input-tax revenue (million), pollution input-coal (million ton), desirable output-GDP (million), undesirable output-trash (ton), and free/fixed link-labor (thousand people). Each city in Taiwan has the data from 2014 to 2019. The data of tax revenue, trash, and labor were obtained by [national statistics](https://winsta.dgbas.gov.tw/DgbasWeb/ZWeb/StateFile_ZWeb.aspx). The amount of coal and GDP were multiplied the amount of coal in use and GDP person by the number of population in cities. The calculation of the amount of coal per person is first to convert the amount of used electricity to the amount of used coal in Taiwan. Next, the amount of used coal in each city is in proportion to the number of population. As for the GDP per person, we first gained GDP per person from 2014 to 2019 from [national statistics](https://winsta.dgbas.gov.tw/DgbasWeb/ZWeb/StateFile_ZWeb.aspx), and then multiplied the number of population by GDP per person to get GDP of each city.


## 4 Conclusion

## Reference
### Data

### Paper
<a id="1">[1]</a> 
Li, Y., & Chen, Y. (2021). Development of an SBM-ML model for the measurement of green total factor productivity: The case of pearl river delta urban agglomeration. Renewable and Sustainable Energy Reviews, 145, 111131.
<a id="1">[2]</a> 
Chung, Y. H., Färe, R., & Grosskopf, S. (1997). Productivity and undesirable outputs: a directional distance function approach. journal of Environmental Management, 51(3), 229-240.

### Others
