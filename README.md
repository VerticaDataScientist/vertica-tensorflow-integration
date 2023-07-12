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
- example_model/: Directory containing an example pre-trained TensorFlow model
- example_data/: Directory containing example data for scoring
- example_query.sql: SQL script demonstrating in-database scoring using the TensorFlow model


## Setup Instructions
#### Clone the repository:

```bash
git clone https://github.com/your-username/vertica-tensorflow-integration.git
```

## Usage

Convert the TensorFlow model to a frozen graph:

```bash
python freeze_tf2_model.py your/model/directory path/to/save/frozen_model
```
This script takes the input model (your/model/directory) and converts it to a frozen graph (path/to/save/model).

### Import the frozen graph into Vertica:

Connect to the Vertica database using your preferred SQL client or command-line interface.
Run the following SQL command to create a user-defined function (UDF) that loads the TensorFlow model:

```sql
SELECT IMPORT_MODELS ( 'path/to/save/frozen_model' USING PARAMETERS category='TENSORFLOW');
```
Optionally, you can create a wrapper function that calls the UDF and performs any necessary preprocessing or postprocessing steps.

### Run the example in-database scoring:

Connect to the Vertica database.
Execute the `example_query.sql` script using your preferred SQL client or command-line interface.

```sql
SELECT PREDICT_TENSORFLOW (*
                   USING PARAMETERS model_name='frozen_model', num_passthru_cols=1)
                   OVER(PARTITION BEST) FROM data_table;
```

The script demonstrates how to use the TensorFlow model imported into Vertica to perform scoring on the provided example data.

## Contributing
Contributions are welcome! If you find any issues or have suggestions for improvements, please open an issue or submit a pull request.

## License
This repository is licensed under the MIT License.

## Acknowledgments
This project is inspired by the need for integrating TensorFlow with Vertica and leveraging the power of in-database scoring. Special thanks to the TensorFlow and Vertica communities for their valuable contributions and support.