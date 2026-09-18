# NEO Defense — Monitoramento Orbital

Painel interativo que monitora objetos próximos à Terra (asteroides) usando dados reais da NASA. Feito como projeto de portfólio: front-end puro, sem framework e sem build step — um único arquivo HTML que roda em qualquer navegador.

![status](https://img.shields.io/badge/status-em%20desenvolvimento-A4FFF0) ![dados](https://img.shields.io/badge/dados-NASA%20NeoWs-FC3D21)

---

## O que o painel faz

- **Mapa orbital em 3D** com a Terra no centro (texturas reais, camada de nuvens, brilho atmosférico) e os asteroides do período posicionados por distância real, cada um com sua própria rocha 3D texturizada
- **Clique num asteroide** para ver a trajetória da passagem (geometria fiel à distância mínima e à velocidade reais), um rótulo flutuante e o detalhe completo do objeto
- **Elementos orbitais reais** — classe, excentricidade, inclinação, período, número de observações — buscados sob demanda no endpoint de detalhe da NASA
- **Leituras ao vivo**: medidor de distância, gráfico de velocidade, índice de risco, postura orbital (nominal / atenção / crítica) calculada a partir dos objetos classificados como potencialmente perigosos
- **Lista de objetos rastreados**, filtrável por risco, com rolagem própria
- **Aba de Histórico**: agrega até 180 dias de dados (encadeando requisições de 7 em 7 dias, o limite da API), com gráfico de aproximações por semana, top 10 passagens mais próximas e distribuição por porte

---

## Rodando localmente

Não tem build, não tem `npm install`. É um arquivo só:

1. Baixe `neo-defense.html`
2. Abra direto no navegador (duplo clique, ou `open neo-defense.html` / `start neo-defense.html`)
3. Pronto — precisa de internet, porque os dados vêm ao vivo da API da NASA

### Conseguindo uma chave de acesso da NASA

O painel funciona sem chave própria (usa `DEMO_KEY`), mas essa chave trava depois de ~30 consultas por hora — pouco para a aba de Histórico com períodos longos.

1. Acesse [api.nasa.gov](https://api.nasa.gov/)
2. Preencha nome, e-mail e uma frase curta descrevendo o uso (ex: *"personal portfolio project using NeoWs data"*)
3. A chave chega por e-mail na hora — sem aprovação manual
4. Cole no campo **"Chave de acesso da NASA"** no painel; ela fica salva no `localStorage` do seu navegador

**Trate a chave como uma senha**: não a commite em repositório público. Se for versionar, use um `.env` e adicione ao `.gitignore` (mesmo sendo uma chave de baixo risco — só dá acesso a dados públicos, sem custo).

---

## Publicando (GitHub Pages / Vercel / Netlify)

Como é HTML estático, qualquer um desses funciona sem configuração:

- **GitHub Pages**: suba o arquivo como `index.html` na branch e ative o Pages nas configurações do repositório
- **Vercel / Netlify**: arraste a pasta no painel deles, ou conecte o repositório — nenhum comando de build é necessário

---

## Stack

| Camada | Tecnologia |
|---|---|
| Estrutura / lógica | HTML + JavaScript vanilla (sem framework) |
| Cena 3D | [three.js](https://threejs.org/) (r128), via CDN |
| Dados | [NASA NeoWs API](https://api.nasa.gov/) — endpoints `/feed` e `/neo/{id}` |
| Tipografia | Urbanist (títulos) + Inconsolata (dados), Google Fonts |
| Texturas da Terra | Extraídas de um modelo Blender (conjunto Blue Marble da NASA), reamostradas e embutidas como `data:` URI |
| Modelo do asteroide | Malha `.obj` fornecida, reindexada e quantizada em JavaScript, também embutida no arquivo |

Não há dependência de servidor, banco de dados ou build step. Tudo — inclusive as texturas — está embutido no único arquivo `.html`, o que facilita a distribuição mas deixa o arquivo com pouco mais de 1 MB.

---

## Decisões técnicas que valem destacar

Estas são as escolhas que acho que rendem boa conversa numa entrevista técnica:

- **Requisições em série, não em paralelo, no Histórico.** O endpoint da NASA aceita no máximo 7 dias por chamada. Disparar todas de uma vez para um período de 180 dias estouraria o limite por hora da chave; encadeá-las com indicador de progresso resolve sem perder dados.
- **Geometria quantizada para o modelo 3D.** O `.obj` original tinha ~1 MB em texto puro. Reindexando os vértices e quantizando posições (Int16), normais (Int8) e UVs (Uint16), o mesmo modelo ficou em ~170 KB binário — sem perda visual perceptível.
- **Fallback sem dependência externa para a Terra.** Uma primeira versão usava o embed oficial da NASA via `<iframe>`, mas o servidor deles envia `X-Frame-Options` e recusa ser exibido em outro domínio. A solução foi renderizar a Terra inteiramente em WebGL com texturas próprias, o que também eliminou qualquer risco de o painel quebrar por causa de terceiros.
- **Honestidade sobre os limites dos dados.** A trajetória desenhada no mapa é geometricamente fiel à distância mínima e à velocidade reais de cada objeto, mas a API não informa a direção de chegada — isso está deixado explícito na interface, em vez de simular uma precisão que os dados não sustentam.
- **CSS Grid e `min-width: auto`.** A lista de objetos rastreados "vazava" para fora do card porque itens de grid não encolhem por padrão. `min-width: 0` nos contêineres resolveu — um bug clássico e fácil de esquecer.

---

## Estrutura do projeto

```
neo-defense.html   # o painel inteiro: HTML, CSS e JS num arquivo só
README.md          # este arquivo
```

---

## Dados e créditos

- **Dados de asteroides**: [NASA NeoWs — Near Earth Object Web Service](https://api.nasa.gov/), domínio público
- **Texturas da Terra**: conjunto Blue Marble da NASA (cor, nuvens, máscara oceânica, luzes noturnas), extraídas de um modelo `.blend` fornecido pelo autor do projeto
- **Modelo do asteroide**: malha `.obj` fornecida pelo autor do projeto. **Antes de redistribuir publicamente**, confirme a licença original do arquivo — modelos de terceiros às vezes têm termos próprios mesmo quando o assunto (um asteroide real) é de domínio público
- **Direção visual**: paleta e tipografia adaptadas de um guia de estilo (Urbanist/Inconsolata, paleta menta/vermelho/laranja/amarelo) e de uma referência de dashboard de operações; cores institucionais da NASA usadas como aceno de marca, sem reproduzir o logotipo oficial

---

## Limitações conhecidas

- A chave `DEMO_KEY` trava em ~30 requisições/hora — insuficiente para o Histórico com períodos longos
- A trajetória de aproximação é esquemática na orientação (fiel na forma e na distância, não na direção real de chegada)
- O modelo 3D do asteroide é único e reaproveitado para todos os objetos (varia porte, rotação e órbita, não a forma)

---
