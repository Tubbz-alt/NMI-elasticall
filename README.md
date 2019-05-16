
# elasticall
Shell tool to monitor Elasticsearch

**Motivation**. I want to be able to run take a random sample from Elasticsearch. I want to do it from the shell and I want the thing to be practical as possible. I would like the computer to manage a question like this "Hey Jarvis, show me a size 100 random sample of all log lines made in the last 2 days". 

The script **elasticall** was made to throw very complex queries to Elasticsearch and beutify the output results. As such, it is a tool that frees the user from the tedious interactions with JSON. Taking random sample is just anohter (complex) Elasticsearch query.

#### Heads up ! 

<span style="color:darkred;font-weight:bold">CAVEAT 1.</span> Before you do any test you must be aware of this important fact. `elasticall` runs in *psmetric01* but the Elasticsearch server is in **psmetric04**. If you run a search that takes time and kill elasticall with `Ctrl+C,` the search will keep on running on psmetric04  Keep an eye on psmetric04 with `top`.

#### Example 1. The Random Sample

The random sample can be interesting per se, in the sense that it can be usefull to spot anomalies, but the first reason I needed it was to measure Elasticsearch performance. 

Measure how much it takes to parse alla Elasticsearch contentent.

    $> time elasticall queries/paramRandomSample2


#### Example 2. Counting documents (log lines)
