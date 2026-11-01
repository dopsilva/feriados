# feriados

Os feriados do Brasil em [Sea](https://github.com/dopsilva/sea): nacionais,
estaduais (as 27 UFs), municipais, pontos facultativos e o calendário dos
bancos, com dias úteis, o próximo dia útil e a soma de dias úteis. Sem
dependências: as datas são calculadas (Páscoa, Carnaval, Corpus Christi,
feriados que mudam para a sexta ou para o domingo), com o histórico das leis
desde 1890.

```sh
sea get github.com/dopsilva/feriados
```

```lua
local feriados = require("feriados")

feriados.year(2026, {state = "SP"})
-- [{date = "2026-01-01", name = "Confraternização Universal", kind = "nacional", scope = "BR"},
--  {date = "2026-04-03", name = "Sexta-feira Santa", ...},
--  {date = "2026-07-09", name = "Revolução Constitucionalista", kind = "estadual", scope = "SP"}, ...]

feriados.holiday("2026-04-23", {state = "RJ"})          -- {name = "São Jorge", kind = "estadual", ...}
feriados.is_holiday("2026-11-20")                       -- true (nacional desde 2024)
feriados.is_business_day("2026-02-17", {calendar = "bank"})   -- false: Carnaval, os bancos fecham
feriados.next_business_day("2026-04-03")                -- "2026-04-06"
feriados.previous_business_day("2026-04-05")            -- "2026-04-02"
feriados.add_business_days("2026-12-23", 2)             -- "2026-12-28"
feriados.business_days_between("2026-01-01", "2026-01-31")    -- 21
feriados.easter(2026)                                   -- "2026-04-05"
```

As datas são texto `"AAAA-MM-DD"` (ou um `Time` de Sea, lido pela data).
Uma data ou um estado que não existe dá um erro.

## Opções

Todas as funções aceitam as mesmas opções (todas opcionais):

| Opção | O que faz |
|---|---|
| `state` | A UF (`"SP"`) ou o nome do estado (`"São Paulo"`, sem diferenciar maiúsculas): soma os feriados estaduais |
| `city` | `"São Paulo"` (a capital): soma os feriados municipais. Para outras cidades, use `extra` |
| `calendar` | `"civil"` (padrão): os feriados que fecham comércio e serviços. `"bank"`: o dos bancos (Resolução CMN 4.880/2020, o calendário da ANBIMA) — os nacionais, mais a segunda e a terça de Carnaval e Corpus Christi |
| `optional` | `true`: também os pontos facultativos federais (Carnaval, Quarta-feira de Cinzas, Corpus Christi, Dia do Servidor Público, vésperas de Natal e de Ano-Novo) |
| `extra` | Outros feriados: os do seu município, uma folga da empresa. Uma lista `[{date = "MM-DD" (todo ano) ou "AAAA-MM-DD" (só aquele), name, since?, until?}]` ou uma função `(ano) → [{date = "AAAA-MM-DD", name}]` |

Cada feriado tem `date`, `name`, `kind` e `scope`:

- `kind`: `"nacional"`, `"estadual"`, `"municipal"`, `"facultativo"`, `"bancario"` (os que só os bancos têm) ou `"extra"`;
- `scope`: `"BR"`, a UF, a cidade ou `""` (os extras).

Um dia pode ter mais de um feriado (no Acre, em 2026, o Tratado de Petrópolis
vai para a sexta, 20 de novembro, junto com a Consciência Negra): `holidays(date)`
traz todos, `holiday(date)` o primeiro (o nacional antes).

## Um calendário guardado

As funções de `feriados` guardam os anos já calculados para cada conjunto de
opções. Para usar sempre as mesmas, `feriados.calendar(opts)` devolve um
calendário com os mesmos métodos:

```lua
local cal = feriados.calendar({state = "PR", calendar = "bank", extra = [
    {date = "09-08", name = "Nossa Senhora da Luz dos Pinhais"},   -- Curitiba
]})
cal:is_business_day("2026-09-08")      -- false
cal:add_business_days("2026-09-04", 1) -- "2026-09-09"
cal:year(2026)
```

Dia útil é de segunda a sexta, sem feriado.
`business_days_between(de, até)` conta os dias úteis depois de `de` até `até`,
inclusive (negativo se `até` vem antes).

## Regras e fontes

- **Nacionais** desde 1890 (Decreto 155-B/1890, Decreto 19.488/1930, Lei
  662/1949, Lei 6.802/1980 — Nossa Senhora Aparecida —, Lei 14.759/2023 —
  Consciência Negra nacional a partir de 2024). A Sexta-feira Santa entra
  como nacional, como nos calendários dos bancos e da administração federal.
- **Estaduais** desde 1996 (Lei 9.093/1995), com as mudanças de cada estado:
  - a Consciência Negra estadual até 2023 (AL, AM, AP, MT, RJ);
  - Acre: os feriados que caem de terça a quinta vão para a sexta, desde 2009;
  - Santa Catarina: os dois feriados no primeiro domingo a partir da data;
  - Pernambuco: a Revolução de 1817 no primeiro domingo de março até 2017, e
    em 6 de março a partir de 2018 (Lei 16.059/2017);
  - Paraná: Nossa Senhora do Rocio no lugar da Emancipação a partir de 2014
    (Leis 18.384 e 18.397/2014);
  - Rio de Janeiro: São Jorge (Lei 5.198/2008) e a terça de Carnaval (Lei 5.243/2008);
  - Espírito Santo: Nossa Senhora da Penha, oito dias depois da Páscoa, a partir de 2020.
- **Capital paulista**: aniversário da cidade e Corpus Christi (Lei Municipal
  7.008/1967) e a Consciência Negra municipal de 2004 a 2023 (Lei Municipal 13.707/2004).

As regras foram portadas do pacote [holidays](https://github.com/vacanza/holidays)
(Python, licença MIT, Vacanza Team), que cita as leis de cada uma.

## Testes

```sh
sea test feriados_test.sea
```
