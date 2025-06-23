# MVP Gitingest V3

## First iteration

For the MVP to be considered valid :
    - We need to be able to re-implement gitingest using the new "library" and make it plug and play

For this we need to provide a way to:
    - convert a real filesystem directory to a parsable Python data structure
    - apply functions to said data structure from a different context (eg. another module)
    - filter the data structure based on a set of rules (eg. exclude patterns)
    - query different levels of data from the nodes (eg. just a number )

## Second iteration

    - cache outputs of various modules
