# Pythonic codebase structure

## Table of Contents

- [Introduction](#introduction)
- [Folder structure](#folder-structure)
- [Data Contract Validation Module](#data-contract-validation-module)
    - [DataContractUtility class](#datacontractutility)
        - [Sorting JSON](#sort_dict)
        - [File check](#check_if_file_exists)
        - [Convertion of Yaml file to JSON](#load_yaml_file)
        - [Fetching list of DataContracts in DataProduct](#list_contract_yamls)
        - [Fetching DataDomain from DP ID](#get_domain)
        - [Get DP tags as JSON from catalog](#get_dp_tags)
        - [Get DP comment from catalog](#get_dp_comment)
    - [DataContractKeyValidator class](#datacontractkeyvalidator)
        - [Not Null check for all Keys](#validate_key_not_null)
        - [Description key check](#validate_key_description)
        - [gaia_dp_id key check](#validate_key_gaia_dp_id)
        - [title key check](#validate_key_title)
        - [product_owner key check](#validate_key_product_owner)
        - [version key check](#validate_key_version)
        - [refresh_rate key check](#validate_key_refresh_rate)
        - [apiVersion key check](#validate_key_apiVersion)
        - [kind key check](#validate_key_kind)
        - [datasetDomain key check](#validate_key_datasetDomain)
        - [sourcePlatform key check](#validate_key_sourcePlatform)
        - [sourceSystem key check](#validate_key_sourceSystem)
    - [DataContract class](#datacontract)
        - [DataProduct Check](#check_if_dp_exists)
        - [Fetch Metadata of a DataProduct](#get_metadata_dict_dp)
        - [Fetch data in the DataContract File](#get_data_contract)
        - [Fetching version to validate DataContract against](#get_version)
        - [Extract the specification Yaml file](#get_specification)
        - [Check if updates present in a DataContract](#check_updates)
        - [Validation of a DataContract](#validate_data_contract)

## Introduction
This folder contains source code (src) in the form of a Python package "gaia_data_products",
comprising a number of modules to host reusable Python classes and functions to support and develop
Databricks data pipelines (bronze to silver).

## Folder structure

- src/ --> container for all reusable code
- src/gaia_data_products/ --> root of "package" (including an __init__.py)
- [src/gaia_data_products/dp_validator/](#data-contract-validation-module) --> module for validating data products (eg yaml specs vs yaml of product)
- [src/gaia_data_products/dp_metadata/](#dp_metadata) --> module to use a yaml of a product, and "attach" that metadata to the product (basically wrapper around current ALTER table setting the TAGS, and the COMMENT statement for the description).

## Data Contract Validation Module
Python Module for validating the DataContract yaml file. (i.e. dp_validator/yaml_validator.py)
This module consists of three classes

- [DataContractUtility](#datacontractutility)
- [DataContractKeyValidator](#datacontractkeyvalidator)
- [DataContract](#datacontract)

### DataContractUtility
This is a utility class, all the methods declared in this class are static. These methods are bound to the class and not the object of the class.
Methods declared in the class:
- ##### sort_dict
    Sorting a Python dictionary based on the keys. 
    **Input:** Python dictionary
    **Returns:** Sorted Python dictionary based on key
- ##### check_if_file_exists
    Check whether a file exists in the mentioned path.
    **Input:** File Path
    **Returns:** AssertException if the file is not present
- ##### load_yaml_file
    Fetches the yaml file in the mentioned path and return its content in the form of JSON using pyyaml library
    **Input:** yaml file path
    **Returns:** JSON variable
- ##### [list_contract_yamls](https://gitlab.com/syngentagroup/cp-rnd/data-platform/gaia/databricks-dataops-data-products/-/blob/GAIA-1449-data-product-v1-0-python-validation-module/src/gaia_data_products/dp_validator/yaml_validator.py?ref_type=heads#L43)
    List the yaml files present in the given directory, if the directory is not passed, it will list the yamls present in the current directory
    **Input:** Directory path
    **Returns:** list of yaml files in the given directory
- ##### get_domain
    Take data product id as input and get the domain to which it is pertaining to
    **Input:** Data Product ID
    **Returns:** Domain of the data product
- ##### get_dp_tags
    This fetches the tags applied to a table in the form of JSON using the information_schema in the specific catalog
    **Input:** catalog_name, table_name
    **Returns:** JSON object containing tags and its values of the mentioned dataproduct
- ##### get_dp_comment
    This method fetches the comment applied to the dataproduct using the information_schema in the specific catalog
    **Input:** catalog_name, table_name
    **Returns:** Returns string which contains the comment applied to the mentioned dataproduct

### DataContractKeyValidator
This class contains methods for validating keys of the datacontract. A pre-defined method for validating each key in a data contract. Apart from this method, there is a method defined to validate every key in the data contract whether it is null or not.
- ##### validate_key_not_null
    Checks if all the keys in the DataContract are not null
    **Input:** DataContract JSON (dp_datacontract)
    **Returns:** Raises AssertException if any of the key is null
- ##### validate_key_description
    Validate the description key value in the datacontract file. Checks if the value in the description key is of string datatype
    **Input:** DataContract JSON (dp_datacontract)
    **Returns:** Raises AssertException if the description is not of string datatype
- ##### validate_key_gaia_dp_id
    Validate the DataProduct ID(gaia_dp_id key) in the datacontract file, if the id is in the format gaiaxxxxx
    **Input:** DataContract JSON (dp_datacontract)
    **Returns:** Raises AssertException if the gaia_dp_id is not in valid format
- ##### validate_key_title
    Validate the title key in the datacontract file. Checks if the value in the title key is of string datatype 
    **Input:** DataContract JSON (dp_datacontract)
    **Returns:** Raises AssertException if the title is not of string datatype
- ##### validate_key_product_owner
    Validate the product_owner key in the datacontract file. The value contains multiple emails seperated by comma(','). Checks if each mail mentioned is from the syngenta organization (mail ends with @syngenta.com)
    **Input:** DataContract JSON (dp_datacontract)
    **Returns:** Raises AssertException if the product_owner deviates the rules specified
- ##### validate_key_version
    Validate the version key in the datacontract file. Checks if the values is of Semantic versioning [SemVar](https://semver.org/). This method checks if the values is of form MAJOR.MINOR.PATCH or MAJOR.MINOR. (i.e. 0.1, 1.0.0, 1.2.0 etc)
    **Input:** DataContract JSON (dp_datacontract)
    **Returns:** Raises AssertException if the version is of SemVar type
- ##### validate_key_refresh_rate
    Validate the refresh_rate key in the datacontract file. Refresh rate of a dataproduct file must be one of the following 'daily','weekly','monthly','once'.
    **Input:** DataContract JSON (dp_datacontract)
    **Returns:** Raises AssertException if the refresh rate key deviates the rules specified.
- ##### validate_key_apiVersion
    Validate the apiVersion key in the datacontract file. Checks if the values is of Semantic versioning [SemVar](https://semver.org/). This method checks if the values is of form MAJOR.MINOR.PATCH or MAJOR.MINOR. (i.e. 0.1, 1.0.0, 1.2.0 etc)
    **Input:** DataContract JSON (dp_datacontract)
    **Returns:** Raises AssertException if the version is of SemVar type
- ##### validate_key_kind
    Validate the kind key in the datacontract file. Checks if the value is equal to DataContract.
    **Input:** DataContract JSON (dp_datacontract)
    **Returns:** Raises AssertException if the kind is not of DataContract.
- ##### validate_key_datasetDomain
    Validate the datasetDomain key in the datacontract file. This checks whether the dataproduct is realted to particular domain or not. To fetch the domain of the dataproduct based on the id, and the domain must be one of the following 'portfolio','ppd','productsafetyregulatory','realworld','research'.
    **Input:** DataContract JSON (dp_datacontract)
    **Returns:** Raises AssertException if the domain is not valid or domain not related to the dataproduct
- ##### validate_key_sourcePlatform
    Validate the sourcePlatform key in the datacontract file. This checks whether the sourcePlatform is 'gaia'.
    **Input:** DataContract JSON (dp_datacontract)
    **Returns:** Raises AssertException if the sourcePlatform key deviates the checks.
- ##### validate_key_sourceSystem
    Validate the sourceSystem key in the datacontract file. This checks whether the sourceSystem is 'databricks'
    **Input:** DataContract JSON (dp_datacontract)
    **Returns:** Raises AssertException if the sourceSystem key deviates the checks.

### DataContract
This class contains methods that check if the dataproduct already exists or not, extract the tags and comments of a dataproduct in the form of JSON, fetch the data contract in the form of JSON, fetch the specification YAML in the form of JSON.
- ##### check_if_dp_exists
    Checks if a dataproduct is present in the specified catalog. If the arguments are not passed then method uses class variables
    **Input:** target_catalog, gaia_dp_id
    **Returns:** True if exists, otherwise False
- ##### get_metadata_dict_dp
    Fetches the metadata of a dataproduct inclusing comment. For fetching these values used methods in DataContractUtility.
    **Input:** target_catalog, gaia_dp_id
    **Returns:** Returns sorted JSON of the metadata
- ##### get_data_contract
    Checks if DataContract file is present or not. If presents then fetches the data in DataContract.yaml file into python JSON object.
    **Input:** gaia_dp_id
    **Returns:** Returns sorted JSON of the datacontract
- ##### get_version
    Fetches the specification version with which datacontract needs to be verified with.
    **Input:** spec_version
    **Returns:** Returns the version
- ##### get_specification
    Fetches the specification yaml file and returns the data as JSON
    **Input:** uses class variables, spec_version
    **Returns:** Returns sorted JSON of the specification yaml file
- ##### check_updates
    Compare the metadata of a dataproduct(tags,comment) with datacontract data. If there are updates returns true and specifies taskValue true for key "valid".
    **Input:** dp_metadata, data_contract
    **Returns:** True if updates are present otherwise False
- ##### validate_data_contract
    Validate the datacontract with the specification yaml of that specific version. Sets taskValues True if the datacontract is valid, version with which the datacontract is validated against
    **Input:** data_contract, dp_specification
    **Returns:** Creates two taskValues Valid, Version based on the validation results.

## dp_metadata
