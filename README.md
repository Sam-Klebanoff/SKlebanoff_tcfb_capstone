# Sam Klebanoff's Capstone Project for TFCB 2020
This is the repository for the TFCB 2020 capstone

## Project Organization
In this project, I ask three separate questions about the data presented in "Development of machine learning models for diagnosis of glaucoma" (Kim SJ et al. PLOS One, 2017). I then write code in R or Python to answer these computational questions.

My analysis begins with the R markdown file [report_r.Rmd](report_r.Rmd). In this file I start with a section entitled "About the data", where I go through the quality of the authors' published datasets, and the assumptions I made before beginning my anlysis. I then address my first question: Are the random subsets of data used for training/testing both truly representative?

After the end of the R markdown report, my analysis continues in the jupyter notebook file [report_python.ipynb](report_python.ipynb). In this notebook I address my second question: In the dataset provided by the authors, are there more patients whose measurements would cause them to be falsely diagnosed with glaucoma, or falsely diagnosed as healthy? I then go on to address my third question: Within the false positives/negatives, which variables contribute most to the mis-identification of each patient?

At the end of the python notebook, I include a section entitled "Reproducibility", which goes through the reproducibility both of the authors' original analysis, and my own follow-up analysis.

### Data
The full citation for the paper I am analyzing is as follows:

> Development of machine learning models for diagnosis of glaucoma <br>
> Kim S.J., Cho K.J., Oh S. <br>
> (2017)  PLoS ONE,  12  (5) , art. no. e0177726

Their data consists of numerous clinical eye-related measurements, from which they build a machine-learning-based model to predict diagnosis of glaucoma.

The original paper can be found here: https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0177726

The authors' published .csv data files can be found here: https://datadryad.org/stash/dataset/doi:10.5061/dryad.q6ft5

However, since the data files published to Dryad are quite small, I have also uploaded them to this repository, where they can be found inside the "data" folder.

### Figures
For each of my questions, I produce a graphical figure to accompany my conlcusion about the answer to the question. These figures were exported from the R markdown/jupyter notebook into the "figures" folder in this repository.

#### The figure from question one, comparing the training dataset to the test and whole datasets:
![Question 1 Figure](/figures/Q1_training_vs_testing_data-1.png)

#### The figure from question two, comparing false negative and false positive predictions on a PCA plot:
![Question 2 Figure](/figures/Q2_false_positives_vs_false_negatives.png)

#### The figure from question three, showing examples of variables which lead to mis-diagnosis from the model:
![Question 3 Figure](/figures/Q3_variable_differences_between_groups.png)
