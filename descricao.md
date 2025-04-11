Perfeito! Aqui está a versão atualizada do comentário com a explicação adicional sobre os aliases usados nas queries:

---

**Ticket Comment – 2025-04-11**

On 2025-04-09, we identified and reported a bug in the communication between EMIS and Change, similar to the issue previously addressed in the attached email dated 2025-01-25, where the operation status was not updated correctly from “M” to “XY”.

This time, the root cause was not related to the query itself (as each operation type – MT300, MT202, and BMC – has its own specific query), but rather to the UAT Autosys job, which did not trigger the corresponding process. There is a dedicated listener for each operation type: MT300 uses process 5027, MT202 uses 5158, and BMC uses 5166. In these cases, the issue occurred at the very beginning of the flow.

Once we identified the problem, we manually triggered the Autosys job, and we were able to confirm the creation of the .txt file and the successful sending to CashPro on 2025-04-10.

It’s important to note that the solution implemented back in January 2025 to support multiple payment references in the MT300 query was written as follows:

```sql
AND ( 
  Ope.REFERENCIA LIKE '%' + CONVERT(VARCHAR, Mo.CONTRATO)
  OR (Ope.FORMA = 'MANUAL' AND Ope.REFERENCIA LIKE '%' + CONVERT(VARCHAR, Mo.CONTRATO) + '%')
)
```

In this case (process 5027 – MT300), the aliases used are `Ope` for OP01 and `Mo` for MO01.

We have now applied a similar update to the MT202 query:

```sql
AND ( 
  Pmt.REFERENCIA LIKE '%' + CONVERT(VARCHAR, Ope.CONTRATO)
  OR (Pmt.FORMA = 'MANUAL' AND Pmt.REFERENCIA LIKE '%' + CONVERT(VARCHAR, Ope.CONTRATO) + '%')
)
```

For process 5158 – MT202, however, the aliases are different: `Pmt` refers to OP01 and `Ope` refers to MO01. This explains the difference in the query structure.

As of today (2025-04-11), we successfully executed three USD operations – one for BMC, one for MT202, and one for MT300 – all of which were completed without issues, as shown in the screenshot below.

---

Se quiser colocar isso diretamente no sistema de tickets ou mandar por e-mail, posso adaptar também.