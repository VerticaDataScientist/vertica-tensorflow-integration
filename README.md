# TensorFlow Integration with Vertica Database
This repository demonstrates the integration of TensorFlow with the Vertica database. It showcases how to import a pre-trained TensorFlow model into Vertica, convert it to a frozen graph using `freeze_tf2_model.py`, and run an example of in-database scoring.

## Prerequisites
1. Vertica database installed and running
2. TensorFlow 2.x integration installed in Vertica
    ```bash
    $ /opt/vertica/bin/admintools -t install_package -d database_name -p 'password' --package TFIntegration
    ```

## Repository Structure
- `freeze_tf2_model.py` : Python script to convert a TensorFlow model to a frozen graph
- Tensorflow_Sentiment_Analysis.ipynb: A notebook example to trained TensorFlow model
- load_data.sql: SQL script to create a data example for scoring


## Setup Instructions
#### Clone the repository:

```bash
git clone https://github.com/VerticaDataScientist/vertica-tensorflow-integration.git
```

## Usage

Convert the TensorFlow model to a frozen graph:

```bash
python freeze_tf2_model.py your/model/directory path/to/save/frozen_model
```
This script takes the input model (your/model/directory) and converts it to a frozen graph (path/to/save/model).

- For complex data type add argument `1`:
```bash
python freeze_tf2_model.py your/model/directory path/to/save/frozen_model 1
```

### Import the frozen graph into Vertica:

Connect to the Vertica database using your preferred SQL client or command-line interface.
Run the following SQL command to import TensorFlow model:

```sql
SELECT IMPORT_MODELS ( 'path/to/save/frozen_model' USING PARAMETERS category='TENSORFLOW');
```
### Run the example in-database scoring:

Connect to the Vertica database.
Execute the `load_tf_data.sql` script using your preferred SQL client or command-line interface. This file will create a table of 3 rows of reviews and their sentiment and embeddings (you can use an UDX to create embedding to your string)

```sql
SELECT PREDICT_TENSORFLOW (* USING PARAMETERS model_name='frozen_model', num_passthru_cols=3) 
OVER(PARTITION BEST) AS (id, reviews, sentiment, prediction) 
FROM test_reviews ORDER BY id
```
or

```sql
-- return id, label and prediction columns
SELECT id, sentiment, prediction
from (SELECT PREDICT_TENSORFLOW (* USING PARAMETERS model_name='frozen_model', num_passthru_cols=3) 
OVER(PARTITION BEST) AS (id, reviews, sentiment, prediction) 
FROM test_reviews ORDER BY id ) AS res;
```

The script demonstrates how to use the TensorFlow model imported into Vertica to perform scoring on the provided example data.


## Embedding UDx Example

If you are interested in exploring the embedding User-Defined Extension (UDx), please refer to the example in the [UDX-Examples](https://github.com/VerticaDataScientist/UDX-Examples/tree/master/scalar-UDXs/embedding) repository. The example provides insights into the implementation and usage of the embedding UDx.

----
📚 _Note: Using TensorFlow with Complex Data Types in Vertica_

If you have a complex data type (ROW type) in a column and wish to leverage TensorFlow for scoring, Vertica provides support through the use of a bounded ARRAY. Specify the dimension during table creation using the syntax outlined in the [Vertica documentation](https://docs.vertica.com/23.4.x/en/sql-reference/data-types/complex-types/array/#syntax-for-column-definition).

Once your table is set up, you can use the `PREDICT_TENSORFLOW_SCALAR` function to perform scoring. Here's an example query:

```sql
SELECT id, review, pred.'Identity:0'[0] 
FROM (
  SELECT id, review, PREDICT_TENSORFLOW_SCALAR(embeddings USING PARAMETERS model_name='frozen_model') as pred 
  FROM idbm
) as res;
```
For more details, refer to the Vertica documentation on [ARRAY](https://docs.vertica.com/23.4.x/en/sql-reference/data-types/complex-types/array/#syntax-for-column-definition) data types and [PREDICT_TENSORFLOW_SCALAR](https://docs.vertica.com/23.4.x/en/sql-reference/functions/ml-functions/transformation-functions/predict-tensorflow-scalar/).

---
## Acknowledgments
This project is inspired by the need for integrating TensorFlow with Vertica and leveraging the power of in-database scoring.