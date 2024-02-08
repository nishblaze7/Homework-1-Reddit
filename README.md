In this project, we present an approach which would involve leveraging Prodigy, a data annotation tool, to preprocess and train a Named Entity Recognition (NER) model on a dataset of annotated Reddit comments from the Cooking subreddit. The primary objective of this model is to classify entities into three categories: Dish, Ingredient, and Equipment; this will provide us the ability to address inconsistencies in the data that could potentially come when annotations are being performed by multiple annotators without any system of secondary review.

Our goal is to develop a model that can accurately classify text into each of our categories based on its contextual meaning rather than a general dictionary definition. It will be important to ensure that the model is capable of discerning the different use-cases of different words or phrases related to our different categories, as it is important to consider that the same words can often have vastly different sentiments or interpretations based on how they are used within a sentence. A tool that is able to consistently and accurately make these distinctions could have the potential to help develop a sort of proof-reading process that could parse manual annotations and make them better. By designing a model that could assist in confirming the validity of manual annotations based on the context of the text being analyzed, we could ensure that there is less error in the data collection process and make sure that the categorization process is standardized, which is much more difficult to accomplish when it comes to manual annotations.

Approach :
We collected a dataset of 1,250 annotated Reddit comments labeled with NER annotations for Dish, Ingredient, and Equipment. Additionally, we obtained 750 unlabeled Reddit comments and 500 LLM zero-shot annotated (GPT3.5) comments for further analysis.

We used Prodigy to preprocess the annotated dataset. First, we loaded the annotated comments into Prodigy and reviewed the annotations to identify inconsistencies and noise. We then used Prodigy's annotation interface to correct and refine the annotations, ensuring consistency and accuracy.

Annotation guidelines :

Since the intent of this model is to categorize different culinary or cooking related terms/phrases, we felt it was important to ensure that our categories were clearly defined so we could ensure that the training process was consistent and resulted in a model that could be considered an accurate classifier. To accomplish this main objective, our model utilizes three different categories in its classification process: “Dish”, “Ingredient”, and “Equipment”. Each of these categories can be best understood as follows:

Dish: This category refers to a fully prepared or completed item or meal, which usually consists of different individual components or ingredients that are combined in a specific series of events to compile the aforementioned final product. This is the final result of a cooking process/procedure and are what people consume during a meal.
Ingredient: An ingredient refers to an individual component or resource in its raw form; ingredients are a single item which is utilized in the preparation of a dish, often being consolidated into a complete product of some sort. Ingredients can be seen as the puzzle pieces and a dish can be seen as the entire, completed puzzle. 
Equipment: This category references any kind of implement used to aid or ease the cooking, serving or preparation process associated with a particular dish or ingredient. Equipment helps facilitate, and in some cases even accelerate, different processes associated with food; some equipment may be essential to different dishes or ingredients. Although equipment may be a part of the preparation, cooking or serving process, these items are never something that is directly consumed.

Our manual annotations involved categorizing all of the given texts based on the guidelines listed above, and there are no notable exceptions or additional rules that we found it was necessary to apply to our manual annotation processes. We found it was especially important to consider the context around each of the different terms we categorized and quickly found that it can be difficult to identify which pieces of a phrase are associated with a particular term. For example, some annotations that would be particularly tricky for the model to accurately classify are listed below, along with a brief description of what makes the sentence difficult for the model to categorize:

Lots of different food related acronyms:
"I make a delicious layered casserole; it is filled with creamy cheese, ground meat, and lasagna noodles, and there are always leftovers."

Humor or wordplay; dual interpretations/meanings:
"As the flour settled, the baker realized the importance of dough in both the bread-making process and life."

Ingredients that are particularly uncommon or niche:
"The avant-garde chef presented a molecular gastronomy creation featuring foie gras, edible flowers, and truffle foam."

Mixing together lots of different terms and categories along with unfamiliar terms:
"The chef, armed with a mandoline, expertly sliced potatoes for the gratin, creating layers of cheesy goodness in the baking dish."

Speaking metaphorically:
"The restaurant's success was the result of a well-oiled machine, with the staff functioning like a finely tuned oven."

Prodigy ner. Methods: 

The pipeline we chose for the refinement process was particularly crucial to the success of our model, as we found that it was thoroughly utilized throughout the preparation/tuning stages of model development. Breaking down our chosen pipeline helps give us a better understanding of why it was chosen for our use-case; for example, the ‘tok2vec’ piece of the pipeline assists our process by tokenizing input text, which involves breaking down sentences into their individual components, including punctuation or words. This is essential for making sure that our data is well suited for analysis and structured in a manner that will allow us to accurately distinguish between the different components of a sentence. Additionally, the ‘ner’ component of our pipeline represents the “named entity recognition” part of the model and its analysis. This component helps the model understand how to accurately identify instances based on the information collected from previous training data.

To enhance the dataset and improve model performance, we employed Prodigy's ner.teach workflow. This allowed us to iteratively annotate the data with the assistance of the NER model, focusing on instances where the model was uncertain or where annotations were inconsistent. After about 250 annotations using ner.teach, we decided to train the model again using both the original dataset and the new dataset; although we were unable to successfully employ the db.merge command, we got around this by adding a ” dataset” to the prodigy train command. After these changes, the model returned an accuracy score of 0.51. 

Considering the success of this initial improvement, we decided that utilizing Prodigy's ner.correct workflow to further refine the annotations and correct errors identified during the active learning process would be beneficial to the model and provide more accurate guidelines for future annotations. To make sure of this, we spent additional time manually reviewing and correcting the model's predictions to ensure that its annotations would be accurate and remain consistent throughout the data it analyzed. After approximately 200 annotations, we decided to train the model again with this new information; this time, the model showed additional improvement and returned an accuracy score of 0.61. 

We developed a custom NER model using Prodigy's train-curve command, incorporating the preprocessed and refined dataset to ensure the model performed to the best of its abilities based on the information we had collected and generated. We used a variety of evaluation metrics, including accuracy, precision, recall, and F1 score, to assess the model's performance and make sure that it was focusing on the aspects of classification which were most important to our specific use-case.
The train-curve command resulted in the following output:- 
Training 4 times with 25%, 50%, 75%, 100% of the data

%      Score    ner   
----   ------   ------
  0%   0.00     0.00  
 25%   0.27 ▲   0.27 ▲
 50%   0.42 ▲   0.42 ▲
 75%   0.49 ▲   0.49 ▲
100%   0.58 ▲   0.58 ▲

We then trained the dataset again and we used label statistics to better understand where our model was succeeding, and where it was performing particularly poorly. We ca
                             P         R          F
EQUIPMENT    77.78   63.64   70.00
INGREDIENT   55.00   56.41   55.70
DISH                 75.00   27.27   40.00

Results: 
Our approach resulted in a trained NER model with an accuracy score of 0.61 on the test dataset; although this accuracy is not particularly impressive, our decision to employ ner.teach and ner.correct increased the effectiveness of our model and may have a more significant impact as additional manual annotations help train the model further. By leveraging Prodigy's annotation workflows, we were able to address data inconsistency issues and improve the quality of the dataset, leading to better model performance. During our initial use of the train-curve command, we received a pop-up message which stated that “As a rule of thumb, if accuracy increases in the last segment, this could indicate that collecting more annotations of the same type will improve the model further.”; we utilized this information by going back through and making sure that there were no additional improvements that ner.teach and ner.correct could offer. In terms of classification accuracy, we found that in its final form, our model did best at classifying “Equipment” and “Ingredients” in the given texts; this makes sense, as for most people, these are the two categories which are easiest defined and least ambiguous in terms of their use case. On the other hand, we found that the model struggled with the “Dish” category most often; we thought this was particularly interesting, as we found that this was one of the harder categories to accurately classify during our manual annotations. This shows that our model still faces some challenges in its classification techniques and could benefit from further improvement, including on-going annotation refinement and model tuning to further enhance accuracy and robustness.



Conclusion: 

In conclusion, our approach emphasizes the effectiveness of using Prodigy for data preprocessing, annotation, and model training in NER tasks. Although our accuracy score was not particularly impressive, we demonstrated that a model could help address data inconsistency issues by leveraging active learning and error correction workflows. The processes we used to maximize the effectiveness of our model could ultimately help improve the quality of our labeled data and make sure that future annotation processes are able to accurately capture and categorize data. Moving forward, it would be beneficial to continue refinement and iteration of the NER model, as a more precise and reliable NER model would further enhance the accuracy and utility this tool could provide in future use-cases. In addition to the continuous refinement of the model, further exploration into how we could provide the model with a better understanding of how context and other advanced linguistic concepts could potentially enhance the model's ability to better discern more complex or unfamiliar terms. In summary, the results of our work successfully accomplished our initial objectives and speak to the importance of understanding the best approach to similar problems that may arise in the future of natural language processing and entity recognition.


