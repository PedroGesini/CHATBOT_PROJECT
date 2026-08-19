# 📋 Como funciona o atendimento — todas as funções, passo a passo

Este documento detalha **tudo o que acontece** em cada uma das 5 opções
do menu principal — não só o caminho feliz, mas também as validações,
erros e casos de borda que o bot trata automaticamente. `🤖` é mensagem
do bot, `👤` é o que o cliente digita, e as notas em itálico explicam a
lógica por trás de cada resposta.

> Antes do menu, todo contato novo passa por um cadastro rápido (nome +
> CPF, ou nome da clínica + nome do atendente + CPF). Depois do primeiro
> cadastro, o telefone fica reconhecido e o bot pula direto pro menu.

```
🐾 Clínica Veterinária Exemplo 🐾

Boa tarde, Ana! Que bom ter você por aqui de novo. 🐾

Como podemos ajudá-lo hoje?

1️⃣ Agendar exame
2️⃣ Agendar mais de um exame
3️⃣ Reagendar exame
4️⃣ Cancelar exame
5️⃣ Falar com atendente

✍️ Digite apenas o número da opção desejada.
```

---

## 1️⃣ Agendar exame

### Passo 1 — Escolha da data
```
👤 1

🤖 📅 Datas disponíveis para agendamento

📌 Semana Atual
1️⃣ 18/08 (Ter) — Hoje  3 vagas ✅
2️⃣ 19/08 (Qua)  9 vagas ✅

📌 Próxima Semana
3️⃣ 24/08 (Seg)  9 vagas ✅

✍️ Digite apenas o número da opção desejada.
```
*A lista é montada na hora: o bot soma os agendamentos já salvos no
SQLite com os eventos encontrados direto no Google Calendar (pra
enxergar também horários marcados manualmente, fora do bot) e usa o
maior dos dois números como "ocupado". Um dia que bate o limite some
da lista — nunca aparece como "0 vagas". Domingos nunca entram na
lista.*

**Casos de borda:**
- Nenhuma data com vaga no mês inteiro → bot avisa e transfere pra um
  atendente automaticamente, sem nem chegar a mostrar a lista.
- Cliente digita algo que não é número → *"❌ Digite apenas o número da
  data desejada."* e reexibe a lista.
- Número fora do intervalo mostrado → *"❌ Número fora das opções
  disponíveis. Tente novamente."* e reexibe a lista.
- Cliente escolhe uma data, mas entre o menu ser mostrado e a escolha
  chegar, o dia lotou (outro cliente foi mais rápido) → bot detecta na
  hora, avisa que o dia "já está com a agenda completa" e reexibe a
  lista atualizada, sem travar a conversa.

### Passo 2 — Endereço (só na primeira vez, ou se ainda não tiver um salvo)
```
🤖 📅 Agora envie a rua, número e bairro. (obrigatório)
   Pode escrever em qualquer formato (com vírgula, um dado por
   linha, ou tudo junto)

👤 Rua das Palmeiras, 245, Jardim das Flores

🤖 📍 Endereço recebido:
   Rua das Palmeiras, 245, Jardim das Flores

   Você confirma este endereço?

   1️⃣ Sim, confirmar
   2️⃣ Corrigir endereço
```
*O bot aceita o endereço em qualquer formato de escrita (linha única,
quebrado em várias linhas, com ou sem vírgula) e normaliza sozinho.
Se o texto não tiver informação suficiente pra identificar rua/número/
bairro, pede de novo com a mesma instrução. Se o cliente já tiver um
endereço salvo de uma vez anterior, o bot pula esse passo inteiro e
só pergunta rapidinho: "Encontrei seu cadastro! Endereço: ...
Deseja usar esse endereço? 1️⃣ Sim 2️⃣ Atualizar".*

### Passo 3 — Escolha do horário
```
👤 1  (confirmando o endereço)

🤖 🕒 Horários — 19/08/2026

1️⃣ 09:00
2️⃣ 10:00
3️⃣ 13:00
4️⃣ 15:00
5️⃣ 16:00

✍️ Digite apenas o número do horário desejado.
```
*Mesma lógica de dupla checagem da data: um horário só aparece se
tiver vaga livre tanto no SQLite quanto no Google Calendar. Se a data
escolhida for hoje, horários que já passaram somem sozinhos da lista.*

**Casos de borda:**
- Todos os horários do dia lotaram entre a escolha da data e agora →
  bot avisa e devolve direto pro menu de datas, sem travar.
- Opção inválida ou fora do intervalo → *"❌ Opção inválida. Escolha um
  número de 1 a N:"* e reexibe a lista de horários.
- Horário escolhido acabou de ser preenchido por outro cliente (ou
  criado manualmente na agenda) no exato momento da escolha → bot avisa
  *"acabou de ser preenchido/ocupado"* e reexibe a lista atualizada.

### Passo 4 — Ficha do paciente
```
🤖 📋 Ficha do paciente

Copie a mensagem abaixo, cole na resposta e preencha os dados
depois de cada dois-pontos. Envie tudo de uma vez só:

🐶 Nome do paciente:
👤 Tutor:
💳 Plano ou particular:
🆔 CPF ou nº do microchip:
👨‍⚕️ Veterinário solicitante:
🐾 Espécie:
🦴 Raça:
🎂 Idade:
📍 Região a ser radiografada:
🔎 Suspeita:
📅 Data de início dos sintomas:
🏥 Paciente internado:
```
```
👤 🐶 Nome do paciente: Bidu
   👤 Tutor: Ana Beatriz Souza
   💳 Plano ou particular: Particular
   🆔 CPF ou nº do microchip: 111.222.333-44
   👨‍⚕️ Veterinário solicitante: Dra. Fernanda Lima
   🐾 Espécie: Canina
   🦴 Raça: SRD
   🎂 Idade: 6 anos
   📍 Região a ser radiografada: Abdômen
   🔎 Suspeita: Ingestão de corpo estranho
   📅 Data de início dos sintomas: 15/08/2026
   🏥 Paciente internado: Não
```
*O bot lê a mensagem inteira de uma vez e reconhece cada campo pelo
rótulo — funciona mesmo que o cliente mude a ordem, apague um emoji ou
escreva ligeiramente diferente.*

**Casos de borda:**
- CPF/microchip inválido → *"⚠️ CPF ou microchip inválido: [motivo]"*
  e reenvia o modelo pedindo pra corrigir só esse campo.
- Faltou preencher algum campo obrigatório → bot lista exatamente quais
  campos não recebeu e reenvia o modelo — **sem perder** os campos que
  já vieram certos.

### Passo 5 — Confirmação final
```
🤖 Recebi os dados! Confirma as informações abaixo?

*Data:* 19/08/2026
*Horário:* 13:00
🐶 Bidu · Ana Beatriz Souza · Particular · ...

Digite SIM para confirmar ou NÃO para corrigir.

👤 sim

🤖 ✅ Agendamento realizado com sucesso!

🐶 Paciente: Bidu
👤 Tutor: Ana Beatriz Souza
📅 Data: 19/08/2026
🕒 Horário: 13:00
👨‍⚕️ Veterinário: Dra. Fernanda Lima

📋 Seu agendamento foi registrado em nosso sistema.
📲 Em caso de dúvidas ou alterações, entre em contato conosco.

🐾 Agradecemos pela confiança!
Até logo!
```
*O bot aceita "sim", "s", "confirmar", "confirmo", "ok" ou "1" como
confirmação — não precisa ser a palavra exata. Antes de gravar,
**revalida a vaga uma última vez** (o tempo entre mostrar a ficha e o
cliente confirmar pode ser longo); se o horário sumiu nesse meio-tempo,
o bot avisa e manda recomeçar a escolha de data/horário, sem perder o
cadastro do cliente. Ao confirmar de verdade, grava no SQLite, cria o
evento no Google Calendar **e** registra a linha no Google Sheets — as
três coisas de uma vez, cada uma numa tentativa separada (se uma
falhar, ex. Sheets fora do ar, as outras duas não são afetadas).*

- Responder "não"/"corrigir"/"2" → bot reabre a ficha em branco pra
  preencher tudo de novo.
- Qualquer outra resposta → *"❓ Não entendi sua resposta. Digite SIM
  para confirmar ou NÃO para corrigir os dados"*.

---

## 2️⃣ Agendar mais de um exame

Segue o **mesmo fluxo da opção 1** (data → endereço → horário → ficha)
para o 1º exame, com uma diferença central: depois da ficha do 1º
paciente, o bot **não pede confirmação ainda** — ele guarda esses dados
de lado e pergunta como você quer agendar o 2º:

```
👤 2

🤖 📋 Agendamento de 2 exames

Vamos começar pelo 1º exame.

📅 Datas disponíveis para agendamento
[...]

(cliente completa data → endereço → horário → ficha do 1º paciente)

🤖 ✅ 1º exame registrado!

Data: 19/08/2026
Horário: 09:00
🐶 Rex · ...

Agora vamos ao 2º exame. Como prefere agendar?

1️⃣ Mesmo horário do 1º exame — 19/08/2026 às 09:00
2️⃣ Escolher outro dia/horário

✍️ Digite apenas o número da opção desejada.
```

**Se escolher "1" (mesmo horário):**
```
👤 1

🤖 ⏰ Horário do 2º exame: 09:00 — 2º exame

Agora vamos preencher a ficha do paciente. 📋

(ficha do 2º paciente)
```
*Os dois exames do mesmo horário ocupam **uma única vaga** — não duas
— porque contam como uma sessão só (o veterinário atende os dois
pacientes na mesma visita). O bot reconfirma que a vaga continua livre
antes de aceitar; se alguém preencheu bem nesse intervalo, avisa e pede
pra recomeçar.*

**Se escolher "2" (outro dia/horário):**
```
👤 2

🤖 📋 2º exame

📅 Datas disponíveis para agendamento
[...]
```
*Repete o fluxo completo de data → horário → ficha, independente do 1º
exame — nesse caso os dois exames podem cair em dias e horários
totalmente diferentes.*

### Confirmação final (dos dois exames juntos)
```
🤖 Recebi os dados dos 2 exames! Confirma as informações abaixo?

📋 1º exame
Data: 19/08/2026 · Horário: 09:00
🐶 Rex · ...

📋 2º exame
Data: 19/08/2026 · Horário: 15:00
🐱 Mimi · ...

Digite SIM para confirmar os dois agendamentos ou NÃO para corrigir.

👤 sim

🤖 ✅ Agendamentos realizados com sucesso!

📋 1º exame
🐶 Paciente: Rex · 👤 Tutor: Ana Beatriz Souza
📅 19/08/2026 · 🕒 09:00

📋 2º exame
🐶 Paciente: Mimi · 👤 Tutor: Ana Beatriz Souza
📅 19/08/2026 · 🕒 15:00

📋 Os dois agendamentos foram registrados em nosso sistema.
🐾 Agradecemos pela confiança!
```
*Ao confirmar, o bot revalida a vaga **dos dois exames** (não só do
segundo) antes de gravar — se o 1º exame perdeu a vaga enquanto você
preenchia o 2º, o bot avisa e reinicia o processo desde a escolha de
data, sem misturar dados velhos com novos. Cada exame vira um registro
próprio no banco, uma linha própria na planilha e um evento próprio no
Calendar — mesmo os que usam "mesmo horário", que geram dois eventos
distintos ocupando a mesma vaga.*

---

## 3️⃣ Reagendar exame

Diferente das opções 1 e 2, o reagendamento **não é 100% automático** —
ele coleta os dados básicos e encaminha para um atendente humano
finalizar, já que envolve localizar um agendamento existente em
diferentes fontes (bot, planilha, agenda) e decidir a melhor forma de
remanejar:

```
👤 3

🤖 Para reagendar, informe o nome do paciente e a data do
   agendamento atual:

👤 Bidu, agendado para 19/08 às 13:00

🤖 Anotei! Um atendente vai te chamar para finalizar o processo.
   Aguarde um momento.
```
*A partir daqui a conversa entra na fila de atendimento humano (mesmo
mecanismo da opção 5) — o bot registra o pedido e para de responder
automaticamente nesse telefone, deixando o atendente assumir.*

---

## 4️⃣ Cancelar exame

O ponto de partida muda dependendo de quem está falando:

```
👤 4

🤖 (se for clínica solicitante)
   ❌ Cancelamento de agendamento

   Para localizar o agendamento, informe o nome da clínica
   solicitante cadastrado:

🤖 (se for cliente/tutor)
   ❌ Cancelamento de agendamento

   Para localizar seu agendamento, informe o CPF cadastrado no
   formato 000.000.000-00 (ou o número do microchip, se o exame
   foi cadastrado por ele):
```

### Busca e lista de agendamentos
```
👤 111.222.333-44

🤖 📋 Agendamentos encontrados:

1️⃣ Bidu — 19/08/2026 às 13:00
2️⃣ Rex, Mimi — 20/08/2026 às 09:00

Digite o número do agendamento que deseja cancelar:
```
*Agendamentos marcados no **mesmo dia e horário** (caso da opção "2
exames com mesmo horário") aparecem **agrupados como um item só** na
lista — assim o cliente cancela a visita inteira de uma vez, e nenhum
dos dois exames fica "esquecido" ocupando vaga ou evento na agenda.*

**Casos de borda:**
- Nome de clínica muito curto (menos de 3 caracteres) → *"❌ Nome
  inválido."* e pede de novo.
- CPF/clínica sem nenhum agendamento encontrado → bot avisa e volta
  direto pro menu principal.
- Número da lista inválido ou fora do intervalo → reexibe a lista com
  *"❌ Opção inválida. Escolha um número de 1 a N:"*.

### Confirmação
```
👤 1

🤖 ⚠️ Confirma o cancelamento do agendamento abaixo?

🐶 Paciente: Bidu
👤 Tutor: Ana Beatriz Souza
📅 Data: 19/08/2026
🕒 Horário: 13:00

Digite SIM para confirmar o cancelamento ou NÃO para voltar ao menu.

👤 sim

🤖 ✅ Agendamento cancelado com sucesso!

A vaga foi liberada automaticamente, e o cancelamento já foi
refletido na planilha e removido da agenda do Google Calendar.
Se precisar reagendar, é só escolher a opção no menu.

[...menu principal...]
```
*Cancelar remove o registro do SQLite (isso é o que realmente libera a
vaga) e tenta remover também a linha da planilha e o evento do
Calendar — se algum desses dois falhar (ex. Google fora do ar), o
cancelamento no banco não é desfeito; o bot informa quantos itens
foram cancelados com sucesso mesmo que um grupo tenha 2 exames e só um
consiga ser removido dessa vez.*

- Responder "não" → *"Ok, cancelamento não realizado."* e volta ao
  menu sem alterar nada.
- Resposta não reconhecida → pede SIM/NÃO de novo.

---

## 5️⃣ Falar com atendente

```
👤 5

🤖 Certo! Vou te conectar com um de nossos atendentes.

Aguarde um momento, alguém irá te atender em breve.
```
*A partir daqui o bot entra em modo de silêncio nesse telefone — não
responde mais automaticamente a nada — pra não atropelar o atendente
humano enquanto ele conduz a conversa. Isso é o mesmo mecanismo usado
internamente pela opção 3 (reagendar) e por qualquer erro grave que o
bot não saiba resolver sozinho (ex. mês inteiro sem vaga).*

**Enquanto está na fila (bot em silêncio):**
```
👤 menu

🤖 Ok, atendimento humano cancelado. 🐾

[...menu principal...]
```
*O cliente pode sair da fila a qualquer momento digitando "menu",
"voltar" ou "cancelar" — única exceção em que o bot volta a responder
sozinho nesse estado.*

**Quando um atendente assume a conversa manualmente:**
```
👤 (qualquer mensagem durante o atendimento humano)

🤖 (sem resposta — bot fica em silêncio até o atendente encerrar)
```
*Terminado o atendimento (comando do atendente ou o próprio cliente
digitando "menu"/"voltar"/"cancelar"), o bot devolve
"✅ Atendimento humano encerrado." e o menu principal, voltando ao
funcionamento normal.*

---

## 🔔 Bônus: lembrete automático

Um dia antes de cada exame, sem nenhuma ação do cliente ou da clínica:

```
🤖 🐾 Lembrete de Agendamento — Clínica Veterinária

Olá, Ana Beatriz Souza! 👋

Lembramos que você tem um exame agendado para amanhã:

🐶 Paciente: Bidu
📅 Data: 20/08/2026
🕒 Horário: 14:00
👨‍⚕️ Veterinário: Dra. Fernanda Lima

📍 Caso precise cancelar ou reagendar, entre em contato conosco.

Até amanhã! 🐾
```
*Uma rotina separada roda uma vez por dia, varre o banco atrás de
agendamentos marcados pra "amanhã" e dispara essa mensagem pra cada um
— sem depender do bot principal estar processando conversa nenhuma
nesse momento.*
