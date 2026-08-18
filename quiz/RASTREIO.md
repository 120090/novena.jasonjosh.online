# RASTREIO — Quiz das Nove Noites, Casa Ch'i

**Esquema versão 1.** Este arquivo é a fonte da verdade do rastreio do quiz. Com ele e a lista de vendas do mês, dá para reconstruir quem comprou, o que a pessoa respondeu nas 14 etapas e por onde ela entrou.

> **Antes de subir para a hospedagem, tire este arquivo da pasta.** Ele está aqui junto do `index.html` porque é aqui que ele é útil, mas se a pasta inteira subir, qualquer pessoa que abrir `seudominio.com/quiz/RASTREIO.md` lê o mapa e passa a decodificar os códigos. Não há dado pessoal nenhum aqui, então o risco não é de vazamento de cliente. O que vaza é o desenho do funil, para quem quiser copiar.

---

## O que é o código

Cada pessoa que termina o quiz recebe um código com cara de identificador comum:

```
4c9e387e-31b8-4460-a2e7-a80211604528
```

Ele viaja em três lugares:

| Onde | Campo |
|---|---|
| URL da página de venda | `?src=` |
| Link do checkout da Hotmart | `src` |
| Eventos do Meta | parâmetro `src` no `QuizComplete`, `QuizCTA`, `QuizRedirect` e `QuizArrival` |

É `src` nos três, sempre com o mesmo valor.

**O campo é o `src`, nunca o `sck`.** O script de UTMs que está nas páginas não tem o `data-utmify-prevent-xcod-sck`, então a Utmify ocupa `sck` e `xcod` para levar os UTMs até a venda. Se o quiz escrevesse ali também, um apagaria o outro e você perderia ou a origem do anúncio ou as respostas.

Para saber se uma venda veio do quiz, olhe o formato do `src`: **se tem cara de UUID, veio do quiz**. As outras origens não têm. O back redirect, por exemplo, chega como `src=backredirect`.

Ele foi feito para **não parecer rastreio**. Passa por um UUID v4 legítimo, inclusive na marca de versão e na variante, e as respostas são embaralhadas para que duas pessoas com respostas parecidas não gerem códigos parecidos. A pessoa vê um identificador qualquer na barra de endereço, e não as próprias respostas escritas de volta para ela.

---

## Como abrir o código

**Passo 1.** Tire os hífens. Sobram 32 casas hexadecimais, numeradas de 0 a 31.

```
4c9e387e-31b8-4460-a2e7-a80211604528
4c9e387e31b84460a2e7a80211604528
0         1         2         3
01234567890123456789012345678901
```

**Passo 2.** Cada casa guarda um número de 0 a 15, e esse número está embaralhado. Para desembaralhar, faça **XOR** entre o valor da casa e o valor da máscara na mesma posição.

```
MÁSCARA = 5c9f2a7e31b8d46052e7a1c93f8b6d40
```

`valor real = XOR(casa do código, casa da máscara)`

**Passo 3.** Leia cada casa com a tabela abaixo.

| Casa | O que é |
|---|---|
| 0 | versão do esquema. Tem que dar `1`. Se der outro número, este arquivo não serve para esse código |
| 1 | **rota final**, a página de novena para onde a pessoa foi mandada |
| 2 | **escolha da etapa 14** |
| 3 | **tempo de convivência**, a resposta da etapa 8 |
| 4 | **kit recomendado** |
| 5 | **motivo do kit** |
| 6 a 11 | etapas 1 a 6 |
| 12 | sempre `4`. É a marca de UUID v4. **Não passa pela máscara e não quer dizer nada** |
| 13 a 15 | etapas 7 a 9 |
| 16 | variante do UUID, sorteada entre `8`, `9`, `a` e `b`. **Não passa pela máscara e não quer dizer nada** |
| 17 a 20 | etapas 10 a 13 |
| 21 a 31 | **semente da sessão**, 11 casas sorteadas. Não passa pela máscara |

A semente das casas 21 a 31 é o mesmo número que vai como `qid` nos eventos do Meta. **É por ela que se cruza uma sessão do Meta com uma venda da Hotmart.**

---

## Tabelas de valores

### Casa 1 e casa 2 — perfis

| valor | tag | rótulo que a pessoa viu |
|---|---|---|
| `0` | `amor` | A Espera Silenciosa |
| `1` | `cura` | A Vigília Solitária |
| `2` | `prosp` | O Bloqueio Selado |
| `3` | `prot` | A Brecha Aberta |
| `4` | `fert` | A Espera Marcada |
| `5` | `liber` | A Corrente Invisível |
| `6` | `vicio` | A Corrente Dentro de Casa |
| `7` | `apost` | O Ciclo do Jogo |
| `8` | `segr` | A Novena Incompleta |
| `9` | `devoc` | A Devoção Firme |
| `f` | — | não identificado |

A casa 1 é a rota final, para onde ela foi. A casa 2 é o que ela escolheu na etapa 14. **As duas podem ser diferentes**, porque a rota tem regras de desempate que às vezes mandam para outra página.

### Casa 3 — tempo de convivência

| valor | resposta |
|---|---|
| `0` | não respondeu, chegou ao fim por um caminho que pulou a etapa 8 |
| `1` | menos de 6 meses |
| `2` | entre 6 meses e 2 anos |
| `3` | entre 2 e 5 anos |
| `4` | mais de 5 anos |

### Casa 4 — kit

| valor | kit |
|---|---|
| `0` | simples, 1 novena |
| `1` | duplo, 2 novenas, 18 velas |

Hoje o quiz recomenda o duplo para todo mundo, então esta casa é sempre `1`. Ela existe para o dia em que isso mudar.

### Casa 5 — motivo do kit

Qual argumento a pessoa viu na tela de resultado.

| valor | motivo | argumento apresentado |
|---|---|---|
| `0` | multi | ela marcou que teme mais de uma coisa ao mesmo tempo |
| `1` | tempo | convive com isso há 2 anos ou mais |
| `2` | agradecimento | padrão, a segunda novena fica para agradecer |

---

## As 14 etapas

O valor guardado é a **posição da opção**, contando de cima para baixo a partir do zero. A etapa 12 tem campo de texto livre, e **o que a pessoa escreve ali não é gravado em lugar nenhum**. Só a posição da opção.

### Etapa 1 — casa 6

> Toda oração começa por um desejo. Neste momento, eu desejo…

| valor | opção |
|---|---|
| `0` | …ver o amor voltar para a minha vida |
| `1` | …ver a saúde restaurada em alguém que eu amo |
| `2` | …ver o dinheiro finalmente entrar e ficar |
| `3` | …ver a minha casa em paz e protegida |
| `4` | …ver um filho chegar |
| `5` | …ver alguém que eu amo livre do que domina ele |
| `6` | …ver a mim mesma leve de novo |
| `7` | …só continuar firme na minha fé |
| `f` | não respondida |

### Etapa 2 — casa 7

> Este ano, eu…

| valor | opção |
|---|---|
| `0` | …tenho entre 25 e 34 anos, e ainda tenho muito caminho pela frente |
| `1` | …tenho entre 35 e 44, e é agora que eu quero ver as coisas se ajeitarem |
| `2` | …tenho entre 45 e 54, e já aprendi o que realmente importa |
| `3` | …tenho 55 ou mais, e a minha fé só cresceu com o tempo |
| `f` | não respondida |

### Etapa 3 — casa 8

> Quando você reza, você reza…

| valor | opção |
|---|---|
| `0` | Todo dia. É um hábito que me sustenta |
| `1` | Nos momentos em que a vida aperta |
| `2` | Já rezei muito. Hoje ando mais distante |
| `3` | Quero voltar a rezar, e é por isso que estou aqui |
| `f` | não respondida |

### Etapa 4 — casa 9

> Eu queria acordar amanhã sentindo…

| valor | opção |
|---|---|
| `0` | …que alguém voltou a me querer perto |
| `1` | …que o resultado do exame veio bom |
| `2` | …que a conta fecha esse mês sem aperto |
| `3` | …que a minha casa está leve, sem clima pesado |
| `4` | …que a espera acabou e a vida vem chegando |
| `5` | …que quem eu amo dormiu em casa, inteiro |
| `6` | …que o peso que eu carrego saiu de cima de mim |
| `f` | não respondida |

### Etapa 5 — casa 10

> Você acredita que existe alguma coisa agindo além do que a gente consegue ver?

| valor | opção |
|---|---|
| `0` | Sim, sempre acreditei |
| `1` | Acredito, mas ando com a fé abalada |
| `2` | Nunca pensei muito nisso, mas estou aberta |
| `3` | Eu já vi coisas acontecerem que ninguém soube explicar |
| `f` | não respondida |

### Etapa 6 — casa 11

> Nos últimos meses, o que mais tem tirado o seu sono?

| valor | opção |
|---|---|
| `0` | Uma pessoa que se afastou e não responde mais |
| `1` | Um diagnóstico, uma internação, um exame |
| `2` | Boleto, dívida, dinheiro que não rende |
| `3` | Briga do nada em casa e uma sequência de azar que não fecha conta |
| `4` | Mais um mês sem a notícia que eu espero |
| `5` | O que a bebida ou a droga fez com alguém que eu amo |
| `6` | O quanto já se perdeu no jogo e nas apostas |
| `7` | Um cansaço e um peso que eu não consigo explicar |
| `f` | não respondida |

### Etapa 7 — casa 13

> Às vezes eu…

| valor | opção |
|---|---|
| `0` | …pego o celular no meio da noite só pra ver se essa pessoa apareceu |
| `1` | …rezo baixinho no corredor do hospital pra ninguém ouvir |
| `2` | …abro o app do banco esperando um número diferente do que eu já sei |
| `3` | …sinto uma coisa pesada dentro de casa, e ninguém ali fala nisso |
| `4` | …evito encontro de família pra não ouvir "e o bebê?" |
| `5` | …confiro escondida o bolso, o quarto ou o extrato de alguém |
| `6` | …acordo de madrugada com o coração disparado, sem motivo nenhum |
| `f` | não respondida |

### Etapa 8 — casa 14

> Eu convivo com isso há…

| valor | opção |
|---|---|
| `0` | Menos de 6 meses |
| `1` | Entre 6 meses e 2 anos |
| `2` | Entre 2 e 5 anos |
| `3` | Mais de 5 anos. Já perdi a conta |
| `f` | não respondida |

### Etapa 9 — casa 15

> Para tentar resolver isso, eu já…

| valor | opção |
|---|---|
| `0` | …conversei, esperei, tentei do meu jeito |
| `1` | …procurei médico, terapeuta, especialista |
| `2` | …fiz simpatia, banho, promessa |
| `3` | …fiz uma novena inteira, cheguei ao nono dia, e ainda assim faltou alguma coisa |
| `4` | …não tentei nada de verdade ainda |
| `f` | não respondida |

### Etapa 10 — casa 17

> Isso é difícil de admitir em voz alta, mas eu sou…

| valor | opção |
|---|---|
| `0` | …a pessoa que ainda espera uma ligação que talvez não venha |
| `1` | …a pessoa que segura sozinha a barra da doença de todo mundo |
| `2` | …a pessoa que trabalha, trabalha, e nunca sobra nada |
| `3` | …a pessoa que sente a casa pesada e não consegue provar isso pra ninguém |
| `4` | …a pessoa que finge que está tudo bem toda vez que alguém anuncia uma gravidez |
| `5` | …a pessoa que reza escondida no banheiro por quem não consegue parar |
| `6` | …a pessoa que acorda cansada mesmo depois de dormir a noite inteira |
| `f` | não respondida |

### Etapa 11 — casa 18

> Por mais que eu tente, eu não consigo…

| valor | opção |
|---|---|
| `0` | …tirar essa pessoa da minha cabeça |
| `1` | …aceitar a frase "já fizemos tudo o que podíamos" |
| `2` | …fazer o dinheiro parar na minha mão |
| `3` | …ter uma semana de paz dentro da minha própria casa |
| `4` | …parar de contar os dias no calendário |
| `5` | …fazer quem eu amo enxergar o que isso está destruindo |
| `6` | …me livrar da sensação de que alguma coisa trava a minha vida por fora |
| `f` | não respondida |

### Etapa 12 — casa 19

> E tem uma coisa que eu sinto e quase nunca digo. Eu sinto…

| valor | opção |
|---|---|
| `0` | …vergonha de ainda estar esperando essa pessoa |
| `1` | …culpa por não estar conseguindo fazer mais |
| `2` | …vergonha do lugar a que as minhas contas chegaram |
| `3` | …medo de que alguém tenha desejado o mal pra mim de propósito |
| `4` | …que o meu corpo está me devendo alguma coisa |
| `5` | …raiva de quem eu amo, e depois culpa por sentir raiva |
| `6` | …que eu não sou mais a mesma pessoa que eu era |
| `7` | …outra coisa |
| `f` | não respondida |

### Etapa 13 — casa 20

> O que eu mais temo é que, daqui a um ano…

| valor | opção |
|---|---|
| `0` | …essa pessoa esteja construindo uma vida com outra |
| `1` | …eu esteja olhando para uma cadeira vazia na mesa |
| `2` | …eu esteja mais endividada do que estou hoje |
| `3` | …essa coisa pesada tenha alcançado os meus filhos também |
| `4` | …eu ainda esteja esperando, e o tempo tenha passado de vez |
| `5` | …chegue aquela ligação de madrugada com a pior notícia |
| `6` | …eu esteja exatamente onde estou hoje, sem ter saído do lugar |
| `7` | Tenho medo de mais de uma dessas ao mesmo tempo |
| `f` | não respondida |

### Etapa 14 — casa 2

> Agora respira. Se estas nove noites pudessem resolver uma única coisa, eu escolheria…

Esta etapa **não tem posição fixa**. As opções são montadas na hora, a partir do que a pessoa respondeu nas 13 anteriores, então a mesma posição apontaria para textos diferentes em cada sessão. Por isso ela é gravada pelo **perfil escolhido**, na casa 2, usando a mesma tabela de perfis lá de cima.

---

## Os 8 degraus no Meta

O funil dispara um evento personalizado por degrau. Todos levam `qid`, que é a semente da sessão, e `etapa`, que é o número do degrau.

| # | Evento | Quando |
|---|---|---|
| 1 | `QuizLand` | abriu a capa |
| 2 | `QuizStart` | clicou em começar |
| 3 | `QuizQ25` | chegou na etapa 5 |
| 4 | `QuizQ50` | chegou na etapa 8 |
| 5 | `QuizQ75` | chegou na etapa 12 |
| 6 | `QuizComplete` | viu o resultado. Leva também `perfil`, `kit`, `kit_motivo` e `ref` |
| 7 | `QuizCTA` | clicou no botão da novena |
| 7 | `QuizRedirect` | saiu pelo cronômetro de 60s, sem clicar |
| 8 | `QuizArrival` | chegou na página de venda |

Cada degrau conta **uma vez por sessão**, mesmo que a pessoa volte e avance de novo.

Existe ainda um `QuizStep` por etapa, com o número em `step`, que só dispara quando `CONFIG.trackCadaPergunta` está ligado no `index.html`. Serve para descobrir em qual etapa as pessoas desistem. Ligue por alguns dias e desligue.

---

## Decodificador

Cole os códigos em `CODIGOS` e rode. Aceita o código com ou sem hífens.

```python
# -*- coding: utf-8 -*-
import json

CODIGOS = [
    "4c9e387e-31b8-4460-a2e7-a80211604528",
]

MASK    = "5c9f2a7e31b8d46052e7a1c93f8b6d40"
PERFIS  = ["amor","cura","prosp","prot","fert","liber","vicio","apost","segr","devoc"]
KITS    = ["simples","duplo"]
MOTIVOS = ["multi","tempo","agradecimento"]
TEMPO   = ["nao respondeu","menos de 6 meses","6 meses a 2 anos","2 a 5 anos","mais de 5 anos"]
CASAS_ETAPA = [6,7,8,9,10,11,13,14,15,17,18,19,20]

def tabela(lista, i):
    return lista[i] if 0 <= i < len(lista) else "?"

def abrir(codigo):
    h = codigo.replace("-", "").strip().lower()
    if len(h) != 32:
        return {"codigo": codigo, "erro": "nao tem 32 casas"}
    casa = lambda i: int(h[i], 16) ^ int(MASK[i], 16)
    if casa(0) != 1:
        return {"codigo": codigo, "erro": "esquema versao %d, este arquivo le a 1" % casa(0)}
    return {
        "codigo":   codigo,
        "rota":     tabela(PERFIS, casa(1)),
        "etapa_14": tabela(PERFIS, casa(2)),
        "tempo":    tabela(TEMPO, casa(3)),
        "kit":      tabela(KITS, casa(4)),
        "motivo":   tabela(MOTIVOS, casa(5)),
        "respostas": {("etapa_%d" % (n + 1)): casa(c) for n, c in enumerate(CASAS_ETAPA)},
        "semente":  h[21:],
    }

for c in CODIGOS:
    print(json.dumps(abrir(c), ensure_ascii=False, indent=1))
```

Em JavaScript, o mesmo:

```javascript
const MASK = '5c9f2a7e31b8d46052e7a1c93f8b6d40';
const CASAS_ETAPA = [6,7,8,9,10,11,13,14,15,17,18,19,20];

function abrir(codigo){
  const h = codigo.replace(/-/g, '').toLowerCase();
  const casa = i => parseInt(h[i], 16) ^ parseInt(MASK[i], 16);
  if(h.length !== 32 || casa(0) !== 1) return null;
  return {
    rota:     ['amor','cura','prosp','prot','fert','liber','vicio','apost','segr','devoc'][casa(1)],
    etapa14:  ['amor','cura','prosp','prot','fert','liber','vicio','apost','segr','devoc'][casa(2)],
    tempo:    casa(3),
    kit:      ['simples','duplo'][casa(4)],
    motivo:   ['multi','tempo','agradecimento'][casa(5)],
    respostas: CASAS_ETAPA.map(casa),
    semente:  h.slice(21)
  };
}
```

---

## Fechamento do mês

Exporte as vendas da Hotmart com a coluna `src`. As linhas em que o `src` tem cara de UUID são as vendas que vieram do quiz. Junte esse arquivo com este documento e peça algo assim:

> Aqui estão as vendas do mês e o RASTREIO.md do quiz. Decodifique o `src` de cada venda seguindo o documento e me diga: qual rota mais vendeu, qual perfil da etapa 14 converte melhor, qual faixa de tempo de convivência compra mais, e se existe alguma resposta nas etapas 1 a 13 que aparece muito mais em quem compra do que em quem só responde o quiz.

Para comparar com quem **não** comprou, exporte também os eventos `QuizComplete` do Gerenciador do Meta, que levam o mesmo `ref`. A diferença entre os dois conjuntos é o retrato de quem chega e não converte.

---

## Quando este arquivo deixa de valer

O código é lido por posição. Se você **mexer nas perguntas ou nas opções**, os códigos antigos passam a ser lidos errado, em silêncio, sem erro nenhum.

Ao mudar qualquer opção de qualquer etapa:

1. suba `COD.versao` no `index.html` e `COD_VERSAO` no `snippet-paginas.html`;
2. guarde uma cópia deste arquivo como `RASTREIO-v1.md`, para os códigos antigos continuarem legíveis;
3. gere o novo `RASTREIO.md`.

Acrescentar uma opção **no fim** de uma etapa é seguro e não exige nova versão, desde que a ordem das opções que já existiam não mude. Cada etapa comporta até 15 opções.
