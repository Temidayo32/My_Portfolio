# Predicting Home Rental Prices with MindsDB in Java

In a previous [tutorial](https://curiousprogrammer.hashnode.dev/predicting-home-rental-prices-with-mindsdb-in-python), we covered creating, training and making predictions with MindsDB in Python. In this tutorial, we would be exploring how to do that in Java. Similarly in this tutorial, we would be predicting home rental prices using some features—number of rooms and home size in square feets—to make the predictions.

First off, we would be setting up our development environment. You will need access to a working MindsDB installation, which can be done locally or via [MindsDB Cloud](https://cloud.mindsdb.com/?_ga=2.121740043.1875715573.1663498526-358045687.1658244666&_gl=1*1hemhn1*_ga*MzU4MDQ1Njg3LjE2NTgyNDQ2NjY.*_ga_7LGFPGV6XV*MTY2MzU3MTAwOS4yOS4xLjE2NjM1NzI4OTEuMC4wLjA).

You can set up your MindsDB cloud account following this [guide](https://docs.mindsdb.com/setup/cloud/?_ga=2.21191739.1875715573.1663498526-358045687.1658244666&_gl=1*19ko74n*_ga*MzU4MDQ1Njg3LjE2NTgyNDQ2NjY.*_ga_7LGFPGV6XV*MTY2MzU3MTAwOS4yOS4xLjE2NjM1NzI4OTEuMC4wLjA). Alternatively, you can set up your MindsDB locally using [Docker](https://docs.mindsdb.com/setup/self-hosted/docker/?_ga=2.47799207.1875715573.1663498526-358045687.1658244666&_gl=1*28ggvt*_ga*MzU4MDQ1Njg3LjE2NTgyNDQ2NjY.*_ga_7LGFPGV6XV*MTY2MzU3MTAwOS4yOS4xLjE2NjM1NzI4OTEuMC4wLjA) or [Python](https://docs.mindsdb.com/setup/self-hosted/pip/source/?_ga=2.47799207.1875715573.1663498526-358045687.1658244666&_gl=1*28ggvt*_ga*MzU4MDQ1Njg3LjE2NTgyNDQ2NjY.*_ga_7LGFPGV6XV*MTY2MzU3MTAwOS4yOS4xLjE2NjM1NzI4OTEuMC4wLjA). For simplicity, we would be using MindsDB via the MindsDB cloud in this tutorial.

I will be coding using Vs-Code. As a Java developer, you might have preference for other IDEs such as Eclipse and Intellij. You can go ahead and use either of them. If you will be coding along with me in Vs-code, make sure to have [Extension pack for Java](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-pack) installed. Extension Pack for Java is a collection of popular extensions for Java, and hence will make it easier for you writing, testing and debugging Java applications in Vs-code. 

When Extension pack for Java is properly installed on your Vs-code, you should have the `Create a Java project` button displayed in the side bar of a new instance of your Vs-code. When prompted after clicking the button to create a new Java project, use the no-build tools option to create the new Java project. Follow the remaining instructions thereafter to successfully create the Java project.

## Installations

The following are the other required installations for this tutorial:

* Java
* Mysql JDBC driver(JDBC API)
  
We would be using the JDBC API to access the database, so make sure to have it downloaded and installed on your local machine. You can download the latest version [here](https://dev.mysql.com/downloads/connector/j/8.0.html). 

To use it in our Java project for accessing the database, make sure to have the Mysql JDBC driver added to your Java classpath. Vs-code allows you do this by adding the driver (jar file) to referenced libraries in the Java project folder in your Vs-code side bar. You add it to the referenced libraries by pointing it to the location on your local machine where you have the driver downloaded.

With all that done, let's get started with the tutorial.

## The Data

### Creating a Database

We would be creating our database from a MindsDB cloud account, and  for ease, we would be utilizing a demo database we've already prepared for you. So if you have an account created already go ahead and login. Copy the below code to create a database.

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

There are also other options for creating a database to use for this project. One of the options available to you is can use an existing database on your local machine. To use it for this project, you need to upload it to MindsDB via [MindsDB SQL Editor](https://docs.mindsdb.com/connect/mindsdb_editor/?_ga=2.79795991.1875715573.1663498526-358045687.1658244666&_gl=1*51tqjx*_ga*MzU4MDQ1Njg3LjE2NTgyNDQ2NjY.*_ga_7LGFPGV6XV*MTY2MzU5MzA4NC4zMy4xLjE2NjM1OTM1MDMuMC4wLjA).

Follow this [guide](https://docs.mindsdb.com/sql/create/file/?_ga=2.88224923.1875715573.1663498526-358045687.1658244666&_gl=1*1razhsj*_ga*MzU4MDQ1Njg3LjE2NTgyNDQ2NjY.*_ga_7LGFPGV6XV*MTY2MzU5NTUzOC4zNC4xLjE2NjM1OTU1NTcuMC4wLjA) to upload the file to MindsDB.

## Connecting to the Database

If you are coding in Vs-code, you should have this file structure created for you automatically:

```
C:.
├───.vscode
├───bin
├───lib
└───src
```

Our source code will be contained in the `src` folder; the compiled classes will be contained in the `bin` folder. You can go ahead and delete the default `app.java` file in the src folder or you can choose to modify it. If you deleted it, you can go ahead and create a new java file in the `src` folder with the name `mindsdb.java`. 

Now, modify it to reflect this:

```
// mindsdb.java

import java.sql.Connection;
import java.sql.DriverManager;

public class mindsdb {
    public static void main(String[] args) {

        final String url = "jdbc:mysql://cloud.mindsdb.com:3306/example_db"; 
        final String user = "MindsDB Cloud Username";  //same as your MindsDB Cloud email address
        final String password = "MindsDB Cloud Password";  //replace this value
        
        Connection conn = null;

        // creating a connection

        try {
            Class.forName("com.mysql.cj.jdbc.Driver");
            conn = DriverManager.getConnection(url, user, password);
            System.out.println("Connection to the database created successfully");

        } catch (Exception e) {
            e.printStackTrace();
            System.out.println("Oops, errors!");
        }  
    }
}
```

You  can also check that your connection is successful by running the java file.

```
Connection to the database created successfully
```

To run further checks on the connection, you might want to run queries on the database to see if it return some data. Now, modify the above code to reflect this:

```
\\ mindsdb.java

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;       //new line
import java.sql.Statement;       //new line

public class mindsdb {
    public static void main(String[] args) {

        final String url = "jdbc:mysql://cloud.mindsdb.com:3306/example_db"; 
        final String user = "MindsDB Cloud Username";  //same as your MindsDB Cloud email address
        final String password = "MindsDB Cloud Password";  //replace this value
        
        Connection conn = null;
        Statement stat = null;    //remember to add this new line!

        // creating a connection

        try {
            
            Class.forName("com.mysql.cj.jdbc.Driver");
            conn = DriverManager.getConnection(url, user, password);
            System.out.println("Connection to the database created successfully");

            // testing the connection

            String sql = "SELECT * FROM example_db.demo_data.home_rentals LIMIT 2";
            stat = conn.createStatement();
            ResultSet result = stat.executeQuery(sql);

            while (result.next()){
                int rooms = result.getInt("number_of_rooms");
                String roomstr = Integer.toString(rooms);
                int baths = result.getInt("number_of_bathrooms");
                String bathstr = Integer.toString(baths);
                int sqft = result.getInt("sqft");
                String sqftstr = Integer.toString(sqft);
                String location = result.getString("location");
                int rental = result.getInt("rental_price");

                System.out.println(roomstr + ',' + bathstr + ',' + sqftstr + ',' + location + ',' + rental);
            }
        }

        ...
    }
}
```

On execution, you should have this printed in your terminal:

```
2,1,917,great,3901
0,1,194,great,2042
```

## Training a Predictor With `CREATE PREDICTOR`

With that done, we can now train our first machine learning predictor. For that, we are going to use the `CREATE PREDICTOR` syntax where we would specify what sub-query to train `FROM` and what we want to  `PREDICT`.

Now, modify your code to include this. The code block should be directly inline and below the `while` statement.

```
\\mindsdb.java

....

.....

 // creating a predictor

            try {
                String predict = "CREATE PREDICTOR mindsdb.home_predictor FROM example_db (SELECT * FROM demo_data.home_rentals) PREDICT rental_price";
                Statement state = conn.createStatement();
                Boolean results = state.execute(predict);

                System.out.println(results);
                
            } catch (Exception e) {
                e.printStackTrace();
                System.out.println("Oops, errors creating predictor!");
            }

        ...
```

!!! note
    You can comment out this code block after running it once without errors. Running it again will trigger a `model exists` error.

## Checking the Status of a Predictor

It might take a couple of minutes to train a predictor. You can monitor the status of your predictor using this:

```
\\ mindsdb.java

...
....

            // checking status of the predictor
            try {
                String status = "SELECT status FROM mindsdb.predictors WHERE name='home_predictor'";
                Statement home = conn.createStatement();
                ResultSet set = home.executeQuery(status);
                
                while (set.next()){
                    String statuss = set.getString("status");

                    System.out.println(statuss);
                }
            } catch (Exception e) {
                e.printStackTrace();
                System.out.println("Oops errors displaying status!");
            }
        ...
    ...
```

If the training is complete, you should get this printed in your terminal:

```
complete
```

Once the status reads complete, we can start making predictions!

## Making Predictions

### Make Predictions with `SELECT`

Now, we can make predictions by querying the predictor. The `SELECT` syntax allows you make predictions for the label based on the chosen features.

We would be making our predictions from within the `try & catch` block for checking the status of the predictor. You can paste the code below inside the `try`block.

```
\\ mindsdb.java

...
....

// single predictions
                try {
                    String prediction = "SELECT * FROM mindsdb.home_rentals_model WHERE number_of_bathrooms=3 AND sqft=1500";
                    Statement get_predict = conn.createStatement();
                    ResultSet rs = get_predict.executeQuery(prediction);

                    while(rs.next()){
                        int rental_price = rs.getInt("rental_price");
                        Float confidence = rs.getFloat("rental_price_confidence");
                        int anomaly = rs.getInt("rental_price_anomaly");
                        int max = rs.getInt("rental_price_max");
                        int min = rs.getInt("rental_price_min");

                        // to string

                        String rentalstr = Integer.toString(rental_price);
                        String confidencestr = Float.toString(confidence);
                        String anomalystr = Integer.toString(anomaly);
                        String maxstr = Integer.toString(max);
                        String minstr = Integer.toString(min);

                        System.out.println("rental_price:" + rentalstr + "," + "confidence:" + confidencestr + ',' + "anomaly:" + anomalystr + ',' + "max:" + maxstr + ',' + "min:" + minstr);
                    }
                    
                } catch (Exception e) {
                    e.printStackTrace();
                    System.out.println("Oops, errors making predictions!");
                }

```

On execution, you should get this printed in your terminal:

```
rental_price:5003,confidence:0.99,anomaly:0,max:5085,min:4922
```

### Make Batch Predictions with `JOIN`

You might also want to make bulk predictions on several records in your table. In that case, you can achieve that by joining the table containing the data (`example_db.demo_data.home_rentals`) to your predictor.

In the status `try` block, paste the code block below to make bulk predictions.

```
\\ mindsdb.java

...
....

// bulk predictions
                try {
                    String bulk = "SELECT t.rental_price AS real_price, m.rental_price_confidence AS confidence, m.rental_price_anomaly AS anomaly, m.rental_price_max AS max, m.rental_price_min AS min, t.number_of_rooms,  t.number_of_bathrooms, t.sqft, t.location, t.days_on_market FROM example_db.demo_data.home_rentals AS t JOIN mindsdb.home_rentals_model AS m LIMIT 10";
                    Statement bulkstat = conn.createStatement();
                    ResultSet bulkset = bulkstat.executeQuery(bulk);

                    while (bulkset.next()) {
                        int brental_price = bulkset.getInt("real_price");
                        Float bconfidence = bulkset.getFloat("confidence");
                        int banomaly = bulkset.getInt("anomaly");
                        int bmax = bulkset.getInt("max");
                        int bmin = bulkset.getInt("min");

                        // to string

                        String brentalstr = Integer.toString(brental_price);
                        String bconfidencestr = Float.toString(bconfidence);
                        String banomalystr = Integer.toString(banomaly);
                        String bmaxstr = Integer.toString(bmax);
                        String bminstr = Integer.toString(bmin);

                        System.out.println("rental_price:" + brentalstr + "," + "confidence:" + bconfidencestr + ',' + "anomaly:" + banomalystr + ',' + "max:" + bmaxstr + ',' + "min:" + bminstr);
                        
                    }
                    
                } catch (Exception e) {
                    e.printStackTrace();
                    System.out.println("Oops, errors making predictions!");
                }
            ...
        ...
```

On execution, you should get this printed in your terminal:

```
rental_price:3901,confidence:0.99,anomaly:0,max:3967,min:3805
rental_price:2042,confidence:0.99,anomaly:0,max:2088,min:1925
rental_price:1871,confidence:0.99,anomaly:0,max:1946,min:1783
rental_price:3026,confidence:0.99,anomaly:0,max:3101,min:2938
rental_price:4774,confidence:0.99,anomaly:0,max:4829,min:4667
rental_price:4382,confidence:0.99,anomaly:0,max:4469,min:4306
rental_price:2269,confidence:0.99,anomaly:0,max:2353,min:2191
rental_price:2284,confidence:0.99,anomaly:0,max:2354,min:2191
rental_price:5420,confidence:0.99,anomaly:0,max:5518,min:5355
rental_price:5016,confidence:0.99,anomaly:0,max:5079,min:4917
```

## What's Next?

Have fun while trying it out yourself!

* Bookmark [MindsDB repository on GitHub](https://github.com/mindsdb/mindsdb).
* Sign up for a [free MindsDB account](https://cloud.mindsdb.com/register?_ga=2.126448781.1875715573.1663498526-358045687.1658244666&_gl=1*1us5k78*_ga*MzU4MDQ1Njg3LjE2NTgyNDQ2NjY.*_ga_7LGFPGV6XV*MTY2MzYwNjcxNi4zNi4xLjE2NjM2MDY3MzUuMC4wLjA)
* Engage with the MindsDB community on [Slack](https://mindsdb.com/joincommunity?_ga=2.126448781.1875715573.1663498526-358045687.1658244666&_gl=1*1us5k78*_ga*MzU4MDQ1Njg3LjE2NTgyNDQ2NjY.*_ga_7LGFPGV6XV*MTY2MzYwNjcxNi4zNi4xLjE2NjM2MDY3MzUuMC4wLjA.) or [GitHub](https://github.com/mindsdb/mindsdb/discussions) to ask questions and share your ideas and thoughts.
  
If this tutorial was helpful, please give us a GitHub star [here](https://github.com/mindsdb/mindsdb).