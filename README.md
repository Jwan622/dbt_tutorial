This is the code repo for the dbt tutorial at https://www.startdataengineering.com/post/dbt-data-build-tool-tutorial

I followed along and my notes are on the bottom\
-Jeff Wan

### Prerequisites

1. [Docker](https://docs.docker.com/get-docker/) and [Docker compose](https://docs.docker.com/compose/install/)
2. [dbt](https://docs.getdbt.com/dbt-cli/installation/)
3. [pgcli](https://www.pgcli.com/install)
4. [git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)

Clone the git repo and start the data warehouse docker container

```bash
git clone https://github.com/josephmachado/simple_dbt_project.git
docker compose up -d
```

## Run dbt 

```bash
export DBT_PROFILES_DIR=$(pwd)
cd sde_dbt_tutorial
dbt snapshot
dbt run
dbt test
dbt docs generate
dbt docs serve
```

Insert updates into source customer table, to demonstrate snapshot

```bash
pgcli -h localhost -U dbt -p 5432 -d dbt
# password is password1234
COPY warehouse.customers(customer_id, zipcode, city, state_code, datetime_created, datetime_updated) FROM '/input_data/customer_new.csv' DELIMITER ',' CSV HEADER;
\q
```

Run snapshot and create models again.

```
dbt snapshot
dbt run
```

You can log into the data warehouse to see the models.

```bash
pgcli -h localhost -U dbt -p 5432 -d dbt
# password is password1234
select * from warehouse.customer_orders limit 3;
\q
```

## Stop docker container

```bash
cd ..
docker compose down
```

#### Other notes by Jeff Wan
Notice a file named `stg_eltool__customers.sql` in `sde_dbt_tutorial/models/staging`. If you used airflow to EL, then it'd be named `stg_airflow__customers.sql`

Notice a marekting models folder where models are created in the postgres database that are the result of joining two tables. We can define the models for sa the marketing department's end users. A project can have multiple business verticals. Having one folder per business vertical provides an easy way to organize the models.

The `stg_eltool__customers` model requires snapshots.customers_snapshot model. But snapshots are not created on dbt run ,so we run dbt snapshot first.

Our staging and marketing models are as materialized views, and the two core models are materialized as tables.


When we run:
```
COPY warehouse.customers(customer_id, zipcode, city, state_code, datetime_created, datetime_updated)
FROM '/input_data/customer_new.csv' DELIMITER ',' CSV HEADER;
```

that copies input data from that csv and causes dbt to create a new row in the snapshots model when you run `dbt snapshots`:
```bash
dbt@localhost:dbt> select * from snapshots.customers_snapshot where customer_id = 83;
+---------------+-----------+-----------+--------------+---------------------+---------------------+----------------------------------+---------------------+---------------------+---------------------+
| customer_id   | zipcode   | city      | state_code   | datetime_created    | datetime_updated    | dbt_scd_id                       | dbt_updated_at      | dbt_valid_from      | dbt_valid_to        |
|---------------+-----------+-----------+--------------+---------------------+---------------------+----------------------------------+---------------------+---------------------+---------------------|
| 83            | 04548     | sao paulo | SP           | 2017-10-18 00:00:00 | 2017-10-18 00:00:00 | 33bd3f9d04b5fc38fa679f8fc34ca3b5 | 2017-10-18 00:00:00 | 2017-10-18 00:00:00 | 2017-10-18 01:20:00 |
| 83            | 24120     | niteroi   | RJ           | 2017-10-18 00:00:00 | 2017-10-18 01:20:00 | 2a0076283bfc6b53a8fca323c95732e9 | 2017-10-18 01:20:00 | 2017-10-18 01:20:00 | <null>              |
```
