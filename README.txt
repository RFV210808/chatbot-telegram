##Telegram Weather Chatbot (n8n)

Este projeto implementa um chatbot no Telegram que retorna a temperatura atual de uma cidade brasileira utilizando a API OpenWeather e automação com n8n.

O bot recebe uma mensagem com o formato Cidade,UF e responde com a temperatura atual em graus Celsius.

Exemplo:

Usuário: Uberlândia,MG
Bot: A temperatura em Uberlândia é de 20°C.

Se a cidade não for encontrada ou o formato estiver incorreto, o bot retorna:
❌ Cidade não encontrada. Use o formato Cidade,UF (ex.: São Paulo,SP).

##Tecnologias utilizadas

n8n – Automação de workflow
Telegram Bot API
OpenWeather API
JavaScript Expressions (n8n)

##Como funciona o workflow

O fluxo do bot é dividido nas seguintes etapas:

#Telegram Trigger

O workflow inicia quando o bot recebe uma mensagem no Telegram.
O texto enviado pelo usuário contém a cidade e estado no formato: Cidade,UF
Exemplo: Uberlândia,MG

#Normalização da entrada

O texto recebido é transformado em uma variável chamada queue.
Expressão utilizada:
{{
(() => {
  const raw = ($json.message?.text || '').trim();

  // remove acentos e coloca minúsculo
  let s = raw.normalize('NFD').replace(/[\u0300-\u036f]/g, '').toLowerCase();

  // troca múltiplos espaços por 1
  s = s.replace(/\s+/g, ' ').trim();

  // aceita "cidade uf" (sem vírgula) e transforma em "cidade,UF"
  if (!s.includes(',')) {
    const parts = s.split(' ');
    if (parts.length >= 2) {
      const uf = parts.pop().toUpperCase();
      const city = parts.join(' ');
      s = `${city},${uf}`;
    }
  } else {
    // remove espaço em volta da vírgula e deixa UF maiúscula
    const [city, uf] = s.split(',').map(t => t.trim());
    s = `${city},${(uf || '').toUpperCase()}`;
  }

  // adiciona país BR (muito útil pro OpenWeather)
  if (!s.endsWith(',BR')) s = `${s},BR`;

  return s;
})()
}}

Exemplo:

Entrada do usuário:  Uberlândia,MG
queue gerado: Uberlândia,MG,BR
Adicionar ",BR" ajuda a API OpenWeather a localizar corretamente cidades brasileiras.

#Requisição para OpenWeather

O node HTTP Request consulta a API:
https://api.openweathermap.org/data/2.5/weather

Parâmetros enviados:
| Parâmetro | Descrição        |
| --------- | ---------------- |
| q         | cidade formatada |
| units     | metric (Celsius) |
| lang      | pt_br            |
| appid     | chave da API     |

Exemplo de requisição:
https://api.openweathermap.org/data/2.5/weather?q=Uberlândia,MG,BR

#Validação da resposta

O node IF verifica se a requisição foi bem-sucedida.

Condições verificadas:
statusCode = 200
AND
body.main.temp não está vazio

Se a validação for verdadeira, o fluxo continua para gerar a resposta.
Caso contrário, o fluxo envia uma mensagem de erro.

#Formatação da resposta

Quando a cidade é encontrada, o bot monta a mensagem final.

Exemplo de expressão:
{{`🌤️ A temperatura em ${$json.body.name} é de ${Math.round($json.body.main.temp)}°C.`}}
Resultado:
🌤️ A temperatura em Uberlândia é de 20°C.
