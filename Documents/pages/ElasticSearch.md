# Commands
	- To check status: `curl "http://localhost:9200/?pretty"`
	- Print all docs: `GET _search { "query": { "match_all": {} } } `
	- Elasticsearch and Lucene use a structure called an inverted index (similar to B-tree in relational databases).
		- * Query String Search
		- * `GET /olive/vendors/_search`
		- * `GET /olive/vendors/_count`
		- * `GET /olive/vendors/_search?q=city:charlotte`
		- **Query DSL**
		  collapsed:: true
			- ```http
			  GET /olive/vendors/_search
			  {
			   "query": {
			     "match": {
			       "city": "charlotte"
			     }
			   }
			  }
			  ```
			- ```http
			  GET /olive/vendors/_search
			  {
			    "aggs": {
			      "city_vendors": {
			        "terms": {
			          "field": "contact.address.city"
			        }
			      }
			    }
			  }
			  ```