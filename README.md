
# Coding Challenge

## Overview

We have data from an external computer simulation that we want incorporated into our application.

The simulation:

* can have any number of input variables
* is given arrays of values for each input variable
* calculates a single output value for each combination of input values
* produces a single JSON object as output, that contains all of the provided input values, as well as the results of running the simulation against all possible combinations of those input values.


![Output Logo](./../diagrams/output-format.png)

**Your task is to create an object that takes one of these simulation output objects as an input, described below, and transform it into a 2-dimensional array that looks like this:**

```
[
    [input-1-1, input-2-1, input-3-1, output-1-1-1],
    [input-1-2, input-2-1, input-3-1, output-2-1-1],
    [input-1-3, input-2-1, input-3-1, output-3-1-1],
    ...
    [input-1-i, input-2-j, input-3-k, output-i-j-k]
]
```

## Inputs
The input to your component is an output from a simulation.

A given simulation output consists of:

1. metadata providing details on the input and output variables, which are used to locate and extract the variable values used in the simulation.
2. arrays of input variables, which contain arrays of values that were used as the inputs for the analysis.
3. arrays of model output values for all possible combinations of input values.


## The Metadata Object Attributes

Each simulation output is guaranteed to have the 2 following keys:


```
{
    "variables": Array<string>;
    "mapping": string;
    ...
}
```

These two keys should be read to find the keys to the simulation inputs and outputs.

### The Input Variables/Values

The root-level `variables` key, an array of strings, provides the names of the input variables.

So if a simulation was run with 3 input variables, the `variables` key would look like this:

```
{
        "variables": ['latitude', 'longitude', 'year']
}
```

Each input variable in the `variables` array has a corresponding root-level key of the same name, which is an array containing every possible input value for that variable.

So given a `variables` key of  ```['latitude', 'longitude', 'year']```, the arrays of input values used in the analysis for each variable could like this:

```
{
    "variables": ['latitude', 'longitude', 'year'],
    ...
    "latitude": [40.71708, 29.452412, 38.370274, 48.69622],
    "longitude": [236.875, 238.125, 251.5625, 279.0625, 275.625, 258.4375]
    "year": [2020, 2021, 2022, 2023, 2024, 2025],
    ...
}
```

A value taken from each input variable array is a set of inputs fed into the model, like so:

```
output_value = model(latitude[0], longitude[0], year[0])
output_value = model(40.71708, 236.875, 2020)
```

## The Model Ouput Variable/Values

The information about the location of the output values can be found in the root-level `mapping` key.  The `mapping` key points to a string value, which is the name of the root-level attribute containing the array of model output values.

A model consisting of an output variable named `number_of_days` would look like this:

```
{
    ...
    "mapping": 'number_of_days',
    ...
    "number_of_days": [N-Dimensional of Output Values] 
}
```

The array of output values is an `N`-dimensional array, where `N` is the number of input variables. Each array dimension corresponds to a particular input variable, whose order is defined by its position in the `variables` input array.

Each output dimension is an array that has the same length as the array of input values for that particular variable. The index of an element in a dimension array points to the value used for that variable in the corresponding input value array.

## Example

Let's say the output from a simulation that you're given looks like this:

```
{
    "variables": ['latitude', 'longitude', 'year'],
    "mapping": 'number_of_days',
    "latitude": [40.71708, 29.452412, 38.370274],
    "longitude": [236.875, 238.125, 251.5625, 279.0625],
    "year": [2020, 2021],
    "number_of_days": [
        [ [ 40, 93 ], [ 10, 15 ], [ 69, 15 ], [ 27, 1 ] ],
        [ [ 99, 35 ], [ 81, 56 ], [ 83, 99 ], [ 48, 3 ] ],
        [ [ 76, 52 ], [ 95, 31 ], [ 5, 62 ], [ 37, 55 ] ]
    ]
}
```

1. Looking at `variables`, there are 3 input variables; `latitude`, `longitude` and `year`; so the output should be a 3-dimensional array
2. Looking at `mapping`, there is a single output variable named `number_of_days`
3. Input Values

    a. The first value in `variables` is `latitude`, so the first dimension of `number_of_days` corresponds to the `latitude` array which has a length of 3, so the first dimension of the output array also has a length of 3.

    b. The second value in `variables` is `longitude`, so the second dimension of `number_of_days` corresponds to the `longitude` array, which has a length of 4

    b. The third value in `variables` is `year`, so the third dimension of `number_of_days` corresponds to the `year` array, which has a length of 2

4. Based on the input variables and the lengths of their input arrays, we can retrieve a model output for a given set of input parameters by: ```data.number_of_days[i][j][k]```, where

   * 0 <= i < 3
   * 0 <= j < 4
   * 0 <= k < 2

5. The input value for a particular variable can by found by referencing the input values array using the same index used in the output value. So for ```data.number_of_days[i][j][k]```:

    a. The input value for `latitude` is found by data.latitude[i]
    b. The input value for `longitude` is found by data.longitude[j]
    c. The input value for `year` is found by data.year[k]

6. The output from the given input should be: 

    ```
    [
        [40.71708, 236.875, 2020, 40],
        [29.452412, 236.875, 2020, 99],
        [38.370274, 236.875, 2020, 76],
        ...
        [38.370274, 279.0625, 2021, 55]
    ]
    ```

    For this example, there should be 24 total rows.
