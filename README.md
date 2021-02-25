# German Legal Sentences

German Legal Sentences (GLS) is an automatically generated training dataset for semantic sentence matching and citation recommendation in the domain in german legal documents. It follows the concept of weak supervision, where imperfect labels are generated using multiple heuristics. For this purpose we use a combination of legal citation matching and BM25 similarity. The contained sentences and their citations are parsed from real judicial decisions provided by [Open Legal Data](http://openlegaldata.io/) ([Paper](https://arxiv.org/abs/2005.13342)).


GLS consists of sentences from German judicial decisions that contain at least one legal citation.  


## Why German Legal Documents?

In recent years, many efforts have been made to make German court decisions freely available to the public. Although a decent number of decisions can now be found on the Internet, it can be difficult for the layperson to find relevant information in them. With this dataset, we want to accelerate research on semantic search technologies in German domain-specific language. We hope that emerging search technologies will enable the layperson to find relevant information without knowing the specific terms used by the professionals. Furthermore, we think that the lessons learned due the work with this dataset can be applied to other domains and languages.

## Examples 

## Download

The current version of the Dataset can be downloaded as a zip file: [GermanLegalSentences_v0.0.2.zip](http://lavis.cs.hs-rm.de/storage/german-legal-sentences/GermanLegalSentences_v0.0.2.zip) 

### Contents

| Filename               | Description                          | Num Records | Size    | Format                  |
|------------------------|-------------------------------------:|------------:|--------:|------------------------:|
| `train.sentences.tsv ` | Training Sentences                   | 1.542.499 | 376MB | `s_id, d_id, sentence`  |    
| `valid.sentences.tsv ` | Validation Sentences                 |    85.375 | 21MB  | `s_id, d_id, sentence`  |   
| `test.sentences.tsv `  | Test Sentences                       |    85.405 | 21MB  | `s_id, d_id, sentence`  |  
| `train.pairs.tsv    `  | Positive Sentence Pairs (Training)   | 1.404.271 | 20MB  | `s_id, s_id'`           |     
| `valid.pairs.tsv  `    | Positive Sentence Pairs (Validation) |    78.472 | 1.1MB | `s_id, s_id'`           |     
| `test.pairs.tsv  `     | Positive Sentence Pairs (Test)       |    76.626 | 20MB  | `s_id, s_id'`           |   
| `refs.tsv    `         | All References                       |   790.508 | 20MB  | `r_id, TYPE, reference` | 
| `sent_ref_map.tsv `    | References used by Sentence X        | 1.713.279 | 25MB  | `s_id, r_id0 r_id1 ...` | 
| `es_neighborts.tsv`    | Elasticsearch Neighbors              |   466.154 | 25MB  | `s_id(query), s_id0 s_id1 ...` | 
   

## 🤗 Hugging Face Dataset ##

*Hugging Face Datasets* makes accessing scientific Datasets very fast and easy. We have uploaded 
German Legal Sentences to the Hugging Face Datasets Hub using the identifier [`lavis-nlp/german_legal_sentences`](https://huggingface.co/datasets/lavis-nlp/german_legal_sentences). 

### Usage ##### 
```
>>> import datasets
>>> ds = datasets.load_dataset("lavis-nlp/german_legal_sentences", "pairs")
Downloading: 100%|████████████████████████████████████████████████████████
...
>>> ds
DatasetDict({
    train: Dataset({
        features: ['query.sent_id', 'query.doc_id', 'query.text', 'query.ref_ids', 'related.sent_id', 'related.doc_id', 'related.text', 'related.ref_ids'],
        num_rows: 1404271
    })
    validation: Dataset({
        features: ['query.sent_id', 'query.doc_id', 'query.text', 'query.ref_ids', 'related.sent_id', 'related.doc_id', 'related.text', 'related.ref_ids'],
        num_rows: 78472
    })
    test: Dataset({
        features: ['query.sent_id', 'query.doc_id', 'query.text', 'query.ref_ids', 'related.sent_id', 'related.doc_id', 'related.text', 'related.ref_ids'],
        num_rows: 76626
    })
})

```
 There are three options to access the data using t:

### Example ###
```
{'query.doc_id': 28860,
 'query.ref_ids': [6215, 248, 248],
 'query.sent_id': 304863,
 'query.text': 'Zudem ist zu berücksichtigen , dass die Vollverzinsung nach '
               '[REF] i. V. m. [REF] gleichermaßen zugunsten wie zulasten des '
               'Steuerpflichtigen wirkt , sodass bei einer Überzahlung durch '
               'den Steuerpflichtigen der Staat dem Steuerpflichtigen neben '
               'der Erstattung ebenfalls den entstandenen potentiellen Zins- '
               'und Liquiditätsnachteil in der pauschalierten Höhe des [REF] '
               'zu ersetzen hat , unabhängig davon , in welcher Höhe dem '
               'Berechtigten tatsächlich Zinsen entgangen sind .',
 'related.doc_id': 56348,
 'related.ref_ids': [248, 6215, 62375],
 'related.sent_id': 558646,
 'related.text': 'Ferner ist zu berücksichtigen , dass der Zinssatz des [REF] '
                 'im Rahmen des [REF] sowohl für Steuernachforderung wie auch '
                 'für Steuererstattungen und damit gleichermaßen zugunsten wie '
                 'zulasten des Steuerpflichtigen wirkt , Vgl. BVerfG , '
                 'Nichtannahmebeschluss vom [DATE] [REF] , juris , mit der '
                 'Folge , dass auch Erstattungsansprüche unabhängig davon , ob '
                 'und in welcher Höhe dem Berechtigten tatsächlich Zinsen '
                 'entgangen sind , mit monatlich 0,0 % verzinst werden .'}
```


## Generation 

The following describes the process of transforming the Open Legal Data corpus into a dataset of semantically related sentences pairs.

### Pre-Processing and Citation Parsing

The documents we take from Open Legal Data are first preprocessed by removing line breaks, enumeration characters and headings. Afterwards we parse legal citations using hand-crafted regular expressions. Each citation is split into it components and normalized, thus different variants of the same citation are matched together. For instance, "§211 Absatz 1 des Strafgesetzbuches" is normalized to "§ 211 Abs. 1 StGB". Every time we discover an unknown citation, we assign an unique id to it. We use these ids to replace parsed citations in the document text with a simple reference tag containing this id (e.g `[REF321]`). At the same time we parse dates and replace them with the date tag `[DATE]`. Both remove dots which can may be confused with the end of a sentence, which makes the next stage easier.

### Sentence Tokenizing

We use [SoMaJo](https://github.com/tsproisl/SoMaJo) to perform sentence tokenizing on the pre-processed documents. Each sentence that does not contain at least one legal citation is discarded. For the rest we assign sentence ids, remove all reference ids from them as well as any contents in braces (braces often contain large enumerations of citations and their sources). At the same time we keep track of the corresponding document from which a sentence originates and which references occur in it. 


## Build Positive Sentence Pair

Based on the parsed references and their occurrences in the tokenized sentences, we create an inverted index. This allows us to quickly retrieve all sentences that share a reference with a target sentence. We assume that a substantial portion of these sentences are semantically related to the target sentence due to fact that they share at least one reference. This basically is our first heuristic. Our second heuristic exploits the documents and their references, from which the sentences originate. We assume that a pair of sentences whose source documents share many references, have an above-average probability of being semantically related. Hence, we measure the *Jaccard-Similarity*
 
 
 documents from which a pair of sentences orginate share many references, this sentence pair have an increased probability of being semantically related 



### Training, Validation and Test Split

The dataset has been split along documents (from which the sentences originate) into a 90/5/5 split for training, validation and testing. We ensure that any query sentence in the validation or test split is not contained by the training set. The sentences to match with, however, are the same for all three sets.   



GLS consists of semantically related sentence pairs from German judicial decisions. Their semantic relations are derived from shared references to laws other judicial decisions. The basis of our dataset is given by [Open Legal Data](http://openlegaldata.io/), which is a collection of more than 200k German legal documents.
 
 
 
 First, we parse references to laws and other judicial decisions  split into sentences using the [SoMaJo Sentence Tokenizer](https://github.com/tsproisl/SoMaJo), which satisfactory solved the challenge posed by the numerous appearances of legal abbreviations.






## Ranking Task / Evaluation

Based on the generated sentences pairs we define a ranking task for the evaluation of text similarity metrics. For this, the first sentence of a pair is used as a query while all sentences that are paired with this sentences are  

### Metrics

### Baselines

| Method                                           | MRR@10   | MAP@200    | Recall@200  |
|--------------------------------------------------|---------:|-----------:|------------:|
| BM25 - default `(k1=1.2; b=0.75)`                |     25.7 |       17.6 |        42.9 |
| BM25 - tuned `(k1=0.47; b=0.97)`                 |     26.2 |       18.1 |        43.3 |
| [CoRT](https://arxiv.org/abs/2010.10252)         |     31.2 |       21.4 |        56.2 |
| [CoRT + BM25](https://arxiv.org/abs/2010.10252)  |     32.1 |       22.1 |        67.1 |



