# phyloseq-cheatsheet

Minimal cheatsheet for functions in the phyloseq R package.


## Workflow

![Analysis workflow](./img/analysis_workflow.png)

> Attribution: McMurdie, Holmes (2013)
> https://doi.org/10.1371/journal.pone.0061217.g002


## Class accessors

![phyloseq class](./img/phyloseq_class.png)

> Attribution: McMurdie, Holmes (2013)
> https://doi.org/10.1371/journal.pone.0061217.g003


## Code snippets

Cheatsheet has four columns:
1. **Type** = Input, Data, Wrangle, Plot
2. **Function** = Function as called in script
3. **Description** = Terse description of function
4. **Example** = Minimal code example to demonstrate function; will often make
   use of built-in data from `phyloseq`

| Type    | Function         | Description          | Example                |
|---------|------------------|----------------------|------------------------|
| Data    | `enterotype`     |                      | `data(enterotype)`     |
| Data    | `GlobalPatterns` |                      | `data(GlobalPatterns)` |
| Data    | `esophagus`      | Subset of esophageal | `data(esophagus)`      |
| Wrangle | `psmelt`         | Change to data frame | `psmelt(esophagus)`    |


## License

[![CC0](http://mirrors.creativecommons.org/presskit/buttons/88x31/svg/cc-zero.svg)](https://creativecommons.org/publicdomain/zero/1.0/)

