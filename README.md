# Using ChatGPT to generate substitute data for language model classifier training

# PROBLEM

Back in 2018 - 2019 I worked on training Machine Learning models to predict which sentences in company conference call transcripts were likely to be "guidance" or forward looking statements.  

The most challenging part of the project editing Excel files with 50,000 lines of text and flagging each sentence as "guidance" or "not guidance".  This was done to have a training set that could then be used to train the ML model.

It was a mind numbing job that took several weeks.  In general, data tagging for machine learning is both incredibly important to get right, and incredibly expensive in terms of time, particularly if domain knowledge is required.

Recently I ran across an article by Piero Paialunga who used ChatGPT for <b>substitute data</b> - i.e. substitute tagging - for sentiment analysis:
https://towardsdatascience.com/hands-on-sentiment-analysis-on-hotels-reviews-using-artificial-intelligence-and-open-ais-chatgpt-d1939850c79e

Often in the real world we're trying to train on no-label data, particularly in specialized fields like investing.  There are techniques for dealing with this like zero-shot classification and data augmentation, but I found the ChatGPT angle of interest - could it live up to the hype?

So I set about trying to use ChatGPT to skip the 50,000 lines of tagging part of the "is it guidance" project, and see if the process Piero outlined could render useful results in a more specific problem than simple sentiment scoring.

## User Story:<BR>
<b>AS</b> A financial analyst<BR>
<b>I WANT TO</b> score guidance sentences in transcripts <BR>
<b>SO THAT</b> I can save time reading the whole transcript and don't miss market moving statements <BR>

# PROCESS

I used two sentences from a recent Discover call (DFS) ... one guidance and one not.

![image](https://user-images.githubusercontent.com/39496491/218327367-df3761cb-680e-423f-8d8b-d0fcef27accc.png)

I then used completion = openai.Completion.create(engine="text-curie-001", prompt=guidance_example,max_tokens=240) to <b>generate</b> 500 "guidance" sentences and 500 "not guidance" sentences using these two sentences as seeds for ChatGPT.

This took about a half hour to run, and repeated runs ended up costing me about $17.00 for my API developer key usage for the day.  
But you can save $17.00 by skipping this step and using the generated_snippets.csv file in this repo.
  
Interestingly, the results were, I thought, fairly realistic.  ChatGPT is impressive.  All of the snippets below are generated text.

![image](https://user-images.githubusercontent.com/39496491/218328051-b8cc4b4e-fbf3-48d6-94a8-56738f9a7fb7.png)

Next I trained two classifiers, one Random Forest and one Logistic Regression.  Piero used RF which is probably more accurate in training but I find in the real world the predict_proba() ability of LR to assign a stochastic value to the sentence is of use - since binary classifiers often don't offer much value as much as a gradient of probabilties.

As you might expect from ChatGPT the fits were good for both models:

![image](https://user-images.githubusercontent.com/39496491/218328318-81af1c14-78c0-42c7-9e2e-feb451797c61.png)

Next I wanted to try these models on raw data.  <b>Into the wild we go:</b>

# CONCLUSION

I used Capital One's most recent conference call transcript, filtered (by hand) down to just management talking.

![image](https://user-images.githubusercontent.com/39496491/218328386-2300ebd1-7b4a-4927-b6de-cd56f84b15a7.png)

As I feared, the RF model classed the majority of management statements as "guidance".  
(This is a Type 1 Error which probably falls out from the domain-specific issue of "financial guidance" vs ChatGPT being more generally trained language model.  Also, the English language doesn't actually have a future tense - a strange quirk which bedeviled my work in this area in prior years and which makes simple "tense" classifiers inaccurate in practice in English.)
  
![image](https://user-images.githubusercontent.com/39496491/218328436-d88132ee-54b7-4f8e-b5fa-56c8313e6329.png)

I then used the LR model in order to rank order the snippets, and the results were ... not too bad!

![image](https://user-images.githubusercontent.com/39496491/218328476-19d04e88-4002-4be8-bc67-8fdc18be291a.png)

<b>TLDR</b>:<br> The results filtered out via Linear Regression were ok, though I'd want to spend a LOT more time thinking about the problem.  
  
Is this really a SUMMARIZATION problem not a CLASSIFICATION problem per se?  
  
Might it make sense to evaluate statements in context vs as individual tokens?  Would a HUGGINGFACE transformer maybe auto gen text better than ChatGPT?  

All these questions to be explored in future!





