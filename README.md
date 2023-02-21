# Read Me
In this app, we tried saving data related to a `Person` containing 
name as both `analyzed` and `not_analyzed` fields.

```
name         : analyzed
name.verbal  : not_analyzed
```
This means that if **Mithun** is the value for `name` field,
it will be broken into tokens in `name` and kep as is in `name.verbal`
field.
```
name         :   Mithun, Mithun
name.verbal  : Mithun
```
This also means that I can search for individual tokens in `name` but
I can only search for complete `Shubham Aggarwal` in the `name.verbal`
field.

## Running Match queries
To search through the name, we made a `match` query as:
```
fun searchPersonAnalysed(text: String): List<Person> {
  val query = NativeSearchQueryBuilder()
                .withQuery(QueryBuilders.matchQuery("name", text))
                .build()
  return esTemplate.queryForList(query, Person::class.java)
}

fun searchPersonNonAnalysed(text: String): List<Person> {
  val query = NativeSearchQueryBuilder()
                .withQuery(QueryBuilders.matchQuery("name.verbal", text))
                .build()
  return esTemplate.queryForList(query, Person::class.java)
}
```

* URL: localhost:8102/add?name=mithun

  Response:
  ```
  {
      "id": "AWBUy6mplvsGm5f68LZn",
      "name": "Mithun"
  }
  ```
  We just added a new Person in DB.
  
* URL: localhost:8102/add?name=Mithun 123

  Response:
  ```
  {
      "id": "AWBU-15U8-Pb-kpCBLtN",
      "name": "Mithun"
  }
  ```
  We added another new Person in DB with numbers.
  
* URL: localhost:8102/search?freeText=**Mithun**&analysed=**true**

  Response:
  ```
  [
    {
        "id": "AWBUy6mplvsGm5f68LZn",
        "name": "Mithun"
    }
  ]
  ```
  Data **found** when `analyzed` fields were queried with 
  **smaller** letter.
  
* URL: localhost:8102/search?freeText=**mithun**&analysed=**false**

  Response:
  ```
  []
  ```
  So, no data found when `not_analyzed` fields were queried with 
  **smaller** letter.
  
* URL: localhost:8102/search?freeText=**mithun**&analysed=**false**

  Response:
  ```
  [
    {
        "id": "AWBUy6mplvsGm5f68LZn",
        "name": "mithun"
    }
  ]
  ```
  Data **found** when `not_analyzed` fields were queried with 
  **Capital** letter, **exactly** as it was entered into the DB.
  
* URL: localhost:8102/search?freeText=**Shub**&analysed=**false**

  Response:
  ```
  []
  ```
  Data **not found** when `not_analyzed` fields were queried with 
  **incomplete String**.
  
* URL: localhost:8102/search?freeText=**shub**&analysed=**true**

  Response:
  ```
  []
  ```
  Data **not found** when `analyzed` fields were queried with 
  **incomplete String**. This is because match queries work with 
  complete Strings as well.

## Running Term queries
This time, we use `term` queries as:
```
fun searchPersonAnalysed(text: String): List<Person> {
  val query = NativeSearchQueryBuilder()
                .withQuery(QueryBuilders.termQuery("name", text))
                .build()
  return esTemplate.queryForList(query, Person::class.java)
}

fun searchPersonNonAnalysed(text: String): List<Person> {
  val query = NativeSearchQueryBuilder()
                .withQuery(QueryBuilders.termQuery("name.verbal", text))
                .build()
  return esTemplate.queryForList(query, Person::class.java)
}
```
* URL: localhost:8102/search?freeText=**mithun**&analysed=**true**

  Response:
  ```
  [
    {
        "id": "AWBUy6mplvsGm5f68LZn",
        "name": "mithun"
    }
  ]
  ```
  No change was observed from `match` query results.
  
* URL: localhost:8102/search?freeText=**mithun**&analysed=**false**

  Response:
  ```
  []
  ```
  No change was observed from `match` query results.
  
* URL: localhost:8102/search?freeText=**Mithun**&analysed=**false**

  Response:
  ```
  [
    {
        "id": "AWBUy6mplvsGm5f68LZn",
        "name": "Mithun"
    }
  ]
  ```
  No change was observed from `match` query results.
  
* URL: localhost:8102/search?freeText=**Mithun**&analysed=**false**

  Response:
  ```
  []
  ```
  No change was observed from `match` query results.
  
* URL: localhost:8102/search?freeText=**Mithun**&analysed=**true**

  Response:
  ```
  []
  ```
  No change was observed from `match` query results.
  
## Running Wildcard queries
This time, we use `wildcardQuery` queries. As `wildcardQuery` only
works with `not_analyzed` fields, we will only check for the same.
```
fun searchPersonNonAnalysed(text: String): List<Person> {
  val query = NativeSearchQueryBuilder()
                .withQuery(QueryBuilders.wildcardQuery("name.verbal", "*$text*"))
                .build()
  return esTemplate.queryForList(query, Person::class.java)
}
```
* URL: localhost:8102/search?freeText=**mithun**&analysed=**false**
  
  Response:
  ```
  [
    {
        "id": "AWBUy6mplvsGm5f68LZn",
        "name": "mithun"
    }
  ]
  ```
  Query worked even with **incomplete String** which did not 
  happened in case of `match` or `term` query.
  
* URL: localhost:8102/search?freeText=**Shub**&analysed=**false**

  Response:
  ```
  [
    {
        "id": "AWBUy6mplvsGm5f68LZn",
        "name": "mithun"
    }
  ]
  ```
  So, even when a **capital incomplete** String was passed, we got the 
  results. But note that **S** was enetered as capital even in 
  original data.
  
* URL: localhost:8102/search?freeText=**shub**&analysed=**false**

  Response:
  ```
  []
  ```
  So, **it did not work**! It needs an exact match and it cares
  for the case of alphabets.
  
* Let's try a search with numbers. URL: localhost:8102/search?freeText=**12**&analysed=**false**

  Response:
  ```
  [
      {
          "id": "AWBU-15U8-Pb-kpCBLtN",
          "name": "mithun 123"
      }
  ]
  ```
  So, **it worked** when only numbers were passed! Try mixing input.
  
* URL: localhost:8102/search?freeText=**M 12**&analysed=**false**

  Response:
  ```
  []
  ```
  So, **it didn't work** when M was capital! it works when its same 
  as in DB, a small `m`.
  
## Running native queries
Now, we will use native queries.

Query looks like:
    
    @Query("{ \"query\": { \"query_string\" : { \"fields\" : [\"name\", \"name.verbal\"], \"query\":\"*?0*\"}}}")
    fun freeTextSearchPerson(text: String, pageable: Pageable): List<Person>

Now, if we run queries for search as:

    fun searchNativelyPerson(text: String): List<Person> {
        return personRepository.freeTextSearchPerson(
                text, PageRequest(0, 100,
                Sort(Sort.Order(Sort.Direction.ASC, "name"))))
    }
    
* URL: localhost:8102/native/search?freeText=**h**

  Response:
  ```
    [
        {
            "id": "AWBU-15U8-Pb-kpCBLtN",
            "name": "Mithun 123"
        },
        {
            "id": "AWBVGrdlQ-Xz405fI9hI",
            "name": "mithu .2"
        },
        {
            "id": "AWBVGnCgQ-Xz405fI9hH",
            "name": "mithu .2"
        },
        {
            "id": "AWBUy6mplvsGm5f68LZn",
            "name": "Mithun"
        },
        {
            "id": "AWBVG1EPQ-Xz405fI9hK",
            "name": "mithu"
        },
        {
            "id": "AWBVGscTQ-Xz405fI9hJ",
            "name": "mithu"
        }
    ]
  ```
  Before running, we added some random objects. Few things to notice 
  are:
  * This works exactly as we need in terms of search.
  * Sorting is correct if we consider the terms without any number 
  and special characters.
  
With slight change in query as:

    @Query("{ \"query\": { \"query_string\" : { \"fields\" : [\"name\"], \"query\":\"*?0*\"}}}")
    fun freeTextSearchPerson(text: String, pageable: Pageable): List<Person>
    
We can hit: localhost:8102/native/search?freeText=.2 and z

  Response will be:
  ```
  [
      {
          "id": "AWBVGrdlQ-Xz405fI9hI",
          "name": "mithu .2"
      },
      {
          "id": "AWBVGscTQ-Xz405fI9hJ",
          "name": "mithu"
      },
      {
          "id": "AWBVGg2VQ-Xz405fI9hF",
          "name": "mith"
      }
  ]
  ```
  
  So, with `and`, we get values with either tokens and values with non-special
  characters are sorted.