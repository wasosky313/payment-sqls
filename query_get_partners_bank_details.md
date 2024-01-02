# Query to get partners bank details from legacy database

### PARTNERS
```sql
select 
        p.razao_social as owner_name,
        db.agencia as account_branch,
        case
            when db.conta is not null then right(db.conta, 1)
            else db.conta
        end as account_digit,
        case
            when db.conta is not null then replace(left(db.conta, length(db.conta) - 1), '-', '')
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
        'P' as receiver_type,
        true as active,
        case
            when db.created_ds is not null then db.created_ds
            else ' ' 
        end as created_ds,
        p.cnpj as document_number,
        row_number() over () as event_id,
        db.created_at as created_at,
        cb.ispb as ispb_number,
        null as branch_digit
from dado_bancario db
    inner join parceiro p 
    on db.id = p.dado_bancario_id
    left join codigo_bancario cb
    on db.numero_do_banco = cb.compe
where p.ativo = true
	and (db.conta <> '12345-1' or db.conta is null);
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
=# \copy allowed_transfer_account from '/path/to/file/csv/partners.csv' delimiter ',' CSV HEADER;
```
