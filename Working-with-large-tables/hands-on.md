## Hands -On

### Preparation

Seeting up HIVE session:

TODO: INSERT IMAGE

### 1 Compute Term Vectors
From texts to term vectors

- Input: gutenberg_docs.tsv 
- Output: gutenberg_terms.tsv 
- Task: Explore data

(Already done, as complicated and time consuming. Output file is in the Group_0 folder.)

This step involves a set of NLP techniques:

- Remove common words (the, of, for, …)
- Part of Speech tagging (Verb, Noun, …)
- Stemming (going -> go)
- Abstract (12, 1.000, 20% -> <NUMBER>)


Create the HIVE table mapped to the files ```gutenberg\_docs.tsv```
```
-- 1.1 TABLE GUTENBERG_DOCS
CREATE EXTERNAL TABLE citylabs_workshop.gutenberg_docs_X
    (ID STRING, SOURCE STRING, LEN INT, CHARSET STRING, CONTENT STRING) 
ROW FORMAT DELIMITED 
FIELDS TERMINATED BY '\t' 
LINES TERMINATED BY '\n' 
STORED AS TEXTFILE 
LOCATION '/project/citylabs-workshop/group_0/gutenberg_docs.tsv'; 
```

```
-- 1.2 COUNT GUTENBERG_DOCS
SELECT COUNT(ID) FROM citylabs_workshop.gutenberg_docs_X;
```

```
-- 1.2 AVG(LEN) GUTENBERG_DOCS_
SELECT AVG(LEN) FROM citylabs_workshop.gutenberg_docs_X;
```
***Remember to replace X with your group number!!!***
```
CREATE EXTERNAL TABLE citylabs.gutenberg_terms 
    (DOC STRING, POSITION INT, TERM STRING) 
ROW FORMAT DELIMITED    
FIELDS TERMINATED BY  '\t' 
LINES TERMINATED BY '\n' 
STORED AS TEXTFILE 
LOCATION '/project/citylabs-workshop/group_X/gutenberg_terms.tsv'; 
```

```
SELECT COUNT(TERM) FROM citylabs.gutenberg_terms;
```

```
SELECT DOC, COUNT(TERM) AS LENGTH FROM citylabs.gutenberg_terms 
GROUP BY DOC ORDER BY LENGTH DESC;
```

Lookup book id Gutenberg-11800 as follows:
http://www.gutenberg.org/ebooks/11800

### 2 Compute Terms Frequency (TF) 

a) From term word vectors to word counts 

b) Generate doc size from word counts

Task: run PIG script to produce the output file:

***Remember to replace X with your group number!!!***
```
set mapreduce.map.memory.mb    2048 
set mapreduce.reduce.memory.mb 5120 
source = LOAD '/project/citylabs-workshop/group_X/gutenberg_terms.tsv' AS (doc_id:chararray, position:int, word:chararray); 
collection = FILTER source BY (SUBSTRING(word, 0, 1) != '<');
doc_words       = GROUP collection BY (doc_id, word);
doc_word_counts = FOREACH doc_words 
                                 GENERATE 
                                    FLATTEN(group) AS (doc_id, word), 
                                    COUNT(collection) AS num_doc_wrd_usages
                    ;

usage_bag    = GROUP doc_word_counts BY doc_id;
usage_bag_ds = FOREACH usage_bag 
      GENERATE 
group AS doc_id,
    FLATTEN 
(doc_word_counts.(word, num_doc_wrd_usages)) 
AS (word,num_doc_wrd_usages),
    SUM(doc_word_counts.num_doc_wrd_usages) 
AS doc_size
                   ;

term_freqs =  
       FOREACH usage_bag_ds 
              GENERATE 
                        doc_id AS doc_id, 
                        word AS word, 
               ((double) num_doc_wrd_usages / (double) doc_size) AS term_freq; 
             ; 

STORE term_freqs 
INTO '/project/citylabs-workshop/group_X/term_freqs.tsv';
```
Execution should take approximately 4 minutes.


### 3 Compute Inverse Document Frequencies (IDF)
Compute inverse document frequences:

Input: term_freqs.tsv

Output: term_usages_idf.tsv

Task: run PIG script to produce the output file

***Remember to replace X with your group number!!!***
```
set mapreduce.map.memory.mb    2048 
set mapreduce.reduce.memory.mb 5120 
term_freqs = LOAD '/project/citylabs-workshop/group_X/term_freqs.tsv’  
      USING PigStorage('\t’)  
      AS (doc_id:chararray, term:chararray, term_freq:double); 
               
term_usage_bag  = GROUP term_freqs BY term; 
term_usages    = FOREACH term_usage_bag 
                            GENERATE 
                                FLATTEN(term_freqs) AS (doc_id, term, term_freq), 
                                COUNT(term_freqs)   AS num_docs_with_term
                   ; 
term_usages_idf = FOREACH term_usages { 
              idf    = LOG((double) 48790 / (double) num_docs_with_term);  
                GENERATE 
                  doc_id AS doc_id, 
                  term  AS term, 
                  term_freq as term_freq,
                  idf AS idf 
                ; 
             }; 
STORE term_usages_idf INTO '/project/citylabs-workshop/group_X/term_usages_idf.tsv'; 
```
Execution should take approximately 4 minutes.

### 4 Compute TF/IDF
Generate IDF and compute TF/IDF

- Input: term_usages_idf.tsv
- Output: tfidf.tsv
- Task: run PIG script to produce the output file.

***Remember to replace X with your group number!!!***
```
set mapreduce.map.memory.mb    2048 
set mapreduce.reduce.memory.mb 5120 

word_usages_idf = LOAD '/project/citylabs-workshop/group_X/word_usages_idf.tsv' 
USING PigStorage('\t') AS (doc_id:chararray, word:chararray, word_freq:double, idf:double); 

tfidf = FOREACH word_usages_idf { 
              tf_idf = (double) word_freq*idf; 
                GENERATE 
                  doc_id AS doc_id, 
                  word  AS word, 
                  tf_idf AS tf_idf 
                ; 
             }; 

STORE tfidf INTO '/project/citylabs-workshop/group_X/tfidf.tsv'; 
```



