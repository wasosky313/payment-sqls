# Query to get partners bank details from legacy database

### PARTNERS
```sql
select 
        case
            when trim(db.banco) = '' then ' '
            when db.banco is not null then db.banco
            else ' '
        end as owner_name,
        case
            when db.agencia is not null then left(trim(db.agencia), length(trim(db.agencia)) - 1)
            else db.agencia
        end as account_branch,
        case
            when db.conta is not null then right(db.conta, 1)
            else db.conta
        end as account_digit,
        case
            when db.conta is not null then left(db.conta, length(db.conta) - 1)
            else db.conta
        end as account_number,
        case
            when db.tipo_conta = 'conta_poupanca' then 'SVGS'
            when db.tipo_conta = 'conta_pagamento' then 'TRAN'
            when db.tipo_conta = 'conta_corrente' then 'CACC'
            when db.tipo_conta = 'conta_corrent' then 'CACC'
            else 'CACC'
        end as account_type,
        db.chave_pix as pix_transfer_key,
        'F' as receiver_type,
        true as active,
        case
            when db.created_ds is not null then db.created_ds
            else ' ' 
        end as created_ds,
        f.cnpj as document_number,
        row_number() over () as event_id,
        db.created_at as created_at,
        case 
            when cb.ispb is not null then cb.ispb
            else null
        end as ispb_number,
        case
            when db.agencia is not null then right(db.agencia, 1)
            else db.agencia
        end as branch_digit
from dado_bancario db
    inner join fornecedor f 
    on db.id = f.dado_bancario_id
    left join codigo_bancario cb
    on db.numero_do_banco = cb.compe
where f.ativo = true
```

* Save cvs file from metabase


### connect to database via psql
- if don't have psql installed -> https://www.timescale.com/blog/how-to-install-psql-on-mac-ubuntu-debian-windows/ 

```shell
psql postgres://<username>:<password>@<host>:<port>/<database>

# for example
psql postgres://postgres:postgres@localhost:5432/postgres
```

### execute csv file into database (postgresql shell)
- How can use psql -> https://hasura.io/docs/latest/schema/postgres/postgres-guides/import-data-from-csv/

```shell
=# \copy allowed_transfer_account from '/path/to/file/csv/suppliers.csv' delimiter ',' CSV HEADER;
```
