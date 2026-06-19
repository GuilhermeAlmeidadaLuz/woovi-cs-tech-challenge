# Desafio Técnico: Customer Success Técnico — Woovi

**Candidato:** Guilherme Almeida da Luz

**Data de Entrega:** Junho de 2026

**Status do Teste:** Entrega Total (Desafio 1 Concluído)

---

<br>
<br>
<br>

# Ticket #01 — Webhook não chegou após pagamento confirmado

**Canal:** WhatsApp (grupo de integração)
**Cliente:** Acme Gateway (fictício) — gateway parceiro, ~R$ 2M / mês processados via Woovi
**Quem fala:** Customer A, dev sênior do lado deles
**Prioridade percebida pelo cliente:** alta

---

## Mensagem do cliente

> Bom dia, pessoal. Estou olhando aqui no console que essas três cobranças aparecem como `COMPLETED`, mas a gente não recebeu o webhook de `OPENPIX:TRANSACTION_RECEIVED` em nenhuma das três. Já passaram mais de 40 minutos.
>
> Charges:
>
> - `0a1b2c3d4e5f60718293a4b5c6d7e8f9`
> - `1f2e3d4c5b6a7980a1b2c3d4e5f60718`
> - `9988776655443322ffeeddccbbaa1100`
>
> Nosso endpoint de webhook está online, conseguimos receber outras cobranças sem problema. Já cliquei em "reenviar webhook" no dashboard pra primeira charge e aí veio — mas isso não é sustentável. Algum incidente rolando?
>
> Pra contexto: a gente credita o saldo do nosso cliente final assim que recebe o webhook. Enquanto não chega, ele fica reclamando que "pagou e não caiu". Imagina se isso escala.

---

# Respostas:

Pick up each ticket as if it had just landed in your inbox. For each one, write your response covering:

- Your understanding of the problem — what is the customer really asking?
- Your investigation plan — where do you look (internal tools, docs, Bacen, code, people) and in what order?
- Hypotheses — 2–4 possible root causes, ranked, with what would confirm each.
- The reply you would send to the customer — written as if you were sending it.
- Internal follow-ups — bug? doc gap? feature request? where do you register it?

We care more about how you investigate and communicate than whether you happen to know the answer by heart.

---

### > O que o Cliente está realmente pedindo:

    Segurança e atualização em tempo real no recebimento das cobranças dos seus clientes, contudo, pode estar ocorrendo uma confusão nos conceitos de __eventos de cobrança__ (`Os eventos de cobrança são enviados quando uma cobrança é paga.`) e __eventos de transação__ (`Os eventos de transação são enviados quando uma transação é recebida.`) ou indisponibilidade que exigiu o reenvio manual após as 8 tentativas automáticas do webhook.

    [Tipos de eventos do Webhook - API Woovi](https://developers.woovi.com/docs/webhook/webhook-events-type)

### > Plano de Investigação:

Como se trata de um dev sênior do lado deles, que utiliza o serviço da Woovi, minha fonte primária de busca seria na documentação da API [Configure a plataforma Woovi](https://developers.woovi.com/docs/intro/getting-started). Dadas as palavras-chaves que ele utilizou: `cobranças`,`OPENPIX:TRANSACTION_RECEIVED`, `Charges`, `endpoint`, `webhook`.

Onde eu olharia:

- 1. [Webhook visão geral](https://developers.woovi.com/docs/category/webhook-2)
  - 1.1. [Webhook > Tipos de eventos do Webhook](https://developers.woovi.com/docs/webhook/webhook-events-type) para visualizar as especificações da API para o tipo de evento `OPENPIX:TRANSACTION_RECEIVED`:
    - 1.1.1. [Payload de transação recebida](https://developers.woovi.com/docs/webhook/examples/webhook-transaction-received) - `OPENPIX:TRANSACTION_RECEIVED`
      - 1.1.1.1. [O que é um Pix avulso?](https://ajuda.woovi.com/hc/duvidas-frequentes/articles/o-que-e-um-pix-avulso)
    - 1.1.2 [Payload de Cobrança Pix Paga](https://developers.woovi.com/docs/webhook/examples/webhook-charge-payload) - `OPENPIX:CHARGE_COMPLETED`

  - 1.2. [Webhook > Verificando se o Webhook está chamando corretamente o endpoint configurado](https://developers.woovi.com/docs/webhook/platform/webhook-check-work) - caso haja necessidade de verificar logs

  - 1.3. [Webhook > Regras de Retentativa do Webhook](https://developers.woovi.com/docs/webhook/webhook-retry)

  - 1.4. [Reenviando webhooks na plataforma](https://developers.woovi.com/docs/webhook/webhook-resend)

### > Hipóteses:

Baseadas em leituras realizadas na documentação com as palavras-chaves mencionadas no plano de investigação e no entendimento obtido.

**1. Tipo de Evento esperado diferente do que chegou no console - o evento que ele espera é do tipo transação para uma cobrança**

Partindo do ponto que:

- "**Eventos de transação:** Os eventos de transação são enviados quando uma transação é recebida";
  - `OPENPIX:TRANSACTION_RECEIVED`: "Esse evento é enviado quando uma transação é recebida, seja ela de uma cobrança ou de um QR code estático. Dependendo da transação elas podem vir seguintes tipos: DYNAMIC ,STATIC ou PAYMENT. Onde PAYMENT seria o [pix avulso](https://ajuda.woovi.com/hc/duvidas-frequentes/articles/o-que-e-um-pix-avulso)";

  ```Json
  <!-- Trecho do payload de transação recebida -->
  {
      "event": "OPENPIX:TRANSACTION_RECEIVED",
      "charge": null,
      "pixQrCode": null,
      "pix": {
          ...
          "payer": {
              "name": "Nome Sobrenome",
              "email": "email@email.com",
              "phone": "+5500988776655",
              "taxID": {
                  "taxID": "98765432100",
                  "type": "BR:CPF"
              },
          "correlationID": "2566b5bd-2e1a-463d-b21c-ec7717cf2df4"
          },
          ...
          "status": "CONFIRMED",
          "type": "PAYMENT",
          ...
      }
  }
  ```

- "**Eventos de cobrança:** Os eventos de cobrança são enviados quando uma cobrança é paga."
  - `OPENPIX:CHARGE_COMPLETED`: Esse evento é enviado quando uma cobrança é paga.
  ```Json
  <!-- Trecho do payload de cobrança paga -->
  {
      "event": "OPENPIX:CHARGE_COMPLETED",
      "charge": {
          ...
          "status": "COMPLETED",
          ...
          "type": "DYNAMIC",
          ...
      }
      ...
  }
  ```

O cliente está esperando um webhook de transação `OPENPIX:TRANSACTION_RECEIVED` quando o webhook que retornou para ele é, possivelmente, o de `OPENPIX:CHARGE_COMPLETED` pelo indício deixado em sua mensagem através da palavra `COMPLETED` no console, o que sugere que apenas este webhook está configurado.

**2. Retentativas sem Sucesso - É um dos casos em que as 8 retentativas da woovi ocorreram sem sucesso:**

Infelizmente o cliente pode ter sido alvo de falha temporária que prejudicou especificamente essas 3 transações de cobrança ([Regras de Retentativa do Webhook](https://developers.woovi.com/docs/webhook/webhook-retry)). Por isso, precisou fazer a tentativa manualmente ([Reenviando webhooks na plataforma](https://developers.woovi.com/docs/webhook/webhook-resend)). Visto que ele afirma que de outros clientes funcionaram normalmente.

### > Resposta ao cliente:

Prezado, "Customer A".
Imagino a insegurança que esse problema pode estar causando na sua operação e a urgência para resolver antes que isso escale para níveis mais preocupantes e que impossibilitem a atualização de saldo em tempo "real".

Já dei uma analisada na sua solicitação para identificar os possíveis problemas de aparecer como `COMPLETED` no console, mas não chegar o evento específico de `OPENPIX:TRANSACTION_RECEIVED`, olhei nos logs do nosso lado da API e fiz o levantamento de 2 hipóteses principais:

**1. Diferença nos Tipos de Eventos (Charge vs Transaction):**
Com o pagamento das cobranças, o weebhook de cobrança paga (`OPENPIX:CHARGE_COMPLETED`) é o principal meio de confirmação vinculado a uma cobrança específica a ser enviado pela API, já o de transação recebida usa uma abordagem de acompanhamento de todas as transações (`OPENPIX:TRANSACTION_RECEIVED`) para monitorar tanto pagamentos que estão associados a uma cobrança (Pix Estático e Dinâmico), como também o Pix avulso. Você poderia verificar nos seus logs se as charges que me enviou estão dentro do payload desse webhook do tipo `OPENPIX:CHARGE_COMPLETED` ao invés do esperado `OPENPIX:TRANSACTION_RECEIVED`?

    Um exemplo do padrão esperado para ambos os payloads pode ser encontrado aqui: [Webhook > Tipos de eventos do Webhook](https://developers.woovi.com/docs/webhook/webhook-events-type).

**2. Retentativas sem Sucesso ou Timeout:**
Caso tenha ocorrido uma indisponibilidade e nenhuma das 8 tentativas automáticas da woovi tenha se finalizado com sucesso por algum dos três motivos especificados na documentação ([Regras de Retentativa do Webhook](https://developers.woovi.com/docs/webhook/webhook-retry)) ou timeout ([Tempo de timeout do Webhook](https://developers.woovi.com/docs/webhook/webhook-timeout)), a alternativa que restou foi reenviar manualmente. Pude perceber através do seu relato que o endpoint está funcionando corretamente, dado que o reenvio foi possível.

**Próximos Passos:**

Você poderia confirmar se ambos os endpoints para os webhooks - CHARGE_COMPLETED e TRANSACTION_RECEIVED - estão configurados? Segue um exemplo dos payloads esperados para cada um deles:

- [CHARGE_COMPLETED - Payload de Cobrança Pix Paga](https://developers.woovi.com/docs/webhook/examples/webhook-charge-payload)
- [TRANSACTION_RECEIVED - Payload de transação recebida](https://developers.woovi.com/docs/webhook/examples/webhook-transaction-received)

Caso estejam configurados corretamente com o que a API espera, mas o `TRANSACTION_RECEIVED` não aparece nos logs de vocês, me avise que estarei repassando imediatamente para a equipe responsável.
Posso te ligar 10min para verificarmos o que está acontecendo juntos.

### > Procedimentos Internos:

Criar um artigo para o blog, vídeo explicativo ou adicionar novos conceitos à documentação da API, para especificar casos de uso ("Imagine que você envia uma cobrança, espere receber nos logs...")

### > Referências Consultadas:

- **API - Documentação**
  - [**Webhook visão geral**](https://developers.woovi.com/docs/category/webhook-2)
    - [Webhook > Tipos de eventos do Webhook](https://developers.woovi.com/docs/webhook/webhook-events-type)
      -
    - [Webhook > Exemplos > Payload de Cobrança Pix Paga](https://developers.woovi.com/docs/webhook/examples/webhook-charge-payload)
    - [Webhook > Exemplos > Criando um webhook para interceptar um Pix via API](https://developers.woovi.com/docs/webhook/api/webhook-api)
    - [Webhook > Exemplos > Payload de transação recebida](https://developers.woovi.com/docs/webhook/examples/webhook-transaction-received)
  - [API no Scalar](https://developers.woovi.com/api)

- **Blog e FAQ de artigos**
  - [Potencialize seus processos de pagamento com o Webhook Pix da Woovi: Integração automatizada e eficiente](https://woovi.com/artigos/potencialize-seus-processos-de-pagamento-com-o-webhook-pix-da-woovi-integracao-automatizada-e-eficiente/)
  - [FAQ - Webhook](https://ajuda.woovi.com/hc/duvidas-frequentes/pt_BR/categories/webhook)
    - [Como reenviar um webhook específico](https://ajuda.woovi.com/hc/duvidas-frequentes/articles/como-reenviar-um-webhook-especifico)
  - [Bancos - Recebendo Pix com API, Webhook em bancos tradicionais](https://ajuda.woovi.com/hc/duvidas-frequentes/pt_BR/categories/bancos)
