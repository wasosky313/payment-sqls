# payment-sqls

 
## Query para ver todos os status do pagamento pelo legacy_id

```sql
select e.id, p.legacy_id, p.receiver_document, p.receiver_name, p.amount, e.payment_status, s.description status,
e.initiation, e.description, e.payment_type, e.transaction_id, e.end_to_end_id,
       e.receiver_key, e.receiver_key_type, e.receiver_account, e.receiver_ispb, e.receiver_account_type,
       e.request, e.response, e.payment_id, to_char(e.updated_at AT TIME ZONE 'UTC','DD-MM-YYYY HH24:MI:SS') updated_at
  from payment_event e
inner join payment_status s on e.payment_status = s.id
inner join payment p on p.id = e.payment_id
where p.legacy_id in (150227)
order by e.updated_at;
```
