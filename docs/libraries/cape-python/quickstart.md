# Cape Python API

This guide provides an example of using Cape Python with either Pandas or Spark.

## Prerequisites

* Python 3.6 or above.
* Cape Privacy recommends using a virtual environment such as [venv](https://docs.python.org/3/library/venv.html).


## Installation

You can install Cape Python with pip:

```shell
pip install cape-privacy
```

## Quickstart

### Write the policy

The data policy file defines the target data and permissions. It is written in YAML. Cape Python reads the `.yaml` policy file and applies the policies based on your [policy application script](#write-the-policy-application-script).

Create a `test-policy.yaml` file in your project, with the following content:

```yaml
label: test-policy
version: 1
rules:
# Set the column name
- match:
    name: weight
  actions:
    - transform:
        # This example shows an unnamed transformation.
        # It tells the policy runner to:
        # (1) Apply the transformation numeric-rounding
        # (2) Round to one decimal place
        type: numeric-rounding
        dtype: Double
        precision: 1
```


### Write the policy application script

To apply the policy `.yaml` to your data, you must run a script that defines which policy you apply to which data target.

Create a `test-transformation.py` file in your project, with the following content:


=== "Pandas"
    ```python
    import cape_privacy as cape
    import pandas as pd

    # Create a simple Pandas DataFrame
    df = pd.DataFrame([114.432, 134.622, 142.984], columns=["weight"])
    # Load the privacy policy
    policy = cape.parse_policy("test-policy.yaml")
    # Apply the policy to the DataFrame
    df = cape.apply_policy(policy, df, inplace=False)
    # Output the altered data
    print(df.head())
    ```

=== "Spark"
    ```python
    import cape_privacy as cape
    from pyspark import sql

    sess_builder = sql.SparkSession.builder
    sess_builder = sess_builder.appName('cape.examples.rounding')
    sess_builder = sess_builder.config('spark.sql.execution.arrow.enabled', 'true')
    sess = sess_builder.getOrCreate()

    # Create a simple Spark DataFrame
    df = sess.createDataFrame([114.432, 134.622, 142.984], "double").toDF("weight")
    # Load the privacy policy
    policy = cape.parse_policy("test-policy.yaml")
    # Apply the policy to the DataFrame
    df = cape.apply_policy(policy, df, inplace=False)
    # Output the altered data
    print(df.show())
    ```


### Run your transformations

The quickstart example creates a dataset programatically, so you can run the policy application script and view the output:

```shell
python test-transformation.py
```

!!! note
    You can choose where in your workflow to run your transformation scripts. Refer to [Best practices - Running transformations](/understand/best-practices/running-transformations/) for guidance.
