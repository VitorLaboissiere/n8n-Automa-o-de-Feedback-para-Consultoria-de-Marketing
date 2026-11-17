# Feedback automático para consultores com IA (n8n)

> 📚 **Projeto de estudo** desenvolvido acompanhando o curso **"Agentes de IA e n8n"** da **Hashtag Treinamentos**. Por conta própria, acrescentei o tratamento de consultores **sem avaliação no dia** (ver "Melhoria que eu fiz").

Automação no **n8n** que, todo dia, analisa as avaliações que os clientes deram a cada consultor de uma consultoria de marketing (respostas de um Google Forms em uma planilha). Quando a **média do dia fica abaixo de 7**, uma **IA (Google Gemini)** escreve um e-mail de feedback construtivo para o consultor, com a planilha das avaliações do dia em anexo.

## Como funciona

```
Schedule Trigger → Lista de consultores → Loop Over Items (um consultor por vez)
   → Buscar avaliações de hoje → Tem avaliação?
        ├─ não → volta ao Loop (próximo consultor)
        └─ sim → Edit Fields → Summarize → Média < 7?
                    ├─ não → volta ao Loop
                    └─ sim → AI Agent (Gemini) → Buscar avaliações → Edit Fields → .xlsx → Gmail → volta ao Loop
```

1. **Schedule Trigger**: roda todos os dias.
2. **Get row(s) BuscarConsultores**: lê a lista de consultores (nome e e-mail). É essa lista que o loop percorre.
3. **Loop Over Items**: entrega **um consultor por vez** pela saída `loop` e só passa ao próximo quando o item volta. A saída `done` dispara quando todos foram processados.
4. **Buscar_Avaliação**: busca as respostas do formulário com **dois filtros (AND)**: o consultor atual e a data de hoje (`$today`).
5. **tem Avaliação?**: se o consultor não teve avaliação hoje, volta direto ao Loop.
6. **Edit Fields + Summarize**: padroniza os campos e calcula a **média** de satisfação, a **quantidade** de avaliações e **junta todas as sugestões** em um texto só.
7. **If (média < 7)**: só quem ficou abaixo da meta recebe feedback. Os demais voltam ao Loop.
8. **AI Agent (Google Gemini)**: recebe o nome do consultor (`$('Loop Over Items').item.json.Nome`), a média, o total e as sugestões. A **System Message** define o papel de "Supervisor de Qualidade", pede para agrupar comentários parecidos por tema, usar um tom de mentoria (não de punição) e responder em **HTML**.
9. **Buscar_Avaliação1 → Edit Fields1 → Convert to File**: como o Summarize resumiu tudo em uma linha, as avaliações são buscadas de novo e viram um **.xlsx** com as avaliações do dia.
10. **Send a message (Gmail)**: envia o feedback ao consultor com o anexo e volta ao Loop.

## Melhoria que eu fiz
No fluxo original, se um consultor **não tinha nenhuma avaliação no dia**, a busca voltava vazia. Com isso, o item não retornava ao loop e **os consultores seguintes não eram analisados**. Se o *Always Output Data* fosse usado para forçar a passagem, a média chegava **vazia** ao If, que valida o tipo número de forma estrita, e o fluxo dava **erro**.

**Solução:** mantive o *Always Output Data* no `Buscar_Avaliação` (para que o próximo nó sempre execute) e criei o If **"tem Avaliação?"** logo depois. Sem avaliação, o fluxo volta direto ao Loop e segue para o próximo consultor. Também desliguei o *Always Output Data* do Loop Over Items, que não era necessário.

## Planilhas esperadas
- **Consultores**: colunas `Nome` e `Email`.
- **Avaliações** (respostas do Forms): `Data`, `Seu Nome`, `Por qual consultor você foi atendido ? *`, nota de satisfação, sugestão de melhoria e notas de performance.

## Como usar
1. No n8n, use **Import from File** para importar `workflow-feedback-consultores.json`.
2. Conecte as credenciais do **Google Sheets**, do **Gmail** e do **Google Gemini**.
3. Nos nós do Google Sheets, aponte para as suas planilhas de consultores e de avaliações.
4. Teste com **Execute workflow**, incluindo um consultor sem avaliação no dia.

## Tecnologias
n8n · AI Agent · Google Gemini · Google Sheets · Google Forms · Gmail · Loop Over Items · Summarize · engenharia de prompt

---
Autor: Vitor Alexandre Laboissiere Pereira · [LinkedIn](https://www.linkedin.com/in/vitorlaboissiere)
