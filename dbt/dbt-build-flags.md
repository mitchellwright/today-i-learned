# Different Flags For Building in dbt

When running the dbt `run` command there are multiple flags you can use to affect what is built. If the command `dbt run` is used without any flags it will execute all of the mdoels in the dependency graph.

The most common flag is `--select` (or `-s` for a short version) which allows you to [specify a subset of nodes to build](https://docs.getdbt.com/reference/node-selection/syntax#specifying-resources).

You can also use [graph operators](https://docs.getdbt.com/reference/node-selection/graph-operators) to specify which edges should also be stepped through. For example, if I want to build the `dim_customers` table and any downstream models, I would run the command `dbt run --select dim_customers+` with the `+` modifier at the end of the model name. To run all the upstream models, I would run the same command but with the modifier at the front of the model, `dbt run --select +dim_customers`.
