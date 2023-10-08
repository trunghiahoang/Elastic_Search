# Elasticsearch: How to deploy NLP: text embedding and vector search
![enter image description here](https://img-blog.csdnimg.cn/761456fbdd9f4bfbb997dca00e448478.png)

As part of our Natural Language Processing (NLP) blog series, we'll introduce an example that uses a text embedding model to generate vector representations of text content and demonstrates a vector similarity search on the generated vectors. We will deploy a publicly available model on Elasticsearch and use it in the ingestion pipeline to generate embeddings from text documents. We then show how these embeddings can be used in vector similarity searches to find semantically similar documents for a given query. 

Vector similarity search , or commonly known as semantic search, goes beyond traditional keyword-based search and allows users to find semantically similar documents that may not have any keywords in common, thus providing a broader range of results. Vector similarity search operates on dense vectors and uses k-nearest neighbor search to find similar vectors. To do this, you first need to convert the content in the form of text into its numerical vector representation using a text embedding model. 

We will demonstrate using a public dataset from the MS MARCO Passage Ranking Task . It consists of real questions and human-generated answers from the Microsoft Bing search engine. This dataset is a perfect resource for testing vector similarity searches, firstly, because question answering is one of the most common use cases for vector search, and secondly, the top papers in the MS MARCO rankings use vector search in some form.

 In our example, we will use a sample of this dataset, use the model to generate text embeddings, and then run a vector search on it. We also wanted to quickly verify the quality of the results produced by the vector search. In today's demonstration, I will use Elastic Stack 8.2 to demonstrate. 

For a vector search, its architecture can be expressed as follows:

![enter image description here](https://img-blog.csdnimg.cn/523d49d5aa384887a2f34d1fd3e5672c.png)
![enter image description here](https://img-blog.csdnimg.cn/3b050972d809472aa02902382a9861b1.png)

Install
 Elasticsearch 及 Kibana

 If you have not installed your own Elasticsearch and Kibana, please refer to the following article to install it: 
 - How to install Elasticsearch on Linux, MacOS and Windows
 -  Kibana: How to install Kibana in the Elastic stack on Linux, MacOS and Windows 

Please note the 8.x installation section of the article. Since using eland to upload models is a function of the Platinum or Enterprise Edition, in our demonstration, we need to enable the Platinum Edition trial function:
![enter image description here](https://img-blog.csdnimg.cn/8f014aac8de043efbc5a1e582a695ff8.png)
![enter image description here](https://img-blog.csdnimg.cn/37e1e67600444f4fb358323ecd835162.png)
![enter image description here](https://img-blog.csdnimg.cn/bbfdb2461fb140d99b8f2e7c7b6f6d65.png)

Eland 
Eland can be installed from PyPI using Pip:

    python -m pip install eland

You can also install Eland from the Conda Forge using Conda:

    conda -c install conda-forge eland

 Users who wish to use Eland without installing it, in order to run only the available scripts, can build a Docker container:

    https://github.com/elastic/eland.git
    cd eland
    docker build -t elastic/eland .
    

Eland encapsulates the conversion and chunking process of the Hugging Face converter model into its TorchScript representation in a Python method; therefore, this is the recommended import method.

 Install the Eland Python client .

Run the eland_import_hub_model script. For example:

    eland_import_hub_model \
          --url http://localhost:9200/ \
          --hub-model-id elastic/distilbert-base-cased-finetuned-conll03-english \
          --task-type ner \
          --start

 Specify the URL to access your cluster. For example, https://<user>:<password>@<hostname>:<port>. 
 Specify the model's identifier in the Hugging Face Model Center. 
 Specify the type of NLP task. Supported values ​​are fill_mask, ner, text_classification, text_embedding, question_answering and zero_shot_classification.
![enter image description here](https://img-blog.csdnimg.cn/cfee5d914b944a8c8c0e4f48d4b8c275.png)

Deploy text embedding model 

The first step is to install the text embedding model. For our model, we use Hugging Face ’s msmarco-distilbert-base-tas-b . This is a sentence transformation model that maps a sentence or a paragraph to a 768-dimensional dense vector. The model is optimized for semantic search and trained specifically on the MS MARCO Passage dataset, making it suitable for our task. In addition to this model, Elasticsearch also supports many other text embedding models. The full list can be found here. 

We use the Eland docker agent installation model we built in the NER sample . Run the following script to import our model into our local cluster and deploy it:

Above, please note that you need to replace the username and password parts above according to your own situation. You also need to modify the corresponding Elasticsearch address. 
Here, since we are using a self-signed installation, I installed using the --insecuer option to circumvent SSL's security certificate check.

If we want to use a self-signed certificate to ensure secure uploading, we can use the following command:

Above, we used the -v flag to install a local http_ca.crt certificate into the container in docker. The result of running the above command is:
