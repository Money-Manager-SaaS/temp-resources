# MM Cloud API 



The API for BFF (Back-end for front-end) and Importer Service (internal API, used by BFF or other internal services only)

## Introduction

This is a document for MM Cloud Architecture and API.

### BFF and Architecture

(Back-end for Front-end), a common name in micro service, in a common Micro Service architecture, only the BFF is used in Front-end application, all other internal service is hidden from internet, and be used in BFF or other services.



The two diagrams below are the comparison between **general purpose api** and **BFF**.



![https://samnewman.io/pattern-img/bff/single-api.jpg](https://samnewman.io/pattern-img/bff/single-api.jpg)



![img](https://samnewman.io/pattern-img/bff/bff-overview.jpg)

Currently, we probably have only on front-end client, there are only one BFF for now.

### SQL Importer

As in the diagram above, **Importer** is one downstream service in the architecture,  BFF or other services can directly talk to **Importer**.

It can transfer a SQLite3 db to a PostgreSQL db.





## API

- `?` stands for optional
- `*` must have

### BFF

The current back-end entry point for out front-end app.

This document only cover the core features for the MVP.

**Requirements**

- Provide `CRUD` for front-end app
- Requires to be asynchronous
- Authentication (after MVP, after all functional features work)

**Endpoints**

| url                | METHOD | Descrtiption                   |
| ------------------ | ------ | ------------------------------ |
| /transactions      | get    | get all transactions           |
|                    | post   | create a new transaction       |
| /transactions/<id> | get    | get one                        |
|                    | put    | replace an existed transaction |
|                    | patch  | update one                     |
|                    | delete | delete one                     |
| /accounts          |        | similar to the above           |
| /categories        |        |                                |
| /subcategories     |        |                                |



**Example**

1. create a transaction

Use transactions as an example, similar to other models.

*Request*

The 3 parameters below, either one exist the API can work, 

```python
    {
      "TransCode": -1, // -1 withdraw, 1 deposit, 0 transfer
      "AccountID": 1,
      "ToAccountID": null, // only have value when transfer
      "SubCategoryID": 1,
      "CategoryID": 1,
      "PayeeID": 1,
      "Notes": "shopping",
      "Amount": 100,
      "ToAmount": 0, // be 0, if it is not transfer
    }


```

*Response*

```python
{
	"id": 123
}
```



**Models Examples**

```json
{
    "transactions": [
        {
            "id": 1,
            "TransCode": -1,
            "AccountID": 1,
            "ToAccountID": null,
            "SubCategoryID": 1,
            "CategoryID": 1,
            "PayeeID": 1,
            "Notes": "shopping",
            "Amount": 100,
            "ToAmount": 0
        },
        {
            "id": 2,
            "TransCode": 0,
            "AccountID": 1,
            "ToAccountID": 2,
            "SubCategoryID": -1,
            "CategoryID": -1,
            "PayeeID": -1,
            "Notes": "transfer to bank",
            "Amount": 100,
            "ToAmount": 100
        }
    ],
    "subCategories": [
        {
            "id": 1,
            "Name": "Drink",
            "Category": {}
        }
    ],
    "accounts": [
        {
            "id": 1,
            "Name": "ANZ",
            "Type": "Cash",
            "CurrencyID": 1
        },
        {
            "id": 2,
            "Name": "BNZ",
            "Type": "Cash",
            "CurrencyID": 1
        }
    ],
    "payees": [
        {
            "id": 1,
            "Name": "Kiwi House",
            "Category": {},
            "SubCategory": {}
        }
    ],
    "currencies": [
        {
            "id": 1,
            "Name": "United States dollar",
            "PFX_Symbol": "$",
            "Symbol": "USD",
            "Scale": 100,
            "BaseConvRate": 1
        }
    ]
}
```

We focus on transactions, categories, subcategories and accounts for the current stage.



### Importer

- Language: Python? JS? SQL (mainly sql)

**Requirements**

- Transfer a SQLite3 Db to a Postgres DB efficiently
- Better to be asynchronous(should be not a concern for the current stage)
- Keep the tables and attributes the same as the original schema (google MMEX db schema)

**Endpoints**

There should be at least one endpoint

| url     | Method | descrition                          |
| ------- | ------ | ----------------------------------- |
| /import | post   | to import the SQLite3 DB to Psql DB |

**Example**

*Request*

The 3 parameters below, either one exist the API can work, 

```python
{
	db_path: [?a local sqlite3 db path],
	db_binary: [?sqlite3 data as a binary file uploaded as requestt body],
   s3_key: [?an AWS s3 resource key]
}
```

*Response*

```python
{
	db_name: [*a unique name for the db]
}
```



## References

- [fake api](https://github.com/Money-Manager-SaaS/money-manager-ss-fake-db/blob/master/db.json)
- [BFF](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=11&cad=rja&uact=8&ved=2ahUKEwig15_hpInoAhVBwzgGHe2cCBoQFjAKegQIBRAB&url=https%3A%2F%2Fsamnewman.io%2Fpatterns%2Farchitectural%2Fbff%2F&usg=AOvVaw0pgT2M2_yFQm-V0jVKItww)
- [mmex db scehma](https://github.com/moneymanagerex/database)
- [simple arch](![arch.jpg](https://github.com/Money-Manager-SaaS/money-manager-ss-fake-db/blob/master/arch.jpg?raw=true))