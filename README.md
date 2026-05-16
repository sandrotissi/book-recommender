# 📚 University Library Book Recommendation System

**Team Name:** Berne 

## 👥 Team Members & Contributions
* **Tissi Sandro**: Worked mostly on the recommendation system code, EDA and Kaggle ranking.
* **Philipona Adrien**: Worked mostly on the user interface and the video.
  Both team member were actually involved in each steps and contributed to new ideas and debugging. 

## Video 
**Link:** [YouTube Video](https://youtu.be/m8BrKFdmduc)

## 🏆 Kaggle Competition Results
* **Final MAP@10 / Precision@10 Score:** **0.1984**
* **Final Rank:** **1st**
* **Screenshot:** 



## 📊 Exploratory Data Analysis (EDA)
During our data exploration, we identified the following key insights:
1.  **Many users interact several times with the same book:** 23'044 out of 87'047 interactions are rereads. This means that 26.47% of the interactions are users interacting with a book that they already interacted with before.
<br/>
<img width="650" height="400" alt="image" src="https://github.com/user-attachments/assets/c69b6e49-bd3e-4cf8-83e5-cecd257a3e8c" />

<br/>
<br/>

2.  **Not all books are reread:** The percentage of rereads is high but the probability varies between books. For the recommendations, we took this into account by giving books with higher rereading probabilities a higher weight. 
<br/>
<img width="243" height="295" alt="image" src="https://github.com/user-attachments/assets/c54cff67-8e0e-4414-b256-762d226aefc3" />


<br/>
<br/>

3.  **Recognizing patterns in the data:** Plotting the heatmap of the interactions revealed that the interactions are not random. There seems to be a line of interactions that increases with the user ID (u) and the book ID (i). In the interaction data, there are no observations on the right side of this line. The higher the user ID (u), the more interactions users have on the left side of the line.
<br/>
<img width="827" height="689" alt="image" src="https://github.com/user-attachments/assets/373f9851-522a-462e-817b-7468a2a4ce34" />


<br/>





## Data Augmentation & Embeddings
We didn't use any additional metadata to improve our model.
To provide rich metadata for the UI, we augmented the dataset with covers, authors (where it was missing), and book summaries from external APIs (Google Books and OpenLibrary).



## ⚙️ Model Creation, Evaluation & Results
To create our hybrid recommendation system, we constructed two baseline collaborative filtering models and combined them with custom, domain-specific features. First, we built an Item-Item Collaborative Filtering model by calculating the cosine similarity between the columns of our interaction matrix, allowing us to predict user preferences based on how closely a book's consumption pattern aligns with books they have already interacted with. Alongside this, we implemented a User-User Collaborative Filtering model that evaluates similarity across rows, capturing shared tastes between different readers to generate recommendations. Moving beyond standard collaborative filtering, we integrated two distinct features tailored to reading behavior. We ingested an external dataset of rereading probabilities, isolating a subset of 5,103 books that carry a greater than X% chance of being read again, and created a filtered matrix that explicitly preserves these past interactions while discarding low-probability ones. Additionally, we engineered a sequential feature that targets potential series-reading behaviors by automatically recommending the next two consecutive item IDs directly following the maximum item ID a user had registered in the training set. In the next phase of our pipeline, we combined these individual components into a single, ensembled predictive model using a linear weighting scheme. Our strategy heavily favors user loyalty and historical preference, assigning X% of the total weight to the filtered, high-probability rereading matrix. The remaining influence is distributed equally among the other signals, with the User-User CF predictions, Item-Item CF predictions, and the next-in-sequence recommendations each receiving a X% weight. For a specific subset of the population (the first 3,400 users) the final prediction score blends all four components, while for any users outside this threshold, the model dynamically adapts by excluding the sequential next-item metric and combining only the collaborative filtering and rereading components. Once these weighted scores were calculated across the entire matrix, we sorted the predicted values for each user to extract their top 10 highest-scoring books, formatted these recommendations into a space-separated string, and successfully compiled the final output into a structured submission file named weighted_model.csv.<br/>
<br/>
We evaluated our models using a local Train/Test/Cross-Validation split to calculate the true Precision@10 and Recall@10. 

| Technique | Precision@10 | Recall@10 |
| :--- | :---: | :---: |
| User-User CF | [0.0476] | [0.2213] |
| Item-Item CF | [0.0464] | [0.2002] |
| **Weighted Approach with Rereading Recommendations** | **[0.0574]** | **[0.2419]** |

### Which is the best model?
The **Weighted Approach with Rereading Recommendations** performed the best. Some books have a relatively high probability to be reread. Combining the User-User CF, Item-Item CF with recommendations based on books that are oftenly reread lead to a higher precision. When user data is sparse, user-user similarity often fails because there are too few overlapping data points to find meaningful "neighbors," leaving recommendations inaccurate or impossible for new users. By combining it with item-item similarity, the system can bypass the need for shared user history by recommending items similar to the few a user has interacted with, effectively filling the gaps in the sparse interaction matrix. <br/>
To identify the optimal configuration for our hybrid recommender system, we performed an extensive grid search, systematically testing various combinations of the parameters (weighting of different models, thresholds for reread etc.) to maximize the model's predictive accuracy.



## 🔍 Qualitative Analysis: "Good" vs. "Bad" Predictions
We analyzed the recommendations generated by our best model for specific users to see if they align with their rental history.

### ✅ Examples of "Good" Predictions
* **User ID 2:** * **User History:** This person mostly rented graphic narratives and specifically manga and graphic novels. We can see in her history that she started different books, like for example she already rented 7 times Kokkoku, but there is 8 books of this one.
    * **Model Recommendation:** The model actually recommend Kokkoku. It also recommended other mangas like Erased or Le sablier.
    * **Why it makes sense:** It is really accurate as she still need to read the last volume and other recommendations are also mangas.

### ❌ Examples of "Bad" Predictions
* **User ID 'new':**
    * **User History:** No history as it is a new user.
    * **Model Recommendation:** The first recommendation is the le petit robert. There is also recommendation for Pons Kompaktwörterbuch or Soins infirmiers : médecine et chirurgie. 
    * **Why it failed:** This model is built only on the most rented books. Nowaday, with AI and internet, dictionnaries are not so usefull anymore. They are probably rented a lot for exams, where no digital are allowed. However, for a random user coming for the first time the dictionnary will not be a good recommendation. Regarding the medecin book, the issue is probably that it is mandatory for a medecin course to rent it so many user are renting it, although medecin student represent a small fraction of the library users so it is also not a pertinent recommendation. 



## User Interface 
We built an interactive UI using Streamlit that allows users to input a User ID and see their personnal book recommendations. If a user is not yet registered, we still make recommendations to them based on the trending picks (most rented books). 
Once users are identified, they see visually appealing book recommendations complete with cover images fetched via the google book api (downloaded csv with links) and completed with openlibrary, they also directly get informations on on the author and the synopsis of the book. This approach helped us to collect a lot of book covers, for example user 567 is missing only one cover! They can also see their history of rented books, with the precise date.

You can access the app by clicking on this link: [**https://bcul-book-recommender.streamlit.app**](https://bcul-book-recommender.streamlit.app)
If the link is expired, no worries. 
Go to the the green code button on the top right, and select the legendary acorn codespace. Than go on the recommender_app folder and select streamlit_app.py. Than paste this code into the terminal to access the app: pip install streamlit pandas requests && streamlit run recommender_app/streamlit_app.py

[link](https://github.com/ad-phil/Book-recommender)
pip install streamlit pandas requests && streamlit run streamlit_app.py
