# Desafio Técnico: Customer Success Técnico — Woovi

**Candidato:** Guilherme Almeida da Luz

**Data de Entrega:** Junho de 2026

**Status do Teste:** Entrega Parcial (Desafio 2 Iniciado)

---

<br>
<br>
<br>

# Ticket #02 — Pix Automático trimestral some entre painel e API

**Canal:** chat in-app
**Cliente:** Helix SaaS (fictício) — SaaS B2B
**Quem fala:** Customer B, CTO
**Prioridade percebida pelo cliente:** média (mas com cara de "isso é bug, não é?")

---

## Mensagem do cliente

> Oi, time. Estou cadastrando uma assinatura de Pix Automático aqui no painel e na hora de escolher a frequência aparece **Trimestral** como opção. Selecionei e segui o fluxo até o final, sem erro.
>
> Aí fui replicar via API e o enum de `frequency` na doc lista só `WEEKLY`, `MONTHLY`, `SEMIANNUALLY` e `ANNUALLY`. Mandei `QUARTERLY` mesmo assim, deu 400 com `invalid enum value`.
>
> O que está certo, painel ou API? Preciso fechar a integração até sexta. Anexo dois prints: um da doc e um do painel mostrando "Trimestral" selecionado.

---

# Respostas:

### > O que o Cliente está realmente pedindo:

### > Plano de Investigação:

**(na segunda avaliação que fiz, após terminar o primeiro teste técnico)**

Olhando na https://woovi.com/pix/pix-automatico/ ,
https://ajuda.woovi.com/hc/duvidas-frequentes/articles/pix-automatico-woovi
https://developers.woovi.com/api-redoc#tag/subscription/paths/~1api~1v1~1subscriptions/post

encontrei,

frequency
string
Enum: "WEEKLY" "MONTHLY" "BIMONTHLY" "TRIMONTHLY" "QUARTERLY" "SEMIANNUALLY" "ANNUALLY"
Frequency of the subscription. When type is PIX_RECURRING, only the BCB-allowed subset applies: WEEKLY, MONTHLY, QUARTERLY, SEMIANNUALLY, ANNUALLY

https://developers.woovi.com/api#tag/subscription/POST/api/v1/subscriptions
frequency
Type:string
enum
Frequency of the subscription

WEEKLY
MONTHLY
BIMONTHLY
TRIMONTHLY
QUARTERLY
SEMIANNUALLY
ANNUALLY

Quarterly is the standard term used to describe events or payments occurring every three months or four times a year, aligned with the four calendar quarters (Q1–Q4). It is preferred in business, finance, and formal contexts because it is widely understood and implies a fixed alignment with the annual cycle (e.g., January–March, April–June).

Trimonthly technically means every three months but is rarely used and can be ambiguous, as some interpret it as occurring three times a month. Due to this potential confusion and lower formality, it is generally recommended to use quarterly or the phrase "every three months" instead.

### > Hipóteses:

_**1. Realmente existe opções à mais na plataforma: (na primeira avaliação que fiz)**_

Com base no landing page que fala sobre Pix Automático, foi identificado o seguinte em [Landing Page - Pix Automático](https://woovi.com/pix/pix-automatico/) no `FAQ (Perguntas Frequentes)`:

> Como funciona o Pix Automático?

    > O Pix Automático permite definir valor e periodicidade (semanal, mensal, **`trimestral`** etc.). Funciona tanto com valores fixos quanto variáveis. A autorização pode ser cancelada pelo pagador ou recebedor até 23h59 do dia anterior (pagador) ou até 22h (recebedor) da data da cobrança.

Observa-se que existe uma afirmação que indica a opção **`trimestral`** (como não possuo acesso ao ambiente de teste para averiguar todas as possibilidades que a plataforma oferece e que podem ser diferentes da documentação da API, faço essa suposição). Além disso, tentei obter essa informação via vídeos tutoriais no YouTube, mas essa opção não foi alterada no exemplo, ficando com a opção mensal pré-selecionada.

Já, na seção [Create a new Subscription - API Scalar​](https://developers.woovi.com/api#tag/subscription/POST/api/v1/subscriptions) ou [Create a new Subscription - API Redoc](https://developers.woovi.com/api-redoc#tag/subscription/paths/~1api~1v1~1subscriptions/get) da API, o `enum` do tipo `string` possui apenas 4 opções em EN-US (Semanalmente, Mensalmente, Semestralmente e Anualmente), a saber:

- WEEKLY
- MONTHLY
- SEMIANNUALLY
- ANNUALLY

_**2. A plataforma pode exibir no Front-End uma coisa e se referir a outra no Back-end: (na primeira avaliação que fiz)**_

Provavelmente a verdade absoluta sobre a pergunta do cliente `"O que está certo, painel ou API?"` esteja com as regras de negócio no Back-end, logo, a documentação da API exibe o `enum` correto, sendo necessário averiguar se houve um mal entendido no Front-end que gerou essa inconsistência de informações.

### > Resposta ao cliente:

### > Procedimentos Internos:

### > Referências Consultadas:

- [Pix Automático - visão geral](https://developers.woovi.com/docs/category/pix-autom%C3%A1tico)
  - [Pix Automático > Como criar um Pix Automático?](https://developers.woovi.com/docs/pix-automatic/pix-automatic-how-to-create)
    - [Create a new Subscription - API Scalar​](https://developers.woovi.com/api#tag/subscription/POST/api/v1/subscriptions)
    - [Create a new Subscription - API Redoc](https://developers.woovi.com/api-redoc#tag/subscription/paths/~1api~1v1~1subscriptions/get)

- [Landing Page - Pix Automático](https://woovi.com/pix/pix-automatico/)

- [Woovi | Dúvidas Frequentes (FAQ) > Pix Automático](https://ajuda.woovi.com/hc/duvidas-frequentes/pt_BR/categories/pix-automtico)
  - [Pix Automático > Pix Automático Woovi](https://ajuda.woovi.com/hc/duvidas-frequentes/articles/pix-automatico-woovi)

https://developers.openpix.com.br/docs/category/introdu%C3%A7%C3%A3o
