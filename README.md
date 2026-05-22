# Como usar a função `GetCustomizedResultForFields`

#### 🛑 DISCLAIMER / AVISO IMPORTANTE

> **Atenção:** Este arquivo e a função M fornecida servem estritamente como um **exemplo didático de integração**. 
> 
> * **Sem Suporte Técnico:** Não prestamos suporte, consultoria ou assistência no desenvolvimento, manutenção ou correção de consultas personalizadas, relatórios e dashboards criados a partir deste código.
> * **Responsabilidade:** A construção das regras de negócio, paginações complexas, tratamento de erros e a performance dos painéis são de inteira responsabilidade do usuário/desenvolvedor do relatório.

A função `GetCustomizedResultForFields` e outdas já estão disponíveis nesse arquivo Power Query. Ela consome a `EngemanDataApi` retornando dados de tabelas com suporte a filtros, ordenação e paginação.

Você pode invocá‑la de **duas formas**:

1. **Via interface visual "Inserir Parâmetros"** – recomendada para usuários que preferem preencher campos sem digitar código.
2. **Diretamente na fórmula M** – útil para automação ou reuso em outras consultas.

## 📌 Pré‑condição obrigatória

Antes de usar a função, certifique‑se de que as **variáveis globais** `BaseUrl`, `Login` e `Password` estejam definidas no ambiente.

> ✅ **Boa prática:** Centralize essas credenciais usando os **Parâmetros** que já estão disponíveis no arquivo Power Query. Assim você altera um único local e evita expor dados sensíveis diretamente na função.

---

## 🖱️ Forma 1 – Usando a interface visual "Inserir Parâmetros"

Esta é a maneira mais simples quando você quer testar a função ou obter uma tabela rapidamente.

### Passo a passo

1. No **Editor Avançado do Power Query**, localize a consulta que contém, por exemplo, a função `GetCustomizedResultforFields`.
2. **Clique** sobre o nome da consulta e **observe o painel a direita**.
3. Será aberto um diálogo com os campos da função:

   | Campo         | Obrigatório | Como preencher                                                                 |
   |---------------|-------------|--------------------------------------------------------------------------------|
   | `tablename`   | ✅ Sim      | Nome da tabela na API (ex: `APLIC`, `ORDSERV`).                   |
   | `onlyFields`  | ✅ Sim      | Lista de campos separados por vírgula. Use `*` para todos.                     |
   | `sort`        | ✅ Sim      | Ex: `"descricao asc"`, `"datpro desc"`.                                               |
   | `condition`   | ❌ Opcional | Filtro SQL‑like (ex: `"codapl > 100 and ativo = 'S'"`). Deixe em branco se não houver. |
   | `range`       | ❌ Opcional | Registro inicial (paginação). Padrão `"0"` Exemplo: `0-50`.                                    |
   | `limit`       | ❌ Opcional | Máximo de registros (padrão `"999999"`). **Sempre utilize um limite razoável**. |

4. Preencha os valores.
5. Clique em **Invocar** – o Power Query executará a função e criará uma nova consulta com o resultado.

> 💡 **Dica:** Após a primeira invocação, você pode editar os parâmetros clicando no ícone de engrenagem ao lado do nome da consulta.

---

## ⌨️ Forma 2 – Chamando a função diretamente em M

Use esta abordagem quando quiser combinar o resultado com outras consultas ou criar processos paginados.

```powerquery
// Exemplo básico
let
    TabelaResultado = GetCustomizedResultForFields(
        "aplic",                   // tablename
        "codapl, tag, descricao",  // onlyFields
        "descricao asc",           // sort
        "ativo = 'S'",             // condition
        "0-100",                   // range
        "100"                      // limit
    )
in
    TabelaResultado
```
### 🔁 Exemplo de paginação manual (para mais de 10 mil registros)

```powerquery
let
    Pag1 = GetCustomizedResultForFields("solicitacao", "*", "codsol asc", null, "0-1000", "1000"),
    Pag2 = GetCustomizedResultForFields("solicitacao", "*", "codsol asc", null, "1000-2000", "1000"),
    Pag3 = GetCustomizedResultForFields("solicitacao", "*", "codsol asc", null, "2000-3000", "1000"),
    Tudo = Table.Combine({Pag1, Pag2, Pag3})
in
    Tudo
```

>📌 Em cenários reais, crie uma função recursiva ou uma lista de chamadas dinâmicas para automatizar a paginação.

---

## ⚠️ Solução de erros comuns

| Erro no Power Query                          | Causa provável                                   | Como resolver                                                                 |
|----------------------------------------------|--------------------------------------------------|-------------------------------------------------------------------------------|
| `The name 'BaseUrl' wasn't recognized`       | Variável global `BaseUrl` não existe.            | Crie a consulta/parâmetro `BaseUrl`.                                          |
| `401 Unauthorized`                           | Login ou senha inválidos.                        | Verifique `Login` e `Password`.                                              |
| `400 Bad Request`                            | `onlyFields` ou `sort` com nome de campo inválido. | Consulte a documentação da API para os nomes corretos.                        |
| `Timeout`                                    | `limit` muito alto ou rede instável.             | Reduza o `limit` (ex: 1000). Aumente o timeout em `Arquivo > Opções > Consulta`. |

---

>Agora você já pode usar a função `GetCustomizedResultforFields` com segurança – tanto pela interface visual quanto por código.

>Em caso de dúvidas sobre tabelas ou campos disponíveis, consulte a [**documentação da API Engeman Data**](https://engeman.com.br/engeman/manual_api_restful_engeman.pdf).


