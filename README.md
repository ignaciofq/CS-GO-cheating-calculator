# CS:GO Cheating Calculator
Creating a Machine Learning model to calculate the probability of a given player being a cheater. The goal is to explore the implementation and key points of Machine Learning models as an additional tool to combat cheating.

The below sections provide a high-level overview of the project. For detailed implementation steps, data visualization and code snippets, please refer to the corresponding Jupyter Notebooks and files in the repository detailed here:
### Files overview
1. **dataset_creation.ipynb**: _Jupyter Notebook containing the process of creating a dataset with Steam IDs and their CS:GO statistics._  
2. **dataset.xlsx**: _The initial dataset mentioned above. SteamIDs have been redacted to protect privacy._  
3. **model_training.ipynb**: _Jupyter Notebook covering the data exploration, feature creation, and model training processes._  
4. **players_stats.xlsx**: _A clean and processed dataset that includes new features for model training._  
5. **cheater_proba_rf_classifier.joblib**: _Trained model exported in the joblib format._  
6. **cheater_probability_calculator.ipynb**: _A standalone calculator that applies the model to out-of-sample accounts for quick cheater probability calculation._  

# Index
[0. Introduction](https://github.com/ignaciofq/CS-GO-cheating-calculator/blob/main/README.md#-introduction)  
[1. Data Collection, Cleaning and Preprocessing](https://github.com/ignaciofq/CS-GO-cheating-calculator/blob/main/README.md#1-data-collection-cleaning-and-preprocessing)  
[2. Exploratory Data Analysis and Feature Creation](https://github.com/ignaciofq/CS-GO-cheating-calculator/blob/main/README.md#2-exploratory-data-analysis-and-feature-creation)  
[3. Model Selection, Training and Evaluation](https://github.com/ignaciofq/CS-GO-cheating-calculator/blob/main/README.md#3-model-selection-training-and-evaluation)  
[4. Limitations](https://github.com/ignaciofq/CS-GO-cheating-calculator/blob/main/README.md#4-limitations)
[5. Application Examples](https://github.com/ignaciofq/CS-GO-cheating-calculator/blob/main/README.md#4-application-examples)  
[6. Conclusion and Further Analysis](https://github.com/ignaciofq/CS-GO-cheating-calculator/blob/main/README.md#5-conclusion-and-further-analysis)  

# Introduction 
This project is a demonstration of the application of Machine Learning algorithms in the fight against cheating. The game of choice is Counter-Strike: Global Offensive thanks to the extensive documentation on [Steamworks Web API](https://partner.steamgames.com/doc/webapi) and the availability of different stat-tracking sources. This made it ~~almost~~ possible to obtain the needed data to create this project. 

Due to critical data limitations ([4. Limitations](https://github.com/ignaciofq/CS-GO-cheating-calculator/blob/main/README.md#4-limitations)), this specific model's accuracy is not reliable. However, any developer with a publisher API key or a studio with direct access to their player's stats and ban status query will be able to generate a reliable-enough dataset.   
# 1. Data Collection, Cleaning and Preprocessing
The very first step was to create two separated lists of Steam IDs, one for banned players and one for experienced players that have not been banned. The sources used for the straction are two:

1. **[Tracker.gg](https://tracker.gg/csgo/leaderboards/stats/all/MatchesPlayed?page=800)**  
Extracted a list of 1,300 players with a substantial number of games played. This ensures that the players used to train the model have reliable and representative average statistics, minimizing the impact of outliers.
2. **[VACList](https://vaclist.net/banned)**  
Due to privacy concerns, it's technically impossible to find a list of VACBanned accounts specifically for CS:GO. VAClist is the general go-to page to consult daily VAC Bans. Therefore, the initial list of 1,200 banned players has been retrieved from here.
   
With the IDs ready, the stats were retrieved using the Steam Web API. The initial dataset consists of Steam64 IDs of 1,622 CS:GO players together with their stats and VAC Status.

The collected data is cleaned and preprocessed to prepare it for further analysis: removing irrelevant or duplicate entries, filtering out missing data, and converting columns to the appropriate data types.

# 2. Exploratory Data Analysis and Feature Creation
New features (statistics that will train the model) were created based on the available data to enhance the predictive power of the Machine Learning model. An Exploratory Data Analysis (EDA) was performed to gain insights into the data and understand the patterns, relationships, and distributions of the different features. The optimal features are selected based on their correlation with cheating behavior, statistical significance and availability, and relevance to the game dynamics (traditionally observed metrics). 

The features of the model are: 

1. **Win Percentage `win_%`:** Calculates the percentage of matches won.
2. **Kill-to-Death Ratio `kd`:** Measures the ratio of kills to deaths.
3. **Headshot Percentage `hs_%`:** Represents the percentage of kills that are headshots.
4. **Accuracy `accuracy`:** Indicates the ratio of shots hit to shots fired.
5. **MVP Percentage `mvp_%`:** Measures the percentage of rounds won as the Most Valuable Player.
6. **Damage per Round `dmg_round`:** Calculates the average damage inflicted per round.
7. **Contribution per Round `contribution_round`:** Represents the average contribution score per round.`

The data is now normalized using a logarithmic function (log1p) to compress the scale of the data and mitigate the effect of outliers. The final dataset is exported as "player_stats.xlsx".

To explore the correlation matrix and density plots, please refer to _[model_training.ipynb](https://github.com/ignaciofq/CS-GO-cheating-calculator/blob/main/model_training.ipynb)_.

# 3. Model Selection, Training and Evaluation
Following the standard process of Machine Learning models, we've split the data in training and test set, tried out different classification logarithms and evaluated them to choose the one with the best accuracy. During the evaluation, we observe that the Random Forests algorithm performs significantly better than Logistic Regression and therefore is the chosen model to continue the project. Since we are looking for a probability and not just binary classification, it's mandatory to use an algorithm that allows probability calculation.
#### Model Performance
The classification model achieved an accuracy of 89% in predicting whether a player is a cheater or not (macro average). It also shows a strong capacity to distinguish classes (ROC AUC Score: 0.94).  
Confusion Matrix:   
>True negatives (TN): 253 non-cheaters correctly classified as non-cheaters.  
>False positives (FP): 32 non-cheaters incorrectly classified as cheaters.  
>False negatives (FN): 26 cheaters incorrectly classified as non-cheaters.  
>True positives (TP): 227 cheaters correctly classified as cheaters.  

To review the Classification Report, Confusion Matrix and ROC AUC Score, please refer to _[model_training.ipynb](https://github.com/ignaciofq/CS-GO-cheating-calculator/blob/main/model_training.ipynb)_.
# 4. Limitations
**General Limitations:**
- The games, their content, their telemetry systems and users' behaviors (together with underlying patterns) change over time. These models will need to be constantly reviewed to adapt to evolving cheating techniques and player strategies.
- There will always be cases that fall in the gray zone between classes, leading to legitimate players being classified as cheaters.
- Distinguishing between a top-tier player and a highly skilled cheater can be challenging, as both may exhibit exceptional statistics.

**Specific Limitations to This Model:**
- Choosing exclusively players of around 5,000 games experience for the Class 0 (legitimate players) represents a bias in the data selection. However, this ensures a realistic and normalized average without outliers, which can be very positive for the model training.
- The size of the training set used for the banned players is concerningly small, which will limit the generalization and performance on unseen data.
- Additionally, we have no guarantee that the VACbann was received for cheating specifically in CS:GO (there are over 100 Steam games that use Valve Anti-Cheat). **This point is enough to question the accuracy of the model and results of the evaluation.**
- The accuracy data does not take weapon type into consideration, missing insights into specific cheating behaviors, like the use of the Scout sniper, for example.
- We have no guarantee that the experienced players are not cheaters.

These limitations should be taken into account when interpreting the model's results and making decisions based on its predictions.
# 5. Application Examples
This project is considered demonstrative as the limitations pose a concern on the model's accuracy. However, these steps can be followed by game developers to implement in their multiplayer games a statistical automated cheat detection. Even with strong data foundations, it would be recommendable to implement a high cutoff (e.g only ban above 90% chance of being a cheater) even at the cost of catching less cheaters, as the risk of banning innocent people could be considered a high one.

This tool can also be used for consultation for Customer Support teams when handling ban appeals, for example in edge cases where the player was banned by the community (Overwatch/Manual ban systems).

Of course, feel free to use this calculator to judge your oponents in matchmaking and make educated accusations of cheating.
# 6. Conclusion
As with any project involving player behavior and cheating detection, there are several ethical considerations that need to be addressed:

1. **Privacy and Data Protection**: The collection and analysis of player data raise concerns about privacy and data protection. This project only retrieves public information, however, game developers may have access to private data that would be extremely interesting to explore for a model like this. Not being able to consult if the VACBan is specific for CS:GO has been a mayor limitation to this model. It goes without saying that a correct and accurate initial dataset is crucial to train accurate and reliable ML models. 

2. **False Positives and Fairness**: The model's predictions, as accurate and refined as they may be, may result in false positives, flagging innocent players as potential cheaters. Striking the right balance between accurately identifying cheaters and minimizing false accusations is essential to ensure fairness and avoid harming innocent players' reputations.    
I believe that a certain degree of false positive is acceptable as long as there is a well-structured process to handle ban appeals and the respective Support Team is equipped with the right tools to make an educated decision.

4. **Transparency and Explainability**: Machine Learning models often operate as black boxes, making it challenging to explain the reasoning behind their predictions. This is probably the trickiest point, as it is important to explain to your playerbase your efforts to fight cheating, while at the same time making the process as opaque as possible for cheaters to adapt to.

5. **Bias and Discrimination**: Biases can inadvertently be present in the collected data or the model itself, leading to discriminatory outcomes. For example, this model has a substantial bias in the "experienced" player pool selected and the decision to train the model with experienced players and not with a random selection of players. It is crucial to either mitigate or justify your bias.
