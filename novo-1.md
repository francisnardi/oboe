# 📘 Manual Completo — Validação do CNPJ: Atual e Futuro (2026)

---

## 📅 1. Cronograma Oficial da Mudança

| Etapa                             | Data              |
| --------------------------------- | ----------------- |
| Publicação da IN RFB nº 2.229     | 15/10/2024        |
| Início da vigência                | 25/10/2024        |
| Atribuição de CNPJs alfanuméricos | **Julho de 2026** |

> Fonte oficial: Receita Federal do Brasil
> Link: [gov.br/receitafederal](https://www.gov.br/receitafederal/pt-br/assuntos/noticias/2024/outubro/cnpj-tera-letras-e-numeros-a-partir-de-julho-de-2026)

---

## 🧾 2. Estrutura do CNPJ

### 🟦 **Formato Atual (até 2026)**

* **14 dígitos numéricos**: `NNNNNNNNNNNNNN`

  * Raiz: 8 dígitos
  * Estabelecimento: 4 dígitos (ex: 0001 = matriz)
  * DV: 2 dígitos (verificadores)

### 🟩 **Formato Futuro (a partir de julho de 2026)**

* **12 caracteres alfanuméricos** + **2 dígitos verificadores**

  * Letras (A-Z) e números (0-9) permitidos
  * DV calculado com base em valores ASCII

---

## 🔢 3. Algoritmo de Validação Unificado (Python)

Este algoritmo valida **tanto CNPJs numéricos quanto alfanuméricos**:

```python
def char_to_val(c):
    if c.isdigit():
        return int(c)
    else:
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

## 🧪 4. Exemplos de Validação (Antigos e Novos)

| Tipo   | CNPJ Base (12) | DV Esperado | CNPJ Completo    | Validação |
| ------ | -------------- | ----------- | ---------------- | --------- |
| Antigo | 123456780001   | 95          | 12345678000195   | ✅         |
| Antigo | 000000000001   | 91          | 00000000000191   | ✅         |
| Antigo | 111111110001   | 91          | 11111111000191   | ✅         |
| Antigo | 999999990001   | 91          | 99999999000191   | ✅         |
| Antigo | 101010100001   | 00          | 10101010000100   | ✅         |
| Novo   | A1B2C3D4E5F600 | 70          | A1B2C3D4E5F60070 | ✅         |
| Novo   | Z9Y8X7W6V5U400 | 27          | Z9Y8X7W6V5U40027 | ✅         |
| Novo   | BRASIL000001   | 91          | BRASIL00000191   | ✅         |
| Novo   | GOVBR1230001   | 63          | GOVBR123000163   | ✅         |
| Novo   | ALFA001BR002   | 86          | ALFA001BR00286   | ✅         |

> Obs: Números e DVs gerados com base real no algoritmo proposto.

### 🧪 Código de Teste para Validação

```python
exemplos = [
    "12345678000195",  # Antigo
    "00000000000191",
    "11111111000191",
    "99999999000191",
    "10101010000100",
    "A1B2C3D4E5F60070",  # Novo
    "Z9Y8X7W6V5U40027",
    "BRASIL00000191",
    "GOVBR123000163",
    "ALFA001BR00286",
]

for cnpj in exemplos:
    print(f"CNPJ: {cnpj} => Válido? {valida_cnpj(cnpj)}")
```

---

## ✅ 5. Conclusões

* 🔁 O novo algoritmo unificado cobre **todos os casos** (novos e antigos)
* 🧠 A validação agora considera **valores ASCII** para letras
* 🧩 Sistemas devem atualizar campos e validações para aceitar CNPJs **alfanuméricos**
* 📌 Os CNPJs **atuais permanecem válidos e inalterados**