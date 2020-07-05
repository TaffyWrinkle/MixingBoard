# MixingBoard: a Knowledgeable Stylized Integrated Text Generation Platform

# News

# Setup

## Packages
We recommend using [Anaconda](https://www.anaconda.com/) to setup
Firstly, create an environment with Python 3.6
```
conda create -n mixingboard python=3.6
conda activate mixingboard
```
Then, install Python packages with
```
sh setup.sh
```

## API Accounts
Then, if you prefer to use the web search and text-to-speech functions, please apply the following accounts.
* **Bing Search API**: open an account and/or try for free on [Azure Cognitive Services](https://azure.microsoft.com/en-us/services/cognitive-services/bing-web-search-api/). Once you obtained the key, please put it in `args/api.tsv`. You can also try other search engine, however we currently only support Bing Search v7.0 in `src/knowledge.py`.
* **Text-to-Speech**: open an account and/or try for free on [Azure Cognitive Services](https://azure.microsoft.com/en-us/services/cognitive-services/text-to-speech/). Once you obtained the key, please put it in `args/api.tsv`.


## `pick_tokens`
Please implement your own `pick_tokens` function in `src/todo.py`. This function is used to pick tokens for a generation time step given predicted token probability distribution. Many choices are available, e.g. greedy, top-k, top-p, or sampling.


# Modules

## Knowledge passage retrieval
We use the following unstructured free-text sources to retrieve relevant knowledge passage: search engine, specialized websites (e.g. wikipedia), and user provided document.
```
python src/knowledge.py
```
The above command calls Bing search API and the following shows results of an example query.
```
QUERY:  what is deep learning?

URL:    https://en.wikipedia.org/wiki/Deep_learning
TXT:    Deep learning is a class of machine learning algorithms that (pp199–200) uses multiple layers to progressively extract higher level features from the raw input. For example, in image processing, lower layers may identify edges, while higher layers may identify the concepts relevant to a human such as digits or letters or faces.. Overview. Most modern deep learning models are based on ...

URL:    https://machinelearningmastery.com/what-is-deep-learning/
TXT:    Deep Learning is Large Neural Networks. Andrew Ng from Coursera and Chief Scientist at Baidu Research formally founded Google Brain that eventually resulted in the productization of deep learning technologies across a large number of Google services.. He has spoken and written a lot about what deep learning is and is a good place to start. In early talks on deep learning, Andrew described deep ...

URL:    https://www.forbes.com/sites/bernardmarr/2018/10/01/what-is-deep-learning-ai-a-simple-guide-with-8-practical-examples/
TXT:    Since deep-learning algorithms require a ton of data to learn from, this increase in data creation is one reason that deep learning capabilities have grown in recent years.
```

## Open-ended dialogue generation
We use [DialoGPT](https://github.com/microsoft/DialoGPT) as an example.
```
python src/open_dialog.py
```
The following shows DialoGPT (`DPT`) predictions of an example query using one implementation of the `pick_tokens` function.
```
CONTEXT:        What's your dream?
DPT 1.008       First one is to be a professional footballer. Second one is to be a dad. Third one is to be a firefighter.
DPT 1.007       First one is to be a professional footballer. Second one is to be a dad. Third one is to be a father of two.
...
```

## Machine reading comprehension
```
python src/mrc.py
```
The above command calls [BiDAF](https://allenai.github.io/bi-att-flow/) model. Given a passage from a [Wikipedia page](https://en.wikipedia.org/wiki/Geoffrey_Hinton) and an example query, it returns the following results
```
QUERY:          Who is Jeffrey Hinton?
PASSAGE:        Geoffrey Everest Hinton CC FRS FRSC is an English Canadian cognitive psychologist and computer scientist, most noted for his work on artificial neural networks. Since 2013 he divides his time working for Google and the University of Toronto. In 2017, he cofounded and became the Chief Scientific Advisor of the Vector Institute in Toronto.
Bidaf 0.352     an English Canadian cognitive psychologist and computer scientist
```

## Text-to-speech
```
python src/tts.py
```
The above command calls [Microsoft Azure Text-to-Speech API](https://azure.microsoft.com/en-us/services/cognitive-services/text-to-speech/), saves and plays the audio. The following is one example.
```
TXT:    Hello there, welcome to the Mixing Board repo!
audio saved to voice/hellotherewelcometothemixingboardrepo_en-US-JessaNeural.wav
```

## Ranking
We consider multiple metrics to rank the hypotheses, including 1) forward and reverse generation likelihood, 2) repetition penalty, 3) informativeness, and 4) style intensity. 
```
python src/ranker.py
```
Following are some examples of the the above command.
```
TXT:    This is a normal sentence.
rep -0.0000 info 0.1619 score 0.1619

TXT:    This is a repetive and repetive sentence.
rep -0.1429 info 0.2518 score 0.1089

TXT:    This is a informative sentence from the MixingBoard GitHub repo.
rep -0.0000 info 0.4416 score 0.4416
```

# Dialog Demo

## Comand-line interface
The comand-line interface can be started with the following command.
```
python src/demo_dialog.py cmd
```
## Webpage interface
```
python src/demo_dialog.py web
```
The comand above creates a webpage demo that can be visited by typing `localhost:5000` in your browser. You can interact with the models, and the following screenshot is an example
![](https://github.com/microsoft/MixingBoard/blob/master/fig/dialog_web_demo.PNG)

## RESTful API
```
python src/demo_dialog.py api
```
Runing the command above on your machine `A` (say its IP address is `IP_address_A`) starts to host the models on machine `A` with a RESTful API. Then, you can call this API on another machine, say machine `B`, with the following command, using "what is machine learning?" as an example context
```
curl IP_address_A:5000 -d "context=what is machine learning?" -X GET
```
which will returns a json object, in the following format
```json
{
  "context": "what is machine learning?", 
  "passages": [[
      "https://en.wikipedia.org/wiki/Machine_learning", 
      "Machine learning (ML) is the study of computer algorithms that improve automatically through experience. It is seen as a subset of artificial intelligence.Machine learning algorithms build a mathematical model based on sample data, known as \"training data\", in order to make predictions or decisions without being explicitly programmed to do so. Machine learning algorithms are used in a wide ..."
    ]], 
  "responses": [
    {
      "rep": -0.0, "info": 0.4280192169639406, "fwd": 0.014708111993968487, "rvs": 0.10698941218944846, "score": 0.5497167508995263, "way": "Bidaf", 
      "hyp": "computer algorithms that improve automatically through experience"}, 
    {
      "rep": -0.0, "info": 0.24637171873352778, "fwd": 0.16426260769367218, "rvs": 0.05065313921885011, "score": 0.46128747495542344, "way": "DPT", 
      "hyp": "I believe that is a fancy way to say artificial intelligence."}, 
    {
      "rep": -0.1428571428571429, "info": 0.22310269295193919, "fwd": 0.1599835902452469, "rvs": 0.21712445686414383, "score": 0.4573535985050974, "way": "DPT", 
      "hyp": "I believe that is a fancy way to put it. Machine learning is a set of algorithms and algorithms are machines."}, 
  ]}
```
Besides calling API by `curl`, you can also lanch a webpage demo on machine `B`, but using the backend running on machine `A` with the API, using the following command
```
python src/demo_dialog.py web --remote=IP_address_A:5000 --port=5001
```

# Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.opensource.microsoft.com.

When you submit a pull request, a CLA bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

# Disclaimer

# Citation

If you use this code in your work, you can cite our [arxiv](https://arxiv.org/abs/2005.08365) paper:

```
@article{gao2020mixingboard,
  title={MixingBoard: a Knowledgeable Stylized Integrated Text Generation Platform},
  author={Gao, Xiang and Galley, Michel and Dolan, Bill},
  journal={Proc. of ACL},
  year={2020}
}
```