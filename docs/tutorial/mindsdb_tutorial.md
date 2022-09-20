# Predicting Home Rental Prices with MindsDB in Python

In this tutorial, we'll will learn to create, train, and query a machine learning model (`predictor`) with Mindsdb in Python. You can check this [tutorial]() for a more generic approach to implementing a machine learning model with MindsDB.

First, we need to set up a development environment. You will need access to a working MindsDB installation, which can be done locally or via [MindsDB Cloud](https://cloud.mindsdb.com/?_ga=2.121740043.1875715573.1663498526-358045687.1658244666&_gl=1*1hemhn1*_ga*MzU4MDQ1Njg3LjE2NTgyNDQ2NjY.*_ga_7LGFPGV6XV*MTY2MzU3MTAwOS4yOS4xLjE2NjM1NzI4OTEuMC4wLjA).

You can set up your MindsDB cloud account following this [guide](https://docs.mindsdb.com/setup/cloud/?_ga=2.21191739.1875715573.1663498526-358045687.1658244666&_gl=1*19ko74n*_ga*MzU4MDQ1Njg3LjE2NTgyNDQ2NjY.*_ga_7LGFPGV6XV*MTY2MzU3MTAwOS4yOS4xLjE2NjM1NzI4OTEuMC4wLjA). Alternatively, you can set up your MindsDB locally using [Docker](https://docs.mindsdb.com/setup/self-hosted/docker/?_ga=2.47799207.1875715573.1663498526-358045687.1658244666&_gl=1*28ggvt*_ga*MzU4MDQ1Njg3LjE2NTgyNDQ2NjY.*_ga_7LGFPGV6XV*MTY2MzU3MTAwOS4yOS4xLjE2NjM1NzI4OTEuMC4wLjA) or [Python](https://docs.mindsdb.com/setup/self-hosted/pip/source/?_ga=2.47799207.1875715573.1663498526-358045687.1658244666&_gl=1*28ggvt*_ga*MzU4MDQ1Njg3LjE2NTgyNDQ2NjY.*_ga_7LGFPGV6XV*MTY2MzU3MTAwOS4yOS4xLjE2NjM1NzI4OTEuMC4wLjA). For simplicity, we would be using MindsDB via the MindsDB cloud in this tutorial.

For this tutorial, I will coding using VS-code. You can use other IDEs of your choice or a python environment like Jupyter Notebook. As with best practices, make sure to code within a virtual environment ( for IDEs). To save yourself the haggle of setting up a virtual environment, you can just use Jupyter Notebook.

## Installations

The following are the requirements for this tutorial:

- Python
- Sqlachemy
- MindsDB (you can install using pip)
- pymysql

With that done, let's get started.

## The Data

### Creating a Database

You can peruse a local data on your local machine for this. In this case, you will need to upload it to MindsDB via [MindsDB SQL Editor](https://docs.mindsdb.com/connect/mindsdb_editor/?_ga=2.79795991.1875715573.1663498526-358045687.1658244666&_gl=1*51tqjx*_ga*MzU4MDQ1Njg3LjE2NTgyNDQ2NjY.*_ga_7LGFPGV6XV*MTY2MzU5MzA4NC4zMy4xLjE2NjM1OTM1MDMuMC4wLjA).

Follow this [guide](https://docs.mindsdb.com/sql/create/file/?_ga=2.88224923.1875715573.1663498526-358045687.1658244666&_gl=1*1razhsj*_ga*MzU4MDQ1Njg3LjE2NTgyNDQ2NjY.*_ga_7LGFPGV6XV*MTY2MzU5NTUzOC4zNC4xLjE2NjM1OTU1NTcuMC4wLjA) to upload the file to MindsDB.

Alternatively, you can utilize a demo database we've already prepared for you.

To create a data connection using the demo database, you will need to create a database from with your MindsDB cloud account.

<!-- === "Connecting as a database via `#!sql CREATE DATABASE`" -->

```
<!-- Connecting as a database via sql CREATE DATABASE -->

CREATE DATABASE example_db
    WITH ENGINE = "postgres",
    PARAMETERS = {
        "user": "demo_user",
        "password": "demo_password",
        "host": "3.220.66.106",
        "port": "5432",
        "database": "demo"
};
```

Or if you want, you can download the demo database as a [.CSV file](https://mindsdb-test-file-dataset.s3.amazonaws.com/home_rentals.csv), then upload it via MindsDB SQL editor.

Follow this [guide](https://docs.mindsdb.com/sql/create/file/?_ga=2.88224923.1875715573.1663498526-358045687.1658244666&_gl=1*1razhsj*_ga*MzU4MDQ1Njg3LjE2NTgyNDQ2NjY.*_ga_7LGFPGV6XV*MTY2MzU5NTUzOC4zNC4xLjE2NjM1OTU1NTcuMC4wLjA) to upload the file to MindsDB.

Once uploaded, you should be able to query the file directly.

```
SELECT *
FROM files.home_rentals
LIMIT 10;
```
For the sake of ease, we would be creating our database from within a MindsDB Cloud Account. You can go ahead and do just that.

## Connecting to the Database

Now, navigate into a working directory where you have you virtual environment setup, then create a `main.py` file. To create a database connection, simply follow [this guide](https://docs.mindsdb.com/connect/sql-alchemy/) on how to do that using Sqlachemy and pymysql.

When you are done, you should have something like this:

```
<!-- main.py -->

import mindsdb
import sqlalchemy
import pymysql
from sqlalchemy import create_engine

user = 'MindsDB Cloud Username' #same as your Mindsdb Cloud email address is your username
password = 'MindsDB Cloud Password' # replace this value
host = 'cloud.mindsdb.com'
port = 3306
database = 'example_db'

def get_connection():
        return create_engine(
                url="mysql+pymysql://{0}:{1}@{2}:{3}/{4}".format(user, password, host, port, database)
        )

if __name__ == '__main__':
        try:
                engine = get_connection()
                print(f"Connection to the {host} for user {user} created successfully.")
        except Exception as ex:
                print("Connection could not be made due to the following error: \n", ex)
```

You  can also check that your connection is successful by running the python file from your terminal using `python main.py`.

```
Connection to the cloud.mindsdb.com for user demo_user created successfully.
```

The Sqlalchemy `create-engine` is lazy. To run further check on your connection, you might want to run queries on the database to see if it return some data. 

```
<!-- main.py -->

with engine.connect() as eng:
    query = eng.execute("SELECT * FROM example_db.demo_data.home_rentals LIMIT 2;")

    for row in query:
        print(row)
```

On execution, you should get this printed in your terminal:

```
(2, 1, 917, 'great', 13, 'berkeley_hills', 3901)
(0, 1, 194, 'great', 10, 'berkeley_hills', 2042)
```
!!! warning "Getting an Sqlalchemy Operational Error"

    While executing your code, you might get an operational error from Sqlalchemy:
    `Lost connection to MySQL server during query ([WinError 10054] An existing connection was forcibly closed by the remote host)`.

    This error is triggered due to an existing connection. In this case, you must close the connection before executing your next line of code. Simply call Sqlalchemy `close` method on the engine. For instance, `eng.close()`

## Training a Predictor With `CREATE PREDICTOR`

With that done, we can now train our first machine learning predictor. For that, we are going to use the `CREATE PREDICTOR` syntax where we would specify what sub-query to train `FROM` and what we want to  `PREDICT`.

```
CREATE PREDICTOR mindsdb.rental_pred
FROM example_db
  (SELECT * FROM demo_data.home_rentals)
PREDICT rental_price;
```

!!! warning " "
    `mindsdb` in `mindsdb.rental_pred` does not reference a database. It is MindsDB standard way of creating a predictor. Alternatively, you could simply write `rental_pred`. However, it is advisable to use `mindsdb.rental_pred` to quicken response time. Slow response time could trigger Sqlachemy operational error.

Implementing this in our code:

```
<!-- main.py -->

....

predictor = eng.execute("CREATE PREDICTOR mindsdb.rental_pred FROM example_db (SELECT * FROM demo_data.home_rentals) PREDICT rental_price;")
```

## Checking the Status of a Predictor

It might take a couple of minutes to train a predictor. You can monitor the status of your predictor using this line of code:

```
....
result = eng.execute("SELECT status FROM mindsdb.predictors WHERE name='rental_pred';")
        for i in result:
                print(i)

```

If the training is complete, you should get this printed in your terminal:

```
('complete',)
```

Once the status reads complete, we can start making predictions!

## Making Predictions

### Make Predictions with `SELECT`

Now, we can make predictions by querying the predictor. The `SELECT` syntax allows you make predictions for the label based on the chosen features.

```
....
predict  = eng.execute("SELECT rental_price, rental_price_explain FROM mindsdb.home_rentals_model WHERE number_of_bathrooms=2 AND sqft=1000;")

        for i in predict:
           print(i)
```
On execution, you should get this printed in your terminal:

```
('4770', '{"predicted_value": 4770, "confidence": 0.99, "anomaly": null, "truth": null, "confidence_lower_bound": 4689, "confidence_upper_bound": 4852}') 

<!-- rental_price = "4770", rental_price_explain= '{"predicted_value":4770, ...}... -->
```

### Make Batch Predictions with `JOIN`

You might also want to make bulk predictions on several records in your table. In that case, you can achieve that by joining the table containing the data (`example_db.demo_data.home_rentals`) to your predictor.

```
<!-- main.py -->

from sqlalchemy import text

....

bulk = text("SELECT t.rental_price AS real_price, m.rental_price_explain AS predicted_price, t.number_of_rooms,  t.number_of_bathrooms, t.sqft, t.location, t.days_on_market FROM example_db.demo_data.home_rentals AS t JOIN mindsdb.home_rentals_model AS m LIMIT 100;")

batch = eng.execute(bulk)

    for i in batch:
        print(i)
```

On execution, you should get this printed in your terminal:

```
(3901, 2, 1, 917, 'great', 13, '{"predicted_value": 3886, "confidence": 0.99, "anomaly": null, "truth": null, "confidence_lower_bound": 3805, "confidence_upper_bound": 3967}')
(2042, 0, 1, 194, 'great', 10, '{"predicted_value": 2007, "confidence": 0.99, "anomaly": null, "truth": null, "confidence_lower_bound": 1925, "confidence_upper_bound": 2088}')
(1871, 1, 1, 543, 'poor', 18, '{"predicted_value": 1865, "confidence": 0.99, "anomaly": null, "truth": null, "confidence_lower_bound": 1783, "confidence_upper_bound": 1946}')
(3026, 2, 1, 503, 'good', 10, '{"predicted_value": 3020, "confidence": 0.99, "anomaly": null, "truth": null, "confidence_lower_bound": 2938, "confidence_upper_bound": 3101}')
....
...
```

## What's Next?

Have fun while trying it out yourself!

- Bookmark [MindsDB repository on GitHub](https://github.com/mindsdb/mindsdb).
- Sign up for a [free MindsDB account](https://cloud.mindsdb.com/register?_ga=2.126448781.1875715573.1663498526-358045687.1658244666&_gl=1*1us5k78*_ga*MzU4MDQ1Njg3LjE2NTgyNDQ2NjY.*_ga_7LGFPGV6XV*MTY2MzYwNjcxNi4zNi4xLjE2NjM2MDY3MzUuMC4wLjA)
- Engage with the MindsDB community on [Slack](https://mindsdb.com/joincommunity?_ga=2.126448781.1875715573.1663498526-358045687.1658244666&_gl=1*1us5k78*_ga*MzU4MDQ1Njg3LjE2NTgyNDQ2NjY.*_ga_7LGFPGV6XV*MTY2MzYwNjcxNi4zNi4xLjE2NjM2MDY3MzUuMC4wLjA.) or [GitHub](https://github.com/mindsdb/mindsdb/discussions) to ask questions and share your ideas and thoughts.
  
If this tutorial was helpful, please give us a GitHub star [here](https://github.com/mindsdb/mindsdb).
