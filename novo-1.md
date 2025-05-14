# 🆕 CNPJ Alfanumérico (2026): Guia Completo de Adaptação

---

## 📅 1. Cronograma Oficial

| Etapa                                   | Data              |
| --------------------------------------- | ----------------- |
| Publicação da IN RFB nº 2.229           | 15/10/2024        |
| Início da vigência                      | 25/10/2024        |
| CNPJs com letras começam a ser emitidos | **Julho de 2026** |

🔗 Fonte: [gov.br/receitafederal](https://www.gov.br/receitafederal/pt-br/assuntos/noticias/2024/outubro/cnpj-tera-letras-e-numeros-a-partir-de-julho-de-2026)

---

## 🧾 2. Estrutura do CNPJ

### 🔹 **Formato Atual (até julho de 2026)**

* **14 dígitos numéricos**: `NNNNNNNNNNNNNN`

  * Raiz: 8 dígitos
  * Estabelecimento: 4 dígitos
  * DV: 2 dígitos verificadores

### 🔸 **Formato Futuro (a partir de julho de 2026)**

* **12 caracteres alfanuméricos (A–Z, 0–9)** + **2 dígitos verificadores**
* DV calculado com base nos valores ASCII das letras

---

## 🧮 3. Algoritmo Unificado de Validação (Python)

```python
def char_to_val(c):
    if c.isdigit():
        return int(c)
    return ord(c.upper()) - 55  # A=10, B=11, ..., Z=35

def calcula_dv_alfanumerico(cnpj_base: str) -> str:
    def calc_dv(cnpj, pesos):
        soma = sum(char_to_val(c) * p for c, p in zip(cnpj, pesos))
        resto = soma % 11
        return '0' if resto < 2 else str(11 - resto)

    pesos_1 = [5, 4, 3, 2, 9, 8, 7, 6, 5, 4, 3, 2]
    pesos_2 = [6] + pesos_1

    dv1 = calc_dv(cnpj_base, pesos_1)
    dv2 = calc_dv(cnpj_base + dv1, pesos_2)

    return dv1 + dv2

def valida_cnpj(cnpj: str) -> bool:
    base = cnpj[:12]
    dv = cnpj[12:]
    return calcula_dv_alfanumerico(base) == dv
```

---

## 🧪 4. Exemplos de Validação

| Tipo   | CNPJ Base (12) | DV | Completo         | Válido |
| ------ | -------------- | -- | ---------------- | ------ |
| Antigo | 123456780001   | 95 | 12345678000195   | ✅      |
| Antigo | 000000000001   | 91 | 00000000000191   | ✅      |
| Novo   | A1B2C3D4E5F600 | 70 | A1B2C3D4E5F60070 | ✅      |
| Novo   | BRASIL000001   | 91 | BRASIL00000191   | ✅      |
| Novo   | ALFA001BR002   | 86 | ALFA001BR00286   | ✅      |

---

## 🛠️ 5. Orientações para Adaptação de Sistemas

### 🔸 5.1 Estruturas de Dados

| Componente             | Ação recomendada                                           |
| ---------------------- | ---------------------------------------------------------- |
| **Banco de Dados**     | Mude de `INT`, `NUMERIC` ou `CHAR(14)` para `VARCHAR(14)`  |
| **Inputs/Formulários** | Permitir letras A–Z                                        |
| **Máscaras**           | Atualizar para `************-**`                           |
| **APIs**               | Validar e aceitar alfanuméricos em contratos de integração |
| **PKs/Índices**        | Evite usar CNPJ como chave se for tipo numérico            |

---

### 🔸 5.2 Validadores

**Antigos validadores numéricos deixarão de funcionar.**
➡️ Adote o **validador unificado** com `char_to_val()`.

---

### 🔸 5.3 Integrações

Audite sistemas como: SEFAZ, eSocial, SPED, ERPs, bancos, certificados digitais.

**Ações:**

* Contato técnico com terceiros
* Versão de API com suporte duplo
* Logs e fallback temporário

---

### 🔸 5.4 Relatórios e BI

Problemas possíveis:

* Ordenação incorreta (alfanumérico ≠ numérico)
* Filtros incompletos

**Soluções:**

* Use `VARCHAR(14)`
* Revise regras de ETL, agrupamentos e dashboards

---

### 🔸 5.5 Boas Práticas

| Ação                         | Objetivo                              |
| ---------------------------- | ------------------------------------- |
| Ambiente de testes           | Validar CNPJs com letras              |
| Validação dupla              | Comparar nova e antiga regra          |
| Logs e monitoramento         | Rastrear erros por tipagem/validação  |
| Documentação técnica interna | Disseminar entendimento e uso correto |

---

## 🚨 6. Riscos e Soluções

| Sintoma                   | Causa Provável                     | Solução                          |
| ------------------------- | ---------------------------------- | -------------------------------- |
| CNPJs válidos rejeitados  | Tipagem errada ou validador antigo | Use novo algoritmo + VARCHAR(14) |
| Integrações quebradas     | Terceiros não atualizados          | Contato técnico e fallback       |
| Relatórios inconsistentes | Agrupamento numérico               | Corrigir ETL e BI                |
| Ordenação incorreta       | Ordenação numérica aplicada        | Adotar lógica alfanumérica       |

---

## ✅ 7. Ações Prioritárias

1. Alterar colunas para `VARCHAR(14)`
2. Atualizar máscaras e validadores
3. Auditar integrações externas
4. Incluir testes automatizados com CNPJs alfanuméricos
5. Treinar equipe e atualizar documentação

---

## 🧩 8. SQL Server: Adaptação Técnica

### 🔹 8.1 Encontrar Colunas com CNPJ

```sql
SELECT TABLE_SCHEMA, TABLE_NAME, COLUMN_NAME, DATA_TYPE
FROM INFORMATION_SCHEMA.COLUMNS
WHERE COLUMN_NAME LIKE '%CNPJ%';
```

---

### 🔹 8.2 Alterar Tipagem para VARCHAR(14)

```sql
ALTER TABLE Empresas
ALTER COLUMN CNPJ VARCHAR(14);
```

> ⚠️ Remova qualquer `CHECK CONSTRAINT` que valide apenas dígitos.

---

### 🔹 8.3 UDF para Validação de CNPJ (ASCII-aware)

```sql
CREATE FUNCTION dbo.ValidaCNPJAlfanumerico (@cnpj VARCHAR(14))
RETURNS BIT
AS
BEGIN
    DECLARE @i INT = 1, @char CHAR(1), @val INT, @soma1 INT = 0, @soma2 INT = 0
    DECLARE @pesos1 TABLE (pos INT, peso INT)
    DECLARE @pesos2 TABLE (pos INT, peso INT)

    INSERT INTO @pesos1 VALUES (1,5),(2,4),(3,3),(4,2),(5,9),(6,8),(7,7),(8,6),(9,5),(10,4),(11,3),(12,2)
    INSERT INTO @pesos2 SELECT pos + 1, peso FROM @pesos1
    INSERT INTO @pesos2 VALUES (1,6)

    WHILE @i <= 12
    BEGIN
        SET @char = SUBSTRING(@cnpj, @i, 1)
        SET @val = CASE WHEN @char BETWEEN '0' AND '9' THEN CAST(@char AS INT)
                       ELSE ASCII(UPPER(@char)) - 55 END
        SELECT @soma1 += @val * peso FROM @pesos1 WHERE pos = @i
        SET @i += 1
    END

    DECLARE @dv1 INT = CASE WHEN (@soma1 % 11) < 2 THEN 0 ELSE 11 - (@soma1 % 11) END

    SET @i = 1
    WHILE @i <= 13
    BEGIN
        SET @char = SUBSTRING(@cnpj, @i, 1)
        SET @val = CASE WHEN @i = 13 THEN @dv1
                        WHEN @char BETWEEN '0' AND '9' THEN CAST(@char AS INT)
                        ELSE ASCII(UPPER(@char)) - 55 END
        SELECT @soma2 += @val * peso FROM @pesos2 WHERE pos = @i
        SET @i += 1
    END

    DECLARE @dv2 INT = CASE WHEN (@soma2 % 11) < 2 THEN 0 ELSE 11 - (@soma2 % 11) END

    RETURN CASE WHEN RIGHT(@cnpj, 2) = CAST(@dv1 AS VARCHAR) + CAST(@dv2 AS VARCHAR)
                THEN 1 ELSE 0 END
END;
```

---

### 🔹 8.4 Testar CNPJs no Banco

**CNPJs válidos:**

```sql
SELECT CNPJ FROM Empresas WHERE dbo.ValidaCNPJAlfanumerico(CNPJ) = 1;
```

**CNPJs inválidos:**

```sql
SELECT CNPJ FROM Empresas WHERE dbo.ValidaCNPJAlfanumerico(CNPJ) = 0;
```

---

## 📚 Conclusão

A chegada dos CNPJs alfanuméricos é uma mudança **inevitável e impactante**. Adaptar seus sistemas com **antecedência** é fundamental para:

✅ Evitar falhas críticas

✅ Garantir conformidade fiscal

✅ Preservar a integridade dos dados

✅ Manter a operação e integração fluídas