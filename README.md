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
### RETRY ULTIMAS 12 HORAS
### Da para ver se hay algum pagamento duplicado com status 2 (risgo)
```sql
SELECT
   payment.legacy_id as transferencia_id,
   contagem_ordem_recebida as "Ordem Recebida",
   contagem_em_andamento as "Em andamento", 
   contagem_aguardando_confirmacao as "Aguardando confirmação", 
   contagem_pagamento_sucesso as "Confirmado com sucesso", 
   contagem_falha as "Falha", 
   to_char(payment.updated_at AT TIME ZONE 'UTC','DD-MM-YYYY HH24:MI:SS')  as "Data"
FROM
   payment
   LEFT JOIN
      (
         select distinct 
  				p.legacy_id as transferencia_id,  
  				s.description as ultimo_status,
  				count(s.description) as contagem_ordem_recebida
  		from payment_event e
  		inner join payment_status s on e.payment_status = s.id 
  		inner join payment p on p.id = e.payment_id  
  		where 1 = 1 
  		and e.payment_status = 0
  		group by s.description, p.legacy_id   
      )
      AS total_ordem_recebida
      ON payment.legacy_id = total_ordem_recebida.transferencia_id
   LEFT JOIN
      (
         select distinct 
  				p.legacy_id as transferencia_id,  
  				s.description as ultimo_status,
  				count(s.description) as contagem_em_andamento
  		from payment_event e
  		inner join payment_status s on e.payment_status = s.id 
  		inner join payment p on p.id = e.payment_id  
  		where 1 = 1 
  		and e.payment_status = 1
  		group by s.description, p.legacy_id   
      )
      AS total_em_andamento
      ON payment.legacy_id = total_em_andamento.transferencia_id
   LEFT JOIN
      (
         select distinct 
  				p.legacy_id as transferencia_id,  
  				s.description as ultimo_status,
  				count(s.description) as contagem_aguardando_confirmacao
  		from payment_event e
  		inner join payment_status s on e.payment_status = s.id 
  		inner join payment p on p.id = e.payment_id  
  		where 1 = 1 
  		and e.payment_status = 2
  		group by s.description, p.legacy_id 
      )
      AS total_aguardando_confirmacao
      ON payment.legacy_id = total_aguardando_confirmacao.transferencia_id
   LEFT JOIN
      (
         select distinct 
  				p.legacy_id as transferencia_id,  
  				s.description as ultimo_status,
  				count(s.description) as contagem_pagamento_sucesso
  		from payment_event e
  		inner join payment_status s on e.payment_status = s.id 
  		inner join payment p on p.id = e.payment_id  
  		where 1 = 1 
  		and e.payment_status = 3
  		group by s.description, p.legacy_id              
      )
      AS total_pagamento_sucesso
      ON payment.legacy_id = total_pagamento_sucesso.transferencia_id
    LEFT JOIN
      (
         select distinct 
  				p.legacy_id as transferencia_id,  
  				s.description as ultimo_status,
  				count(s.description) as contagem_falha
  		from payment_event e
  		inner join payment_status s on e.payment_status = s.id 
  		inner join payment p on p.id = e.payment_id  
  		where 1 = 1 
  		and e.payment_status in (4, 5, 6, 7, 8)
  		group by s.description, p.legacy_id 
      )
      AS total_falha
      ON payment.legacy_id = total_falha.transferencia_id      
where
   1 = 1  
   --and contagem_aguardando_confirmacao > 1
   and (contagem_ordem_recebida > 1 or contagem_em_andamento > 1 or contagem_aguardando_confirmacao > 1 or contagem_pagamento_sucesso > 1 or contagem_falha > 1)
   and payment.updated_at >= (now() - interval '24 hour')
   order by payment.updated_at desc;
```
